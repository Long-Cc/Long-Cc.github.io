---
layout:     post
title:      面试问答(一)
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


####  四种引⽤，强，软，弱，虚的区别，软引⽤使⽤场景。
**强引用**（Strong Reference）：最常见的一种引用类型，它指的是在程序代码中普遍存在的引用，如果一个对象具有强引用，那么它永远不会被垃圾回收器回收，即使内存空间不足也不会。

```java
Object obj = new Object();
```

强引用可能导致内存泄漏，因为即使对象不再被使用，只要还有强引用指向它，垃圾回收器就无法回收这个对象。为了避免内存泄漏，可以手动将不再使用的强引用设置为null，这样垃圾回收器就可以回收该对象了：
```java
obj = null;
```

**软引用**（Soft Reference）：是一种相对于强引用而言的引用类型，它用于描述一些有用但非必需的对象。软引用可以通过`java.lang.ref.SoftReference`类来实现。当系统内存不足时，垃圾回收器会优先回收软引用指向的对象，从而为其他对象腾出内存空间。
特点：
1. 内存敏感：当JVM的内存空间不足时，垃圾回收器会回收软引用指向的对象，即使这些对象仍然有强引用指向它们。
2. 生命周期：软引用对象的生命周期比强引用对象短，因为它们更容易被垃圾回收器回收。
3. 内存溢出：如果内存空间非常紧张，即使有强引用指向对象，垃圾回收器也可能回收软引用指向的对象，以避免内存溢出。
4. 内存优化：软引用可以用于实现内存敏感的缓存，例如web服务器上的内存缓存，当内存不足时，可以回收这些缓存对象，从而为更重要的对象腾出空间。

```java
import java.lang.ref.SoftReference;

public class SoftReferenceExample {
    public static void main(String[] args) {
        // 创建一个软引用，指向一个对象
        SoftReference<Object> softRef = new SoftReference<>(new Object());

        // 获取软引用指向的对象
        Object obj = softRef.get();

        // 此时obj是可达的，因为软引用仍然持有对它的引用
        System.out.println(obj);

        // 清除软引用
        softRef.clear();

        // 再次尝试获取软引用指向的对象，此时可能已经为null
        System.out.println(softRef.get());
    }
}
```

在这个示例中，创建了一个软引用`softRef`，它指向一个新创建的对象。然后，我们通过调用`softRef.get()`来获取这个对象。接着，我们调用`softRef.clear()`来清除软引用，之后再次尝试获取对象，这时对象可能已经被垃圾回收，因此get()方法可能返回null。

需要注意的是，软引用的回收时机是不确定的，它依赖于JVM的垃圾回收策略和内存使用情况。因此，在使用软引用时，应该考虑到这一点，并做好相应的错误处理。

**弱引用**（Weak Reference）：是一种比软引用生存期更短的引用类型。弱引用通过`java.lang.ref.WeakReference`类来实现。一个对象拥有弱引用，并不妨碍垃圾回收器（Garbage Collector）对其的回收。也就是说，只要对象只有一个弱引用，那么在下一次垃圾回收时，这个对象就会被回收。
弱引用的特点包括：

1. 生命周期：弱引用所指向的对象比软引用的对象更容易被垃圾回收器回收。只要没有任何强引用指向该对象，无论内存空间是否充足，该对象都会被回收。
2. 垃圾回收：弱引用为垃圾回收提供了更多的灵活性。当垃圾回收器准备回收一个对象时，所有指向该对象的弱引用都会被自动清除。
3. 内存优化：弱引用通常用于实现内存敏感的缓存，如对一些临时对象的缓存，这些对象在内存不足时可以被回收。
4. 线程安全：与软引用一样，弱引用的使用也是线程安全的。
5. 引用队列：可以为弱引用设置一个引用队列（java.lang.ref.ReferenceQueue），当弱引用所指向的对象被垃圾回收后，这个弱引用会被放入到引用队列中，这可以用于跟踪对象的回收。

