
# 1. 控制资源并发访问--Semaphore #

Semaphore可以理解为**信号量**，用于控制资源能够被并发访问的线程数量，以保证多个线程能够合理的使用特定资源。Semaphore就相当于一个许可证，线程需要先通过acquire方法获取该许可证，该线程才能继续往下执行，否则只能在该方法出阻塞等待。当执行完业务功能后，需要通过`release()`方法将许可证归还，以便其他线程能够获得许可证继续执行。


Semaphore可以用于做流量控制，特别是公共资源有限的应用场景，比如数据库连接。假如有多个线程读取数据后，需要将数据保存在数据库中，而可用的最大数据库连接只有10个，这时候就需要使用Semaphore来控制能够并发访问到数据库连接资源的线程个数最多只有10个。在限制资源使用的应用场景下，Semaphore是特别合适的。

下面来看下Semaphore的主要方法：

	//获取许可，如果无法获取到，则阻塞等待直至能够获取为止
	void acquire() throws InterruptedException 
	
	//同acquire方法功能基本一样，只不过该方法可以一次获取多个许可
	void acquire(int permits) throws InterruptedException
	
	//释放许可
	void release()

	//释放指定个数的许可
	void release(int permits)
	
	//尝试获取许可，如果能够获取成功则立即返回true，否则，则返回false
	boolean tryAcquire()

	//与tryAcquire方法一致，只不过这里可以指定获取多个许可
	boolean tryAcquire(int permits)

	//尝试获取许可，如果能够立即获取到或者在指定时间内能够获取到，则返回true，否则返回false
	boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException

	//与上一个方法一致，只不过这里能够获取多个许可
 	boolean tryAcquire(int permits, long timeout, TimeUnit unit)

	//返回当前可用的许可证个数
	int availablePermits()
	
	//返回正在等待获取许可证的线程数
	int getQueueLength()

	//是否有线程正在等待获取许可证
	boolean hasQueuedThreads()
	
	//获取所有正在等待许可的线程集合
	Collection<Thread> getQueuedThreads()

另外，在Semaphore的构造方法中还支持指定是够具有公平性，默认的是非公平性，这样也是为了保证吞吐量。


> 一个例子

