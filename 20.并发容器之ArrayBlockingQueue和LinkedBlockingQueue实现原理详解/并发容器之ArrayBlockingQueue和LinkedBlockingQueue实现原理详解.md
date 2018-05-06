
# 1. ArrayBlockingQueue简介 #

在多线程编程过程中，为了业务解耦和架构设计，经常会使用并发容器用于存储多线程间的共享数据，这样不仅可以保证线程安全，还可以简化各个线程操作。例如在“生产者-消费者”问题中，会使用阻塞队列（BlockingQueue）作为数据容器，关于BlockingQueue可以[看这篇文章](https://juejin.im/post/5aeebd02518825672f19c546)。为了加深对阻塞队列的理解，唯一的方式是对其实验原理进行理解，这篇文章就主要来看看ArrayBlockingQueue和LinkedBlockingQueue的实现原理。

# 2. ArrayBlockingQueue实现原理 #

阻塞队列最核心的功能是，能够可阻塞式的插入和删除队列元素。当前队列为空时，会阻塞消费数据的线程，直至队列非空时，通知被阻塞的线程；当队列满时，会阻塞插入数据的线程，直至队列未满时，通知插入数据的线程（生产者线程）。那么，多线程中消息通知机制最常用的是lock的condition机制，关于condition可以[看这篇文章的详细介绍](https://juejin.im/post/5aeea5e951882506a36c67f0)。那么ArrayBlockingQueue的实现是不是也会采用Condition的通知机制呢？下面来看看。

## 2.1 ArrayBlockingQueue的主要属性 

ArrayBlockingQueue的主要属性如下:

	/** The queued items */
	final Object[] items;
	
	/** items index for next take, poll, peek or remove */
	int takeIndex;
	
	/** items index for next put, offer, or add */
	int putIndex;
	
	/** Number of elements in the queue */
	int count;
	
	/*
	 * Concurrency control uses the classic two-condition algorithm
	 * found in any textbook.
	 */
	
	/** Main lock guarding all access */
	final ReentrantLock lock;
	
	/** Condition for waiting takes */
	private final Condition notEmpty;
	
	/** Condition for waiting puts */
	private final Condition notFull;

从源码中可以看出ArrayBlockingQueue内部是采用数组进行数据存储的（`属性items`），为了保证线程安全，采用的是`ReentrantLock lock`，为了保证可阻塞式的插入删除数据利用的是Condition，当获取数据的消费者线程被阻塞时会将该线程放置到notEmpty等待队列中，当插入数据的生产者线程被阻塞时，会将该线程放置到notFull等待队列中。而notEmpty和notFull等中要属性在构造方法中进行创建：

	public ArrayBlockingQueue(int capacity, boolean fair) {
	    if (capacity <= 0)
	        throw new IllegalArgumentException();
	    this.items = new Object[capacity];
	    lock = new ReentrantLock(fair);
	    notEmpty = lock.newCondition();
	    notFull =  lock.newCondition();
	}

接下来，主要看看可阻塞式的put和take方法是怎样实现的。

## 2.2 put方法详解

` put(E e)`方法源码如下：

	public void put(E e) throws InterruptedException {
	    checkNotNull(e);
	    final ReentrantLock lock = this.lock;
	    lock.lockInterruptibly();
	    try {
			//如果当前队列已满，将线程移入到notFull等待队列中
	        while (count == items.length)
	            notFull.await();
			//满足插入数据的要求，直接进行入队操作
	        enqueue(e);
	    } finally {
	        lock.unlock();
	    }
	}


该方法的逻辑很简单，当队列已满时（`count == items.length`）将线程移入到notFull等待队列中，如果当前满足插入数据的条件，就可以直接调用` enqueue(e)`插入数据元素。enqueue方法源码为：

	private void enqueue(E x) {
	    // assert lock.getHoldCount() == 1;
	    // assert items[putIndex] == null;
	    final Object[] items = this.items;
		//插入数据
	    items[putIndex] = x;
	    if (++putIndex == items.length)
	        putIndex = 0;
	    count++;
		//通知消费者线程，当前队列中有数据可供消费
	    notEmpty.signal();
	}

enqueue方法的逻辑同样也很简单，先完成插入数据，即往数组中添加数据（`items[putIndex] = x`），然后通知被阻塞的消费者线程，当前队列中有数据可供消费（`notEmpty.signal()`）。

## 2.3 take方法详解 

take方法源码如下：


	public E take() throws InterruptedException {
	    final ReentrantLock lock = this.lock;
	    lock.lockInterruptibly();
	    try {
			//如果队列为空，没有数据，将消费者线程移入等待队列中
	        while (count == 0)
	            notEmpty.await();
			//获取数据
	        return dequeue();
	    } finally {
	        lock.unlock();
	    }
	}

take方法也主要做了两步：1. 如果当前队列为空的话，则将获取数据的消费者线程移入到等待队列中；2. 若队列不为空则获取数据，即完成出队操作`dequeue`。dequeue方法源码为：

	private E dequeue() {
	    // assert lock.getHoldCount() == 1;
	    // assert items[takeIndex] != null;
	    final Object[] items = this.items;
	    @SuppressWarnings("unchecked")
		//获取数据
	    E x = (E) items[takeIndex];
	    items[takeIndex] = null;
	    if (++takeIndex == items.length)
	        takeIndex = 0;
	    count--;
	    if (itrs != null)
	        itrs.elementDequeued();
	    //通知被阻塞的生产者线程
		notFull.signal();
	    return x;
	}

dequeue方法也主要做了两件事情：1. 获取队列中的数据，即获取数组中的数据元素（`(E) items[takeIndex]`）；2. 通知notFull等待队列中的线程，使其由等待队列移入到同步队列中，使其能够有机会获得lock，并执行完成功退出。

从以上分析，可以看出put和take方法主要是通过condition的通知机制来完成可阻塞式的插入数据和获取数据。在理解ArrayBlockingQueue后再去理解LinkedBlockingQueue就很容易了。


# 3. LinkedBlockingQueue实现原理 #
LinkedBlockingQueue是用链表实现的有界阻塞队列，当构造对象时为指定队列大小时，队列默认大小为`Integer.MAX_VALUE`。从它的构造方法可以看出：

	public LinkedBlockingQueue() {
	    this(Integer.MAX_VALUE);
	}


# 3.1 LinkedBlockingQueue的主要属性 #


LinkedBlockingQueue的主要属性有：

	/** Current number of elements */
	private final AtomicInteger count = new AtomicInteger();
	
	/**
	 * Head of linked list.
	 * Invariant: head.item == null
	 */
	transient Node<E> head;
	
	/**
	 * Tail of linked list.
	 * Invariant: last.next == null
	 */
	private transient Node<E> last;
	
	/** Lock held by take, poll, etc */
	private final ReentrantLock takeLock = new ReentrantLock();
	
	/** Wait queue for waiting takes */
	private final Condition notEmpty = takeLock.newCondition();
	
	/** Lock held by put, offer, etc */
	private final ReentrantLock putLock = new ReentrantLock();
	
	/** Wait queue for waiting puts */
	private final Condition notFull = putLock.newCondition();

可以看出与ArrayBlockingQueue主要的区别是，LinkedBlockingQueue在插入数据和删除数据时分别是由两个不同的lock（`takeLock`和`putLock`）来控制线程安全的，因此，也由这两个lock生成了两个对应的condition（`notEmpty`和`notFull`）来实现可阻塞的插入和删除数据。并且，采用了链表的数据结构来实现队列，Node结点的定义为：

	static class Node<E> {
	    E item;
	
	    /**
	     * One of:
	     * - the real successor Node
	     * - this Node, meaning the successor is head.next
	     * - null, meaning there is no successor (this is the last node)
	     */
	    Node<E> next;
	
	    Node(E x) { item = x; }
	}

接下来，我们也同样来看看put方法和take方法的实现。

## 3.2 put方法详解 ##

put方法源码为:

	public void put(E e) throws InterruptedException {
	    if (e == null) throw new NullPointerException();
	    // Note: convention in all put/take/etc is to preset local var
	    // holding count negative to indicate failure unless set.
	    int c = -1;
	    Node<E> node = new Node<E>(e);
	    final ReentrantLock putLock = this.putLock;
	    final AtomicInteger count = this.count;
	    putLock.lockInterruptibly();
	    try {
	        /*
	         * Note that count is used in wait guard even though it is
	         * not protected by lock. This works because count can
	         * only decrease at this point (all other puts are shut
	         * out by lock), and we (or some other waiting put) are
	         * signalled if it ever changes from capacity. Similarly
	         * for all other uses of count in other wait guards.
	         */
			//如果队列已满，则阻塞当前线程，将其移入等待队列
	        while (count.get() == capacity) {
	            notFull.await();
	        }
			//入队操作，插入数据
	        enqueue(node);
	        c = count.getAndIncrement();
			//若队列满足插入数据的条件，则通知被阻塞的生产者线程
	        if (c + 1 < capacity)
	            notFull.signal();
	    } finally {
	        putLock.unlock();
	    }
	    if (c == 0)
	        signalNotEmpty();
	}

put方法的逻辑也同样很容易理解，可见注释。基本上和ArrayBlockingQueue的put方法一样。take方法的源码如下：

	public E take() throws InterruptedException {
	    E x;
	    int c = -1;
	    final AtomicInteger count = this.count;
	    final ReentrantLock takeLock = this.takeLock;
	    takeLock.lockInterruptibly();
	    try {
			//当前队列为空，则阻塞当前线程，将其移入到等待队列中，直至满足条件
	        while (count.get() == 0) {
	            notEmpty.await();
	        }
			//移除队头元素，获取数据
	        x = dequeue();
	        c = count.getAndDecrement();
	        //如果当前满足移除元素的条件，则通知被阻塞的消费者线程
			if (c > 1)
	            notEmpty.signal();
	    } finally {
	        takeLock.unlock();
	    }
	    if (c == capacity)
	        signalNotFull();
	    return x;
	}

take方法的主要逻辑请见于注释，也很容易理解。

# 4. ArrayBlockingQueue与LinkedBlockingQueue的比较 #

**相同点**：ArrayBlockingQueue和LinkedBlockingQueue都是通过condition通知机制来实现可阻塞式插入和删除元素，并满足线程安全的特性；

**不同点**：1. ArrayBlockingQueue底层是采用的数组进行实现，而LinkedBlockingQueue则是采用链表数据结构；
2.  ArrayBlockingQueue插入和删除数据，只采用了一个lock，而LinkedBlockingQueue则是在插入和删除分别采用了`putLock`和`takeLock`，这样可以降低线程由于线程无法获取到lock而进入WAITING状态的可能性，从而提高了线程并发执行的效率。
