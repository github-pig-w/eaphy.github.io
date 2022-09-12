---
title: spring 多线程保证事务特性
tags:
  - 线程池
categories:
  - 多线程
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2019-12-22 14:02:51
password:
---

在业务逻辑中，开启多线程，可以提高性能，但是子线程报错，主线程难以捕获，导致事务特性难以保证。即主线程异常，如何回滚所有子线程，一个子线程异常，如何回滚主线程和其它所有子线程。经过实践，封装了一个工具类，用以保证多线程事务特性。
思路：
线程相互等待。主线程等子线程执行，如子线程有异常，主线程手动抛出，如无异常，子线程等主线程执行，如主线程有异常，回滚事务，如无，提交事务，等所有子线程执行完事务，主线程返回结果。
代码如下：

<!-- more -->

```java
@Slf4j
public class X {

    public static Map<Object,Throwable> exceptionWeakMap = Collections.synchronizedMap(new WeakHashMap<>());

    /**
     * 1.用于大量任务 for 循环开启线程
     * 2.自动阻塞，for 循环所有线程任务结束，才会继续向下执行
     * 3.支持线程报错的抛出，适用于查询报错，逻辑校验报错，不适用于增删改事务
     *
     * @param list 需要操作的集合
     * @param semapAmount 信号量（最多并发多少线程）
     * @param action 循环里执行的动作
     */
    public static <T> void asyncList(List<T> list,int semapAmount, Consumer<? super T> action){
        list = list == null ? new ArrayList<>() : list;
        CountDownLatch latch = new CountDownLatch(list.size());
        Semaphore semaphore = new Semaphore(semapAmount);
        list.forEach(t -> DoHelper.asyncCouSemProxy(semaphore, latch, () -> action.accept(t)));
        try {
            latch.await(30, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            throw new ServiceException(e);
        }
        if (exceptionWeakMap.containsKey(latch)) {
            log.error(exceptionWeakMap.get(latch).getMessage(), exceptionWeakMap.get(latch));
            throw new RuntimeException(exceptionWeakMap.get(latch).getMessage());
        }
    }

    /**
     * public int test(){
     *   Map<String, CountDownLatch> map  = new HashMap<>();
     *   try {
     *      // 业务代码……
     *      X.asyncTransacTion(map,()->{},()->{}……);
     *      // 业务代码……
     *   } catch (Exception e) {
     *      X.doCatch(e,map);
     *   } finally {
     *      X.doFinally(map);
     *   }
     *   return 0;
     * }
     *  1.只能在 try 里面写所有业务代码，catch 、finally 必须如上
     *  2.不可用于for循环内，仅用于单次传多个线程
     *  3.支持多线程事务，主线程报错，所有子线程事务回滚，一个子线程报错，其它子线程和主线程也回滚
     * @param map   一个空map
     * @param runnables  多个线程
     */
    public static void asyncTransacTion(Map<String,CountDownLatch> map,Runnable...runnables){
        CountDownLatch threadLatch = new CountDownLatch(runnables.length);
        CountDownLatch transLatch = new CountDownLatch(runnables.length);
        CountDownLatch mainLatch = new CountDownLatch(1);
        map.put("threadLatch",threadLatch);
        map.put("transLatch",transLatch);
        map.put("mainLatch",mainLatch);
        Arrays.stream(runnables).forEach(runnable -> DoHelper.asyncCoutProxy(mainLatch, threadLatch, transLatch, runnable));
        try {
            threadLatch.await(30, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            throw new ServiceException(e);
        }
        if (exceptionWeakMap.containsKey(threadLatch)) {
            throw new RuntimeException(exceptionWeakMap.get(threadLatch).getMessage());
        }
    }


    public static void doCatch(Exception e,Map<String,CountDownLatch> map){
        X.exceptionWeakMap.put(map.get("threadLatch"), e);
        throw new ServiceException(e.getMessage());
    }

    public static void doFinally(Map<String,CountDownLatch> map){
        if (map.size()>0) {
            map.get("mainLatch").countDown();
            try {
                map.get("transLatch").await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
@Slf4j
public class DoHelper {

    private static AsyncTaskExecutor asyncExecutor = (new AnnotationConfigApplicationContext(ThreadPoolConfig.class)).getBean(AsyncTaskExecutor.class);
    
    public static void asyncDo(Runnable...runnables){
        IntStream.range(0,runnables.length).forEach(i->asyncExecutor.execute(runnables[i]));
    }
    
    public static void asyncCouSemProxy(Semaphore semaphore,CountDownLatch latch, Runnable...runnables){
        IntStream.range(0,runnables.length).forEach(i->asyncDo(ThreadProxy.getInstance(semaphore,latch,runnables[i])));
    }
    
    public static void asyncCoutProxy(CountDownLatch mainLatch,CountDownLatch threadLatch,CountDownLatch transLatch,Runnable runnable){
        asyncDo(ThreadProxy.getTransacTionInstance(mainLatch,threadLatch,transLatch,runnable));
    }
}

```

