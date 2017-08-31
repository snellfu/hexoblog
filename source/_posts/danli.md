---
category: coding
title: Java基础与干货---单例
date: 2016-08-31 15:01:45
tags: Java
toc: true
---



单例能做那些事:

- 延迟加载
- 线程安全
- 没有性能问题
- 防止序列化产生新对象
- 防止反射攻击





### 最简单的单例---**饿汉式**
---
```
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    // 私有化构造函数
    private Singleton(){}

    public static Singleton getInstance(){
        return INSTANCE;
    }
}
```
这种单例的写法最简单，但是缺点是一旦类被加载，单例就会初始化，没有实现懒加载。而且当实现了Serializable接口后，反序列化时单例会被破坏。


实现Serializable接口需要重写 **readResolve**，才能保证其反序列化依旧是单例：

```
public class Singleton implements Serializable {
    private static final Singleton INSTANCE = new Singleton();
    // 私有化构造函数
    private Singleton(){}

    public static Singleton getInstance(){
        return INSTANCE;
    }

    /**
     * 如果实现了Serializable, 必须重写这个方法
     */
    private Object readResolve() throws ObjectStreamException {
        return INSTANCE;
    }
}
```
### 最体现技术的单例---**懒汉式**
---
懒汉式即实现延迟加载的单例，为上述饿汉式的优化形式。而因其仍需要进一步优化，往往成为面试考点。

懒汉式的最初形式是这样的：
```
public class Singleton {
    private static Singleton INSTANCE;
    private Singleton (){}

    public static Singleton getInstance() {
     if (INSTANCE == null) {
         INSTANCE = new Singleton();
     }
     return INSTANCE;
    }
}
```
这种写法就轻松实现了单例的懒加载，只有调用了**getInstance**方法才会初始化。但是这样的写法出现了新的问题--线程不安全。当多个线程调用**getInstance**方法时，可能会创建多个实例，因此需要对其进行同步。

如何使其线程安全呢？简单，加个**synchronized**关键字就行了

```
public static synchronized Singleton getInstance() {
    if (INSTANCE == null) {
        INSTANCE = new Singleton();
    }
    return INSTANCE;
}
```
可是...这样又出现了性能问题，简单粗暴的同步整个方法，导致同一时间内只有一个线程能够调用**getInstance**方法。

因为仅仅需要对初始化部分的代码进行同步，所以再次进行优化：
```
public static Singleton getSingleton() {
    if (INSTANCE == null) {               // 第一次检查
        synchronized (Singleton.class) {
            if (INSTANCE == null) {      // 第二次检查
                INSTANCE = new Singleton();
            }
        }
    }
    return INSTANCE ;
}
```
  执行两次检测很有必要：当多线程调用时，如果多个线程同时执行完了第一次检查，其中一个进入同步代码块创建了实例，后面的线程因第二次检测不会创建新实例。

这段代码看起来很完美，但仍旧存在问题，以下内容引用自黑桃夹克大神的如何正确地写出单例模式

  这段代码看起来很完美，很可惜，它是有问题。主要在于**instance = new Singleton()**这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情。
- 给 **instance** 分配内存
- 调用 **Singleton** 的构造函数来初始化成员变量
- 将**instance**对象指向分配的内存空间（执行完这步 **instance** 就为非 null 了）

但是在 JVM 的即时编译器中存在指令重排序的优化。也就是说上面的第二步和第三步的顺序是不能保证的，最终的执行顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，则在 3 执行完毕、2 未执行之前，被线程二抢占了，这时 instance 已经是非 null 了（但却没有初始化），所以线程二会直接返回 instance，然后使用，然后顺理成章地报错。

我们只需要将 **instance**变量声明成 **volatile** 就可以了。
```
public class Singleton {
    private volatile static Singleton INSTANCE; //声明成 volatile
    private Singleton (){}

    public static Singleton getSingleton() {
        if (INSTANCE == null) {                         
            synchronized (Singleton.class) {
                if (INSTANCE == null) {       
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }

}
```
使用 volatile 的主要原因是其另一个特性：禁止指令重排序优化。也就是说，在 volatile 变量的赋值操作后面会有一个内存屏障（生成的汇编代码上），读操作不会被重排序到内存屏障之前。比如上面的例子，取操作必须在执行完 1-2-3 之后或者 1-3-2 之后，不存在执行到 1-3 然后取到值的情况。从「先行发生原则」的角度理解的话，就是对于一个 **volatile**变量的写操作都先行发生于后面对这个变量的读操作（这里的“后面”是时间上的先后顺序）。

但是特别注意在 Java 5 以前的版本使用了 **volatile** 的双检锁还是有问题的。其原因是 Java 5 以前的 JMM （Java 内存模型）是存在缺陷的，即时将变量声明成 volatile 也不能完全避免重排序，主要是 **volatile** 变量前后的代码仍然存在重排序问题。这个 **volatile**屏蔽重排序的问题在 Java 5 中才得以修复，所以在这之后才可以放心使用 **volatile**。
至此，这样的懒汉式才是没有问题的懒汉式。
### 内部类实现单例
```
public class Singleton { 
    /** 
     * 类级的内部类，也就是静态的成员式内部类，该内部类的实例与外部类的实例没有绑定关系， 
     * 而且只有被调用到才会装载，从而实现了延迟加载 
     */ 
    private static class SingletonHolder{ 
        /** 
         * 静态初始化器，由JVM来保证线程安全 
         */ 
        private static final Singleton instance = new Singleton(); 
    } 
    /** 
     * 私有化构造方法 
     */ 
    private Singleton(){ 
    } 

    public static  Singleton getInstance(){ 
        return SingletonHolder.instance; 
    } 
}
```
使用内部类来维护单例的实例，当**Singleton**被加载时，其内部类并不会被初始化，故可以确保当 **Singleton**类被载入JVM时，不会初始化单例类。只有 **getInstance()** 方法调用时，才会初始化 **instance**。同时，由于实例的建立是时在类加载时完成，故天生对多线程友好，**getInstance()** 方法也无需使用同步关键字。


### 年度最佳实践单例---**枚举**
---
这货长这样：
```
public enum Singleton{
    INSTANCE;
}
```
这种方式的好处是：

1、利用的枚举的特性实现单例
2、由JVM保证线程安全
3、序列化和反射攻击已经被枚举解决

调用方式为```Singleton.INSTANCE```, 出自《Effective Java》第二版第三条: 用私有构造器或枚举类型强化Singleton属性。

[原文章](http://www.jianshu.com/p/e84529b464d3)