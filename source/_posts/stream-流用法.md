---
title: 'stream 流用法 '
tags:
  - stream
categories:
  - java
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2019-10-27 14:52:11
password:
---

## 准备
初始一个集合对象，以供测试：

```java
    List<User> list = new ArrayList<>();
    list.add(new User("张三", 12));
    list.add(new User("李四", 16));
    list.add(new User("王五", 18));
    list.add(new User("赵六", 20));
```

## filter 

```java
    list.stream().filter(u -> u.getAge() >= 18).collect(Collectors.toList());
```

```java
	User{name='王五',  age=18}
	User{name='赵六',  age=20}
```

<!-- more -->

## map 

```java
    list.stream().map(User::getName).collect(Collectors.toList());
```
```java
	张三
	李四
	王五
	赵六
```

## sorted

```java
    list.stream().sorted(Comparator.comparing(User::getAge)).collect(Collectors.toList());

    list.stream().sorted(Comparator.comparing(User::getAge).reversed()).collect(Collectors.toList());
```

```java
	User{name='张三',  age=12}
	User{name='李四',  age=16}
	User{name='王五',  age=18}
	User{name='赵六',  age=20}
	
	
	User{name='赵六',  age=20}
	User{name='王五',  age=18}
	User{name='李四',  age=16}
	User{name='张三',  age=12}
```

## list	转 map

```java
    list.stream().collect(Collectors.toMap(User::getAge, Function.identity()));
    
    list.stream().collect(Collectors.toMap(User::getAge,Function.identity(),(a,b)->b));
```

```java
	{
	16=User{name='李四',  age=16}, 
	18=User{name='王五',  age=18}, 
	20=User{name='赵六',  age=20}, 
	12=User{name='张三',  age=12}
	}
	{
	16=User{name='李四',  age=16}, 
	18=User{name='王五',  age=18}, 
	20=User{name='赵六',  age=20}, 
	12=User{name='张三',  age=12}
	}
```


## IntStream
  \> `range(a,b)`, 包头不包尾，相当于  `(int i = a;i<b;i++)`;< >`rangeClosed(a,b)`,包头包尾，相当于  `(int i = a;i<=b;i++)`<
```java
    IntStream.range(0,list.size()).forEach(i->System.out.println(list.get(i).getName()));
    
    IntStream.rangeClosed(1,list.size()-1).forEach(i->System.out.println(list.get(i).getName()));
```

```java
	张三
	李四
	王五
	赵六
	
	李四
	王五
	赵六
```