```java
@Component
public class ThreadProxy {

    private static PlatformTransactionManager transactionManager;

    @Autowired
    public void setTransactionManager(PlatformTransactionManager transactionManager){
        ThreadProxy.transactionManager = transactionManager;

    }


    public static Runnable getTransacTionInstance(CountDownLatch mainLatch,CountDownLatch threadLatch,CountDownLatch transLatch,Runnable runnable) {
        return (Runnable) Proxy.newProxyInstance(
                ThreadProxy.class.getClassLoader(),
                runnable.getClass().getInterfaces(),
                (proxy, method, args) -> {
                    DefaultTransactionDefinition def = new DefaultTransactionDefinition();
                    def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
                    TransactionStatus status = transactionManager.getTransaction(def);
                    Object invoke = null;
                    try {
                        invoke = method.invoke(runnable, args);
                    }catch (InvocationTargetException e){
                        X.exceptionWeakMap.put(threadLatch,e.getTargetException());
                    }finally {
                        threadLatch.countDown();
                    }
                    mainLatch.await(10, TimeUnit.MINUTES);
                    try {
                        if (X.exceptionWeakMap.containsKey(threadLatch)) {
                            transactionManager.rollback(status);
                        } else {
                            transactionManager.commit(status);
                        }
                    }finally {
                        transLatch.countDown();
                    }
                    return invoke;
                });
    }

    public static  Runnable getInstance(Semaphore semaphore,CountDownLatch latch,Runnable runnable) {
        return (Runnable) Proxy.newProxyInstance(
                ThreadProxy.class.getClassLoader(),
                runnable.getClass().getInterfaces(),
                (proxy, method, args) -> {
                    try {
                        semaphore.acquire();
                        return method.invoke(runnable, args);
                    } catch (InvocationTargetException e) {
                        X.exceptionWeakMap.put(latch,e.getTargetException());
                    } finally {
                        semaphore.release();
                        latch.countDown();
                    }
                    return null;
                });
    }
}

```

```java
@Configuration
@EnableAsync
public class ThreadPoolConfig implements AsyncConfigurer {

    private AsyncTaskExecutor getExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(100);
        executor.setQueueCapacity(1000);
        executor.setKeepAliveSeconds(1);
        return executor;
    }

    @Primary
    @Bean("taskExecutor")
    public AsyncTaskExecutor taskExecutor(){
        return getExecutor();
    }
    @Bean("asyncExecutor")
    public AsyncTaskExecutor asyncExecutor(){
        return getExecutor();
    }
    @Bean("otherExecutor")
    public AsyncTaskExecutor otherExecutor(){
        return getExecutor();
    }


    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (throwable, method,obj)->{
            throwable.printStackTrace();
            System.out.println("Method name - " + method.getName());
            for (Object param : obj) {
                System.out.println("Parameter value - " + param);
            }
        };
    }
}

```
使用方法：直接调用 X.asyncList()、X.asyncTransacTion()，前者支持子线程报错抛到主线程，后者支持事务

```java
    @Transactional
    public Object add() throws InterruptedException {
        Map<String, CountDownLatch> map  = new HashMap<>();
        try {
        ArrayList<User> list = new ArrayList<>();
        for (int i = 0; i <19; i++) {
            User u = new User();
            u.setId(i);
            list.add(u);
        }
        X.asyncList(list,2,user -> System.out.println(user.getName()));
        X.asyncTransacTion(map,
                ()->{userManager.save(list.get(0));},
                ()->{userManager.save(list.get(1));});
        userManager.save(list.get(list.size()-1));
        }catch (Exception e){
            X.doCatch(e,map);
        }finally {
            X.doFinally(map);
        }
        return ResponseModel.successful();
    }
```
