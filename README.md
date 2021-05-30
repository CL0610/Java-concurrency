
> 努力的意义，就是，在以后的日子里，放眼望去全是自己喜欢的人和事！

欢迎提issue和Pull request。所有的文档都是自己亲自码的，如果觉得不错，欢迎给star鼓励支持 :)


整个系列文章为**Java并发专题**，一是自己的兴趣，二是，这部分在实际理解上很有难度，另外在面试过程中也是经常被问到。所以在学习过程中，记录了Java并发相关的基础知识，一是自己对知识能够建立体系，同时也希望有幸能够对其他人有用。


关于Java并发专题：


（1）包含了并发的基础知识，每个标题链接到一篇具体的文章；

（2）包含了秋招面试的问题，弄懂了会让你有所收获（也祝大家都能找到心仪的工作 :) ）

（3）在阅读过程中，如果有所帮助，麻烦点赞，算是对我码字的这份坚持的鼓励。

**注：转载请标明原处，谢谢！**

1. **基础知识**
	
	1.1 [并发编程的优缺点](https://juejin.im/post/5ae6c3ef6fb9a07ab508ac85)
	
	知识点：（1）为什么要用到并发？（优点）；（2）并发编程的缺点；（3）易混淆的概念

	1.2 [线程的状态和基本操作](https://juejin.im/post/5ae6cf7a518825670960fcc2)

	知识点：（1）如何新建线程；（2）线程状态的转换；（3）线程的基本操作；（4）守护线程Daemon；

2. **并发理论（JMM）**

	[java内存模型以及happens-before规则](https://juejin.im/post/5ae6d309518825673123fd0e)

	知识点：（1）JMM内存结构；（2）重排序；（3）happens-before规则


3. **并发关键字**

	3.1 [让你彻底理解Synchronized](https://juejin.im/post/5ae6dc04f265da0ba351d3ff)

	知识点：（1）如何使用synchronized；（2）monitor机制；（3）synchronized的happens-before关系；（4）synchronized的内存语义；（5）锁优化；（6）锁升级策略

	3.2 [让你彻底理解volatile](https://juejin.im/post/5ae9b41b518825670b33e6c4)
	
	知识点：（1）实现原理；（2）happens-before的关系推导；（3）内存语义；（4）内存语义的实现

	3.3 [你以为你真的了解final吗？](https://juejin.im/post/5ae9b82c6fb9a07ac3634941)

	知识点：（1）如何使用；（2）final的重排序规则；（3）final实现原理；（4）final引用不能从构造函数中“溢出”（this逃逸）
	
	3.4 [三大性质总结：原子性，有序性，可见性](https://juejin.im/post/5aeb022cf265da0b722af7b8)
	
	知识点：（1）原子性：synchronized；（2）可见性：synchronized，volatile；（3）有序性：synchronized，volatile

4. **Lock体系**

	4.1 [初识Lock与AbstractQueuedSynchronizer(AQS)](https://juejin.im/post/5aeb055b6fb9a07abf725c8c)
	
	知识点：（1）Lock和synchronized的比较；（2）AQS设计意图；（3）如何使用AQS实现自定义同步组件；（4）可重写的方法；（5）AQS提供的模板方法；

	4.2 [深入理解AbstractQueuedSynchronizer(AQS)](https://juejin.im/post/5aeb07ab6fb9a07ac36350c8)

	知识点：（1）AQS同步队列的数据结构；（2）独占式锁；（3）共享式锁；

	4.3 [再一次理解ReentrantLock](https://juejin.im/post/5aeb0a8b518825673a2066f0)
	
	知识点：（1）重入锁的实现原理；（2）公平锁的实现原理；（3）非公平锁的实现原理；（4）公平锁和非公平锁的比较

	4.4 [深入理解读写锁ReentrantReadWriteLock](https://juejin.im/post/5aeb0e016fb9a07ab7740d90)

	知识点：（1）如何表示读写状态；（2）WriteLock的获取和释放；（3）ReadLock的获取和释放；（4）锁降级策略；（5）生成Condition等待队列；（6）应用场景

	4.5 [详解Condition的await和signal等待/通知机制](https://juejin.im/post/5aeea5e951882506a36c67f0)

	知识点：（1）与Object的wait/notify机制相比具有的特性；（2）与Object的wait/notify相对应的方法；（3）底层数据结构；（4）await实现原理；（5）signal/signalAll实现原理；（6）await和signal/signalAll的结合使用；

	4.6 [LockSupport工具](https://juejin.im/post/5aeed27f51882567336aa0fa)

	知识点：（1）主要功能；（2）与synchronized阻塞唤醒相比具有的特色；


5. **并发容器**

	5.1 [并发容器之ConcurrentHashMap(JDK 1.8版本)](https://juejin.im/post/5aeeaba8f265da0b9d781d16)

	知识点：（1）关键属性；（2）重要内部类；（3）涉及到的CAS操作；（4）构造方法；（5）put执行流程；（6）get执行流程；（7）扩容机制；（8）用于统计size的方法的执行流程；（9）1.8版本的ConcurrentHashMap与之前版本的比较

	5.2 [并发容器之CopyOnWriteArrayList](https://juejin.im/post/5aeeb55f5188256715478c21)

	知识点：（1）实现原理；（2）COW和ReentrantReadWriteLock的区别；（3）应用场景；（4）为什么具有弱一致性；（5）COW的缺点；

	5.3 [并发容器之ConcurrentLinkedQueue](https://juejin.im/post/5aeeae756fb9a07ab11112af)

	知识点：（1）实现原理；（2）数据结构；（3）核心方法；（4）HOPS延迟更新的设计意图

	5.4 [并发容器之ThreadLocal](https://juejin.im/post/5aeeb22e6fb9a07aa213404a)
	
	知识点：（1）实现原理；（2）set方法原理；（3）get方法原理；（4）remove方法原理；（5）ThreadLocalMap
	
	[一篇文章，从源码深入详解ThreadLocal内存泄漏问题](https://www.jianshu.com/p/dde92ec37bd1)
	
	知识点：（1）ThreadLocal内存泄漏原理；（2）ThreadLocal的最佳实践；（3）应用场景

	5.5 [并发容器之BlockingQueue](https://juejin.im/post/5aeebd02518825672f19c546)

	知识点：（1）BlockingQueue的基本操作；（2）常用的BlockingQueue；

	[并发容器之ArrayBlockingQueue和LinkedBlockingQueue实现原理详解](https://juejin.im/post/5aeebdb26fb9a07aa83ea17e)

6. **线程池（Executor体系）**

	6.1 [线程池实现原理](https://juejin.im/post/5aeec0106fb9a07ab379574f)

	知识点：（1）为什么要用到线程池？（2）执行流程；（3）构造器各个参数的意义；（4）如何关闭线程池；（5）如何配置线程池；

	6.2 [线程池之ScheduledThreadPoolExecutor](https://juejin.im/post/5aeec106518825670a10328a)
	
	知识点：（1）类结构；（2）常用方法；（3）ScheduledFutureTask；（3）DelayedWorkQueue;

	6.3 [FutureTask基本操作总结](https://juejin.im/post/5aeec249f265da0b886d5101)

	知识点：（1）FutureTask的几种状态；（2）get方法；（3）cancel方法；（4）应用场景；（5）实现 Runnable接口

7. **原子操作类**

	7.1  [Java中atomic包中的原子操作类总结](https://juejin.im/post/5aeec351518825670a103292)
	
	知识点：（1）实现原理；（2）原子更新基本类型；（3）原子更新数组类型；（4）原子更新引用类型；（5）原子更新字段类型

8. **并发工具**

	8.1 [大白话说java并发工具类-CountDownLatch，CyclicBarrier](https://juejin.im/post/5aeec3ebf265da0ba76fa327)

	知识点：（1）倒计时器CountDownLatch；（2）循环栅栏CyclicBarrier；（3）CountDownLatch与CyclicBarrier的比较

	8.2 [大白话说java并发工具类-Semaphore，Exchanger](https://juejin.im/post/5aeec49b518825673614d183)

	知识点：（1）资源访问控制Semaphore；（2）数据交换Exchanger

9. **并发实践**

	9.1 [一篇文章，让你彻底弄懂生产者--消费者问题](https://juejin.im/post/5aeec675f265da0b7c072c56)


> JAVA并发知识图谱

**可移动到新窗口，放大查看效果更好或者查看原图**

[知识图谱原图链接，如果有用，可克隆给自己使用](https://www.processon.com/view/5ab5a979e4b0a248b0e026b3?fromnew=1)

![JAVA并发知识图谱.png](https://github.com/CL0610/Java-concurrency/blob/master/Java并发知识图谱.png)
