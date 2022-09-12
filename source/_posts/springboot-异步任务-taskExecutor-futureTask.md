---
title: springboot 异步任务(taskExecutor+FutureTask)
tags:
  - futuretask
  - taskexecutor
  - springboot
  - java
categories:
  - 多线程
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2019-07-08 22:11:03
password:
---

# 不使用异步提交任务

```java
@Service
public class UserManager {
    // 模拟耗时业务
    public int sleep1() throws InterruptedException {
        Thread.sleep(2000);
        return 2;
    }
    // 模拟耗时业务
    public int sleep2() throws InterruptedException {
        Thread.sleep(3000);
        return 3;
    }
}
```

<!-- more -->


```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Resource
    private UserManager userManager;

    @GetMapping("/noExecutor")
    public Object noExecutor() throws InterruptedException {
        long a =System.currentTimeMillis();
        sleep1();
        sleep2();
        long b = System.currentTimeMillis();

        JSONObject object = new JSONObject();
        object.put("同步耗时",b-a);
        return object;
    }
 }
```

```java
结果： {"同步耗时":5001}
```

# 使用 TaskExecutor 手动完成异步任务
在springboot启动文件中，添加 `@EnableAsync` 注解
```java
@EnableAsync
@SpringBootApplication
public class LearnningApplication {
    public static void main(String[] args) {
        SpringApplication.run(LearnningApplication.class, args);
    }
}

```
新建一个线程池配置类

```java
@Configuration
@EnableAsync
public class ThreadPoolConfig {

    private ThreadPoolTaskExecutor getExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(100);
        executor.setQueueCapacity(10);
        executor.setKeepAliveSeconds(60);
        return executor;
    }
    // 这里新建了两个 ThreadPoolTaskExecutor ，根据业务需求更改
    @Primary
    @Bean("taskExecutor")
    public ThreadPoolTaskExecutor taskExecutor(){
        return getExecutor();
    }
    @Bean("otherExecutor")
    public ThreadPoolTaskExecutor otherExecutor(){
        return getExecutor();
    }
}
```
在 controller 测试

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Resource(name="taskExecutor")
    private ThreadPoolTaskExecutor taskExecutor;

    @Resource
    private UserManager userManager;

    @GetMapping("/taskExecutor")
    public Object taskExecutor() throws Exception {
        long a =System.currentTimeMillis();
        CountDownLatch countDownLatch = new CountDownLatch(2);
        taskExecutor.execute(()->{
            try {
                userManager.sleep1();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            countDownLatch.countDown();
        });

        taskExecutor.execute(()->{
            try {
                userManager.sleep2();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            countDownLatch.countDown();
        });
        countDownLatch.await();
        long b = System.currentTimeMillis();

        JSONObject object = new JSONObject();
        object.put("异步耗时",b-a);
        return object;
    }
}
```

```java
 结果： {"异步耗时":3002}
```

# 使用 TaskExecutor 注解完成异步任务
只需要将 service 层的方法加上 `@Async("taskExecutor")` 注解
```java
@Service
public class UserManager {
    // 模拟耗时业务
    @Async("taskExecutor")
    public void sleep3() throws InterruptedException {		//必须无返回值，或者返回future
        Thread.sleep(3000);
    }
    // 模拟耗时业务
    @Async("taskExecutor")
    public void sleep4() throws InterruptedException {
        Thread.sleep(3000);
    }
}
```
```java
    @GetMapping("/anoTaskExecutor")
    public Object anoTaskExecutor() throws Exception {
        long a =System.currentTimeMillis();
        userManager.sleep3();
        userManager.sleep4();
        long b = System.currentTimeMillis();

        JSONObject object = new JSONObject();
        object.put("异步耗时",b-a);
        return object;
    }
```
```java
  结果： {"异步耗时":1}
```
# 使用 FutureTask 完成异步任务

```java
    @GetMapping("/futureTask")
    public Object futureTask() throws ExecutionException, InterruptedException {
        long a =System.currentTimeMillis();
        List<FutureTask<Integer>> list = new ArrayList<>();

        FutureTask<Integer> futureTask1 = new FutureTask<>(()->{
            return userManager.sleep1();
        });
        list.add(futureTask1);
        FutureTask<Integer> futureTask2 = new FutureTask<>(()->{
            return userManager.sleep2();
        });
        list.add(futureTask2);

        list.parallelStream().forEach(futureTask->{
            taskExecutor.submit(futureTask);
        });

        int i =0;
        for (FutureTask<Integer> futureTask:list) {
            i+=futureTask.get();    //获取异步任务结果
        }
        long b = System.currentTimeMillis();
        JSONObject object = new JSONObject();
        object.put("耗时",b-a);
        object.put("结果",i);
        return object;
    }
```

```java
结果： {"耗时":3001,"结果":5}
```