```java
import java.lang.ref.WeakReference;
import java.lang.ref.ReferenceQueue;

public class WeakReferenceExample {
    public static void main(String[] args) {
        Object object = new Object();
        ReferenceQueue<Object> queue = new ReferenceQueue<>();
        WeakReference<Object> weakReference = new WeakReference<>(object, queue);

        // 显示弱引用对象
        System.out.println("Object: " + weakReference.get());

        // 清除强引用
        object = null;

        // 强制进行垃圾回收
        System.gc();

        // 等待一段时间，让垃圾回收器有机会执行
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 由于强引用被清除，弱引用所指向的对象被垃圾回收
        // 弱引用也随之变为null
        System.out.println("Object after GC: " + weakReference.get());

        // 检查引用队列，看是否收到了弱引用
        if (queue.poll() != null) {
            System.out.println("Weak reference has been enqueued.");
        }
    }
}
```

在这个示例中，我们创建了一个对象object和一个`ReferenceQueue`，然后创建了一个指向object的弱引用`weakReference`，并将这个引用与队列关联起来。之后，我们清除了对object的强引用，强制执行了垃圾回收，并且稍作等待。由于没有强引用指向object，object被垃圾回收，弱引用`weakReference`变为null。同时，弱引用被放入了引用队列中。

需要注意的是，调用`System.gc()`并不会立即触发垃圾回收，它只是建议JVM进行垃圾回收。实际上，垃圾回收的时机由JVM的垃圾回收策略决定。此外，由于垃圾回收的非实时性，示例中的`Thread.sleep()`调用用来增加垃圾回收执行的可能性。

**虚引用**（Phantom Reference）：是一种最弱的引用类型，它通过`java.lang.ref.PhantomReference`类来实现。虚引用主要用于跟踪对象被垃圾回收的活动，比如监控对象的垃圾回收过程。与软引用和弱引用不同，虚引用必须和引用队列（java.lang.ref.ReferenceQueue）一起使用。

虚引用的主要特点包括：
1. 不阻止垃圾回收：即使存在虚引用，对象也会被垃圾回收器正常回收。
2. 无法访问引用对象：无法通过虚引用获取到引用的对象，因为`PhantomReference`的`get()`方法总是返回null。
3. 回收通知：当垃圾回收器准备回收一个对象时，所有指向该对象的虚引用都会被放入到与之关联的引用队列中。
4. 清理资源：虚引用可以用来在对象被回收后执行一些清理工作，比如释放资源。
5. 线程安全：虚引用的使用是线程安全的。

6. 生命周期：虚引用所指向的对象的生命周期是最短的，因为它们不会对对象的回收产生任何影响。

```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

public class PhantomReferenceExample {

    public static void main(String[] args) {
        Object object = new Object();
        ReferenceQueue<Object> queue = new ReferenceQueue<>();
        PhantomReference<Object> phantomReference = new PhantomReference<>(object, queue);

        // 虚引用的get方法总是返回null
        System.out.println("Object via Phantom Reference: " + phantomReference.get());

        // 清除强引用
        object = null;

        // 强制进行垃圾回收
        System.gc();

        // 等待一段时间，让垃圾回收器有机会执行
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 检查引用队列，看是否收到了虚引用
        PhantomReference<Object> polledRef = (PhantomReference<Object>) queue.poll();
        if (polledRef != null) {
            System.out.println("Phantom reference has been enqueued.");
        }
    }
}
```

创建了一个对象object，一个`ReferenceQueue`，然后创建了一个指向object的虚引用`phantomReference`，并将这个引用与队列关联起来。之后，我们清除了对object的强引用，强制执行了垃圾回收，并且稍作等待。由于没有强引用指向object，object被垃圾回收，虚引用`phantomReferenc`e被放入了引用队列中。

虚引用通常用于一些需要精细控制的内存管理场景，比如资源清理、监控对象的生命周期等。由于虚引用的`get()`方法总是返回null，因此它不能用来获取对象实例。虚引用主要用于跟踪对象被垃圾回收的活动，而不是用来直接操作对象。

##### 总结

|  引用类型   | 被回收时间  |  用途   | 生存时间  |
|  :----:  | :----:  |  :----:  | :----:  |
| **强引用**  | 从来不会 | 对象的一般状态  | `JVM`停止运行时 |
| **软引用**  | 内存不足时 | 对象缓存  | 内存不足时 |
| **弱引用**  | `jvm`垃圾回收时 | 对象缓存  | `gc`运行后 |
| **虚引用**  | 未知 | 未知  | 未知 |

**强引用**：不回收。
**软引用**：内存不够就回收。
**弱引用**：一定回收。
**虚引用**：一定回收，`get`出来就是null，引用形同虚设，主要和引用队列联合使用，在finalize之前会被放到引用队列中。