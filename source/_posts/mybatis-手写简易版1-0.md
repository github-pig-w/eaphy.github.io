---
title: mybatis - 手写简易版1.0
tags:
  - mybatis
categories:
  - 开源框架
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2019-07-19 23:01:06
password:
---

# 架构组成
<img src="/img/19071901.png"  align=center/>

<!-- more -->

##  sqlsession
sqlsession 是我们直接操作数据的入口，我们通过它来获取代理mapper，进而操作数据库，获取数据。其中包含了configuration、executor的引用，sqlsession 本身不做具体的事情，而是委托configuration、executor去做具体的操作。
## configuration
configuration 是一个配置类，用它来做一些框架的配置，包括初始化所有 mapper 、statement。用它来真正的获取mapper、statement。
## executor
executor 主要用来数据库的连接、执行 crud 等操作。

## proxy
proxy 主要用来生成代理mapper，我们不是通过真实的mapper接口去执行具体的业务，而主要是通过代理mapper去执行。

**主要原理：**
 1. sqlssion 持有 configuration、executor 的引用，以及获取 mapper 代理、执行 crud 的外露接口，当然，获取 mapper 代理由configuration 去具体执行，crud 操作由 executor 去具体执行，sqlsession 只是一个主管，分配任务即可。
 2. configuration 在实例化时，会初始化且保存所有 mapper 代理，以及 statement（sql语句），并且外露一个获取 mapper 代理的接口、一个获取statement的接口给 sqlsession。（mapper代理的保存方式是一个map，key是真实的mapper接口，value是该接口的代理对象；statement 的保存方式是一个map，key是 mapper 接口的方法名，value是该方法的sql语句）
 3. executor 就是建立数据库连接，外露crud接口给 sqlsession，供其差遣。
 4. proxy 通过 sqlsession 委托 configuration 获取 statment ， 委托 executor 去执行 sql 语句获取结果。（我们知道 proxy 生成的代理对象必会走invoke方法，在invoke方法中，我们可以得到mapper具体执行的哪个方法，根据该方法名获取到statment，委托给configuration、sqlsession）。


# 核心代码
## sqlsession
```java
public class SqlSession {

    private ConfiguRation configuRation;
    private Executor executor;

    public SqlSession(ConfiguRation configuRation,Executor executor){
        this.configuRation = configuRation;
        this.executor = executor;
    }

    public  ConfiguRation getConfiguRation(){
        return configuRation;
    }

    public <T> T getMapper(Class<T> clazz){
        return configuRation.getMapper(clazz,this);
    }

    public <T> T selectByPrimaryKey(String statement, String parameter){
        return executor.query(statement,parameter);
    }
}
```
## configuration

```java
public class ConfiguRation {

    private Map<Class<?> ,BatisProxyFactory> mapperHouse = new HashMap<>();
    public static final Map<String, String> mappedStatements = new HashMap<>();

    public ConfiguRation() {
        mapperHouse.put(UserMapper.class,new BatisProxyFactory(UserMapper.class));
        mappedStatements.put("com.eaphy.handBatis.UserMapper.selectByPrimaryKey"
                , "select * from t_user where id = %d");
    }

    public boolean containStatement(String statementName) {
        return mappedStatements.containsKey(statementName);
    }

    public String getMappedStatement(String id) {
        return mappedStatements.get(id);
    }

    public <T> T getMapper(Class<T> clazz, SqlSession sqlSession) {
        BatisProxyFactory proxyFactory = mapperHouse.get(clazz);
        if (proxyFactory == null) {
            throw new RuntimeException("Type: " + clazz + " can not find");
        }
        return (T)proxyFactory.newInstance(sqlSession);
    }
}
```

## executor

```java
public class Executor {

    public <T> T query(String statement, String parameter) {
        Connection conn = null;
        PreparedStatement preparedStatement = null;
        User user = null;
        try {
            conn = getConnection();
            preparedStatement = conn.prepareStatement(String.format(statement, Integer.parseInt(parameter)));
            preparedStatement.execute();
            ResultSet rs = preparedStatement.getResultSet();
            user = new User();
            while (rs.next()) {
                user.setId(rs.getInt(1));
                user.setName(rs.getString(2));
                user.setPhone(rs.getString(3));
                user.setEmail(rs.getString(4));
                user.setQq(rs.getString(5));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        return (T)user;
    }

    public Connection getConnection() throws SQLException {
        String driver = "com.mysql.cj.jdbc.Driver";
        String url = "jdbc:mysql://127.0.0.1:3306/eaphy?serverTimezone=UTC";
        String username = "root";
        String password = "123456";
        Connection conn = null;
        try {
            Class.forName(driver);
            conn = DriverManager.getConnection(url, username, password);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return conn;
    }
}
```

## proxy

```java
public class BatisProxy implements InvocationHandler {

    private SqlSession sqlSession;

    public BatisProxy(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (sqlSession.getConfiguRation().containStatement(method.getDeclaringClass().getName()+"."+method.getName())) {
            String sql = sqlSession.getConfiguRation().getMappedStatement(method.getDeclaringClass().getName()+"."+method.getName());
            return sqlSession.selectByPrimaryKey(sql, args[0].toString());
        }
        return method.invoke(proxy, args);
    }
}
```

```java
public class BatisProxyFactory<T> {

    private Class<T> mapperInterface;

    public BatisProxyFactory(Class<T> mapperInterface) {
        this.mapperInterface = mapperInterface;
    }

    public T newInstance(SqlSession sqlSession) {
        return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, new BatisProxy(sqlSession));
    }

}
```
## mapper

```java
public interface UserMapper {
    User selectByPrimaryKey(int id);
}
```
## Test

```java
@Data
public class User {
    private Integer id;
    private String name;
    private String phone;
    private String email;
    private String qq;
}
```

```java
@Slf4j
public class UserTest {

    public static void main(String[] args) {
        SqlSession sqlSession = new SqlSession(new ConfiguRation(), new Executor());
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User user = userMapper.selectByPrimaryKey(1);
        log.info("用户信息：{}",user);
    }
}
```

```java
/* 执行结果 */
用户信息：User(id=1, name=张三12233, phone=12345672233, email=1234567@qq2233.com, qq=12345672233)
```





