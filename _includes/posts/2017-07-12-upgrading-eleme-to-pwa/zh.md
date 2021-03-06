>在学习java的过程中会遇到各个各样锁的概念：公平锁/非公平锁、可重入锁、单独锁/共享锁、互斥锁/读写锁、乐观锁/悲观锁、分段锁、偏向锁/轻量级锁/重量级锁、自旋锁、闭锁、活锁，当然最高名的锁就是无锁，也是高并发过程中的上层武功
>在这些概念中，有的指锁的状态或类型， 有的指锁的设计，在这里整理并记录一下：

#### 一、公平锁/非公平锁  
1.公平锁是指按多个线程申请锁的顺序来获取锁。
2.非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。
比如：对于Java ReentrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。
对于Synchronized而言，也是一种非公平锁。由于其并不像ReentrantLock是通过AQS(AbstractQueuedSynchronized 抽象的队列式的同步器 ReentrantLock，Semaphore，CountDownLatch，ReentrantReadWriteLock，FutureTask)的来实现线程调度，所以并没有任何办法使其变成公平锁。

#### 二、可重入锁  
可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁
ReentrantLock、Synchronized都是可重入锁

#### 三、独享锁/共享锁  
1.独享锁是指该锁一次只能被一个线程所持有。
2.共享锁是指该锁可被多个线程所持有。
对于ReentrantLock而言，其是独享锁。但是对于Lock的另一个实现类ReadWriteLock，其读锁是共享锁，其写锁是独享锁。
读锁的共享锁可保证并发读是非常高效的，读写，写读 ，写写的过程是互斥的。
独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。
对于Synchronized而言，当然是独享锁。

#### 四、互斥锁/读写锁  
上面讲的独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。互斥锁在Java中的具体实现就是ReentrantLock
读写锁在Java中的具体实现就是ReadWriteLock

#### 五、乐观锁/悲观锁  
不是具体类型的锁，而是指看待并发同步的角度
悲观锁在Java中的使用，就是利用各种锁。
乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。

#### 六、分段锁  
分段锁其实是一种锁的设计，并不是具体的一种锁，对于ConcurrentHashMap而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作，目的在于细化锁粒度。
我们以ConcurrentHashMap来说一下分段锁的含义以及设计思想，ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap（JDK7与JDK8中HashMap的实现）的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock（Segment继承了ReentrantLock)。

#### 七、偏向锁、轻量级锁、重量级锁  
这三种锁是指锁的状态，并且是针对Synchronized。在Java 5通过引入锁升级的机制来实现高效Synchronized。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。
轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。

#### 八、自旋锁  
在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。

#### 九、闭锁  
闭锁是一种同步工具类，可以延迟线程的进度直到其到达终止状态。闭锁的作用相当于一扇门：在闭锁到达结束状态之前，这扇门一直是关闭的，并且没有任何线程能通过，当到达结束状态时，这扇门会打开允许所有的线程通过。当闭锁到达结束状态后，将不会再改变状态，因此这扇门将永远保持打开状态。闭锁可以用来确保某些活动指导其他活动都完成后才继续执行。CountDownLatch就是一种灵活的闭锁实现。

#### 十、活锁  
LiveLock是一种形式活跃性问题，该问题尽管不会阻塞线程，但也不能继续执行，因为线程将不断重复执行相同的操作，而且总会失败。活锁通常发送在处理事务消息的应用程序中：如果不能成功地处理某个消息，那么消息处理机制将回滚整个事务，并将它重新放到队列的开头：如果不能成功地处理某个消息，那么消息处理机制将回滚整个事务，并将它重新放到队列的开头。如果消息处理器在处理某种特定类型的消息时存在错误并导致它失败，那么每当这个消息从队列中取出并传递到存在错误的处理器时，都会发生事务回滚。由于这条消息又被放回到队列开头，因此处理器将被反复调用，并返回相同的结果。

#### 十一、无锁  
要保证现场安全，并不是一定就要进行同步，两者没有因果关系。同步只是保证共享数据争用时的正确性的手段，如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性，因此会有一些代码天生就是线程安全的。
无状态编程。无状态代码有一些共同的特征：不依赖于存储在对上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非无状态的方法等。可以参考Servlet。
线程本地存储。可以参考ThreadLocal
volatile
CAS
协程：在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换。


[1]: https://twitter.com/vuejs/status/834087199008239619
[2]: https://developers.google.com/web/progressive-web-apps/
[3]: https://blog.twitter.com/2017/how-we-built-twitter-lite
[4]: https://medium.com/progressive-web-apps/building-flipkart-lite-a-progressive-web-app-2c211e641883
[5]: https://medium.com/engineering-housing/progressing-mobile-web-fac3efb8b454
[6]: https://shop.polymer-project.org/
[7]: https://developers.google.com/web/fundamentals/performance/prpl-pattern/
[8]: https://calendar.perfplanet.com/2013/big-bad-preloader/
[9]: https://w3c.github.io/ServiceWorker/v1/
[10]: https://webpack.github.io/
[11]: https://medium.com/@Huxpro/how-does-sw-precache-works-2d99c3d3c725
[12]: https://developers.google.com/web/updates/2015/11/app-shell
[13]: https://googlechrome.github.io/sw-toolbox/

