---
layout:     post
title:      面试内容
subtitle:   面经 问答
date:       2024-05-08
author:     Belial
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 面试
    - Android
    - flutter
---

# 求职面试必问

#### == 和equals的区别
Java中的8种基本数据类型（byte,short,char,int,long,float,double,boolean）比较他们之间的值是否相等。
引用数据类型，比较的是他们在堆内存地址是否相等。每新new一个引用类型的对象，会重新分配堆内存空间，使用==比较返回`false`。

`equals`方法是`Object`类的一个方法，`Java`当中所有的类都是继承于Object这个超类。
`JDK1.8 Object`类`equals`方法源码如下，即返回结果取决于两个对象的使用`==`判断结果。
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
在实际使用中，一般会重写定义的class的`equals`方法，如JDK1.8 java.lang.String类的equals源码如下。

即两个字符串使用` == `相等  或者  两个字符串的所有组成字符都相等返回`true`，其他情况返回`false`。这里就定义`String`根据`equals`方法判断是否相等的逻辑。
```java
public boolean equals(Object anObject) {
	if (this == anObject) {
		return true;
	}
	if (anObject instanceof String) {
		String anotherString = (String) anObject;
		int n = value.length;
		if (n == anotherString.value.length) {
			char v1[] = value;
			char v2[] = anotherString.value;
			int i = 0;
			while (n-- != 0) {
				if (v1[i] != v2[i])
						return false;
				i++;
			}
			return true;
		}
	}
	return false;
}
```
`==` 的作用：
　　基本类型：比较值是否相等
　　引用类型：比较内存地址值是否相等

`equals` 的作用:
　　引用类型：默认情况下，比较内存地址值是否相等。可以按照需求逻辑，重写对象的equals方法。

#### 线程安全的单例模式，DCL为什么是两次判断

名称解释：DCL： Double Check Lock 双重锁定检查
在双重检查锁定实现中，实例的创建过程会被划分为两个部分。首先，检查实例是否已经创建。如果已经创建，则直接返回该实例。否则，进入第二个检查，即在实例的创建过程中使用互斥锁。这样，只有一个线程会进入到实例的创建过程，从而确保了线程安全。

```java
public class Singleton {
    // volatile 防止指令重排
    private volatile static Singleton singleton = null;
    
    private Singleton() {
    }

    public static Singleton getInstance() {
        if (null == singleton) {
            synchronized (Singleton.class) {
                if (null == singleton) { //DCL
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```
####  为什么设计双亲委派机制
类加载机制
一个类从加载到使用到卸载一共经过了5个步骤:
加载 -> 连接 -> 初始化

双亲委派机制（Parent-Delegate Model）是Java类加载器中采用的一种类加载策略。该机制的核心思想是：如果一个类加载器收到了类加载请求，默认先将该请求委托给其父类加载器处理。只有当父级加载器无法加载该类时，才会尝试自行加载。

##### 类加载器与层级关系
Java中的类加载器主要有如下三种：
![alt text](/img/image.png)
**启动类加载器**（Bootstrap ClassLoader）： 负责加载 %JAVA_HOME%/jre/lib 目录下的核心Java类库如 rt.jar、charsets.jar 等。
**扩展类加载器**（Extension ClassLoader）： 负责加载 %JAVA_HOME%/jre/lib/ext 目录下的扩展类库。
**应用类加载器**（Application ClassLoader）： 负责加载用户类路径（ClassPath）下的应用程序类。

1. 通过双亲委派机制，可以避免类的重复加载，当父加载器已经加载过某一个类时，子加载器就不会再重新加载这个类。

2. 通过双亲委派机制，可以保证安全性。因为BootstrapClassLoader在加载的时候，只会加载JAVA_HOME中的jar包里面的类，如java.lang.String，那么这个类是不会被随意替换的。

那么，就可以避免有人自定义一个有破坏功能的java.lang.String被加载。这样可以有效的防止核心Java API被篡改。

##### 双亲委派机制的优缺点
优点：

避免重复加载：由于类加载器直接从父类加载器那里加载类，避免了类的重复加载。
提高安全性：通过双亲委派模型，Java 标准库中的核心类库（如 java.lang.*）由启动类加载器加载，这样能保证这些核心类库不会被恶意代码篡改或替换，从而提高程序的安全性。
保持类加载的一致性：这种方式确保了同一个类的加载由同一个类加载器完成，从而在运行时保证了类型的唯一性和相同性。这也有助于减轻类加载器在处理相互关联的类时的复杂性。

缺点：

灵活性降低：由于类加载的过程需要不断地委托给父类加载器，这种机制可能导致实际应用中类加载的灵活性降低。
增加了类加载时间：在类加载的过程中，需要不断地查询并委托父类加载器，这意味着类加载所需要的时间可能会增加。在类数量庞大或类加载器层次比较深的情况下，这种时间延迟可能会变得更加明显。

                        
原文链接：https://blog.csdn.net/weixin_45716968/article/details/132543500