下面用一个简单的例子来说明Semaphore的具体使用。我们来模拟这样一样场景。有一天，班主任需要班上10个同学到讲台上来填写一个表格，但是老师只准备了5支笔，因此，只能保证同时只有5个同学能够拿到笔并填写表格，没有获取到笔的同学只能够等前面的同学用完之后，才能拿到笔去填写表格。该示例代码如下：



	public class SemaphoreDemo {
	
	    //表示老师只有10支笔
	    private static Semaphore semaphore = new Semaphore(5);
	
	    public static void main(String[] args) {
	
	        //表示50个学生
	        ExecutorService service = Executors.newFixedThreadPool(10);
	        for (int i = 0; i < 10; i++) {
	            service.execute(() -> {
	                try {
	                    System.out.println(Thread.currentThread().getName() + "  同学准备获取笔......");
	                    semaphore.acquire();
	                    System.out.println(Thread.currentThread().getName() + "  同学获取到笔");
	                    System.out.println(Thread.currentThread().getName() + "  填写表格ing.....");
	                    TimeUnit.SECONDS.sleep(3);
	                    semaphore.release();
	                    System.out.println(Thread.currentThread().getName() + "  填写完表格，归还了笔！！！！！！");
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            });
	        }
	        service.shutdown();
	    }
	
	}
	输出结果：

	pool-1-thread-1  同学准备获取笔......
	pool-1-thread-1  同学获取到笔
	pool-1-thread-1  填写表格ing.....
	pool-1-thread-2  同学准备获取笔......
	pool-1-thread-2  同学获取到笔
	pool-1-thread-2  填写表格ing.....
	pool-1-thread-3  同学准备获取笔......
	pool-1-thread-4  同学准备获取笔......
	pool-1-thread-3  同学获取到笔
	pool-1-thread-4  同学获取到笔
	pool-1-thread-4  填写表格ing.....
	pool-1-thread-3  填写表格ing.....
	pool-1-thread-5  同学准备获取笔......
	pool-1-thread-5  同学获取到笔
	pool-1-thread-5  填写表格ing.....


	pool-1-thread-6  同学准备获取笔......
	pool-1-thread-7  同学准备获取笔......
	pool-1-thread-8  同学准备获取笔......
	pool-1-thread-9  同学准备获取笔......
	pool-1-thread-10  同学准备获取笔......


	pool-1-thread-4  填写完表格，归还了笔！！！！！！
	pool-1-thread-9  同学获取到笔
	pool-1-thread-9  填写表格ing.....
	pool-1-thread-5  填写完表格，归还了笔！！！！！！
	pool-1-thread-7  同学获取到笔
	pool-1-thread-7  填写表格ing.....
	pool-1-thread-8  同学获取到笔
	pool-1-thread-8  填写表格ing.....
	pool-1-thread-1  填写完表格，归还了笔！！！！！！
	pool-1-thread-6  同学获取到笔
	pool-1-thread-6  填写表格ing.....
	pool-1-thread-3  填写完表格，归还了笔！！！！！！
	pool-1-thread-2  填写完表格，归还了笔！！！！！！
	pool-1-thread-10  同学获取到笔
	pool-1-thread-10  填写表格ing.....
	pool-1-thread-7  填写完表格，归还了笔！！！！！！
	pool-1-thread-9  填写完表格，归还了笔！！！！！！
	pool-1-thread-8  填写完表格，归还了笔！！！！！！
	pool-1-thread-6  填写完表格，归还了笔！！！！！！
	pool-1-thread-10  填写完表格，归还了笔！！！！！！


根据输出结果进行分析，Semaphore允许的最大许可数为5，也就是允许的最大并发执行的线程个数为5，可以看出，前5个线程（前5个学生）先获取到笔，然后填写表格，而6-10这5个线程，由于获取不到许可，只能阻塞等待。当线程`pool-1-thread-4`释放了许可之后，`pool-1-thread-9`就可以获取到许可，继续往下执行。对其他线程的执行过程，也是同样的道理。从这个例子就可以看出，**Semaphore用来做特殊资源的并发访问控制是相当合适的，如果有业务场景需要进行流量控制，可以优先考虑Semaphore。**


# 2.线程间交换数据的工具--Exchanger #

Exchanger是一个用于线程间协作的工具类，用于两个线程间能够交换。它提供了一个交换的同步点，在这个同步点两个线程能够交换数据。具体交换数据是通过exchange方法来实现的，如果一个线程先执行exchange方法，那么它会同步等待另一个线程也执行exchange方法，这个时候两个线程就都达到了同步点，两个线程就可以交换数据。

Exchanger除了一个无参的构造方法外，主要方法也很简单：
	
	//当一个线程执行该方法的时候，会等待另一个线程也执行该方法，因此两个线程就都达到了同步点
	//将数据交换给另一个线程，同时返回获取的数据
	V exchange(V x) throws InterruptedException

	//同上一个方法功能基本一样，只不过这个方法同步等待的时候，增加了超时时间
	V exchange(V x, long timeout, TimeUnit unit)
        throws InterruptedException, TimeoutException 


> 一个例子

Exchanger理解起来很容易，这里用一个简单的例子来看下它的具体使用。我们来模拟这样一个情景，在青春洋溢的中学时代，下课期间，男生经常会给走廊里为自己喜欢的女孩子送情书，相信大家都做过这样的事情吧 ：)。男孩会先到女孩教室门口，然后等女孩出来，教室那里就是一个同步点，然后彼此交换信物，也就是彼此交换了数据。现在，就来模拟这个情景。

	public class ExchangerDemo {
	    private static Exchanger<String> exchanger = new Exchanger();
	
	    public static void main(String[] args) {
	
	        //代表男生和女生
	        ExecutorService service = Executors.newFixedThreadPool(2);
	
	        service.execute(() -> {
	            try {
	                //男生对女生说的话
	                String girl = exchanger.exchange("我其实暗恋你很久了......");
	                System.out.println("女孩儿说：" + girl);
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        });
	        service.execute(() -> {
	            try {
	                System.out.println("女生慢慢的从教室你走出来......");
	                TimeUnit.SECONDS.sleep(3);
	                //男生对女生说的话
	                String boy = exchanger.exchange("我也很喜欢你......");
	                System.out.println("男孩儿说：" + boy);
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        });
	
	    }
	}

	输出结果：

	女生慢慢的从教室你走出来......
	男孩儿说：我其实暗恋你很久了......
	女孩儿说：我也很喜欢你......

这个例子很简单，也很能说明Exchanger的基本使用。当两个线程都到达调用exchange方法的同步点的时候，两个线程就能交换彼此的数据。