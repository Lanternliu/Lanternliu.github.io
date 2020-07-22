 ---
 layout:    post
 title:      "ThreadLocal使用与原理"
 subtitle:   "Java基础"
 date:       2020-07-22
 author:     "liuwenzi"
 tags:
     - Java基础
 ---
### ThreadLocal使用与原理

-------

>```java
>ThreadLocals rely on per-thread linear-probe hash maps attached
>* to each thread (Thread.threadLocals and
>* inheritableThreadLocals).  The ThreadLocal objects act as keys,
>* searched via threadLocalHashCode.  This is a custom hash code
>* (useful only within ThreadLocalMaps) that eliminates collisions
>* in the common case where consecutively constructed ThreadLocals
>* are used by the same threads, while remaining well-behaved in
>* less common cases.
>  
>简单地说，ThreadLocal适用于每个线程需要自己独立的示例且该实例在多个方法或类中被使用，即变量在线程间隔离而在方法或类中共享的场景。
>```

#### ThreadLocal使用

```java
public class ThreadLocalTest implements Runnable{
    private static ThreadLocal<String> threadLocal = ThreadLocal.withInitial(()->new String(String.valueOf(System.currentTimeMillis())));
    public static void main(String[] args) throws InterruptedException {
        ThreadLocalTest test = new ThreadLocalTest();
        for (int i = 0;i<3; i++){
            new Thread(test,""+i).start();
            Thread.sleep(200);
        }
    }
    @Override
    public void run() {
        System.out.println(threadLocal.get());
    }
}	
result:
1595424239276
1595424239484
1595424239686
```

可以看出，不同的线程有不同的threadlocal值。

#### ThreadLocal原理

- 猜想：应该是ThreadLocal持有一个map，按thread存取线程的值

- Java的设计

  > Java实现中持有ThreadLocalMap的不是ThreadLocal，而是Thread，ThreadLocalMap中key的值是ThreadLocal，这样做有两个好处	

  		- ThreadLocal仅仅是一个代理工具类，内部并不应该持有任何与线程相关的数据，所有和线程相关的数据都存储在Thread里面。这样设计更加合理
    - 这样设计**不容易产生内存泄露**
      		- 如果按照我们的设计，只要ThreadLocal对象存在，那么Map中的Thread就永远不会被回收（因为有其他对象引用它），而ThreadLocal的声明周期往往比线程药厂，所以这种设计方案很容易导致内存泄露。 Java的设计可以防止内存泄露

#### ThreadLocal与内存泄露和脏数据

#### 内存泄露

> 在线程池中使用ThreadLocal容易导致内存泄露，原因是线程池中thread的生命周期往往很长，这意味着线程持有的thread持有的ThreadLocalMap不能被回收。ThreadLocalMap中的Entry对ThreadLocal是弱引用，只要ThreadLocal结束了自己的生命周期，那么ThreadLocal是可以被回收的。但是ThreadLocalMap中Entry中的value确实被Entry强引用，所以即便Value的生命周期结束也无法被回收，从而导致内存泄露。
>
> ![9hAnAL](https://cdn.jsdelivr.net/gh/Lanternliu/pic@master/uPic/9hAnAL.png)

#### 脏数据

>**使用线程池的时候，线程结束是不会销毁的，会再次使用的就可能出现内存泄露 。（在web应用中，每次http请求都是一个线程，tomcat容器配置使用线程池时会出现内存泄漏问题）**

##### 处理方法

当用完ThreadLocal之后手动释放

```java
ExecutorService es;
ThreadLocal tl;
es.execute(()->{
  //ThreadLocal增加变量
  tl.set(obj);
  try {
    // 省略业务逻辑代码
  }finally {
    //手动清理ThreadLocal 
    tl.remove();
  }
});
```

#### 总结

Tips：ThreadLocal建议使用static修饰

原因：

- *ThreadLocal 对象建议使用 static 修饰。这个变量是针对一个线程内所有操作共享的，所以设置为静态变量，所有此类实例共享 此静态变量 ，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只 要是这个线程内定义的)都可以操控这个变量。* 

  ​																															---------------摘自阿里巴巴规范手册

- 我们知道，一个ThreadLocal实例对应当前线程中的一个TSO实例。因此，如果把ThreadLocal声明为某个类的实例变量（而不是静态变量），**那么每创建一个该类的实例就会导致一个新的TSO实例被创建。**设想一下，一个线程内创建两个类实例，这个类中定义了一个非stattic的变量，并把它set到自己的map中，会发生什么，没错两个重复的object，因为两次的threadlocal不一样。这也是为什么要使用static来标识



#### 参考文章

[ThreadLocal内存泄漏问题](https://juejin.im/post/5ba9a6665188255c791b0520)

[线程本地存储模式：没有共享，就没有伤害](https://time.geekbang.org/column/article/93745)

[Java进阶（七）正确理解Thread Local的原理与适用场景](http://www.jasongj.com/java/threadlocal/)

