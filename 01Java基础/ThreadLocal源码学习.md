## Java ThreadLocal类
### ThreadLocal理解
ThreadLocal一般称为**线程本地变量**，它是一种特殊的线程绑定机制，将变量与线程绑定在一起，为每一个线程维护一个独立的变量副本。通过ThreadLocal可以将对象的可见范围限制在同一个线程内。**ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景。**

#### 跳出误区
需要重点强调的的是，不要拿ThreadLocal和synchronized做类比，因为这种比较压根就是无意义的！sysnchronized是一种互斥同步机制，是为了保证在多线程环境下对于共享资源的正确访问。而ThreadLocal从本质上讲，无非是提供了一个“线程级”的变量作用域，它是一种线程封闭（每个线程独享变量）技术，更直白点讲，ThreadLocal可以理解为将对象的作用范围限制在一个线程上下文中，使得变量的作用域为“线程级”。

### ThreadLocal源码分析

#### 如何使用(初始化)
```java 
# 源码初始化方法
protected T initialValue() {
    return null;
}
# 源码 JDK8 添加
public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {
    return new SuppliedThreadLocal<>(supplier);
}

# 使用：1.创建对象并重写初始化方法
private static ThreadLocal<SimpleDateFormat> local = new ThreadLocal<SimpleDateFormat>() {
	@Override
	protected SimpleDateFormat initialValue() {
		return new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");
	}
};
# 使用：2.使用lambda表达式初始化
private static ThreadLocal<SimpleDateFormat> local = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss"));

# 使用：3.创建对象 + 调用Set() 方法。注意空指针异常[未set()调用get()]
private static ThreadLocal<SimpleDateFormat> local = new ThreadLocal<SimpleDateFormat>();
local.set(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
```

#### 内部类
在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本.  
初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。

```java
# ThreadLocalMap的Entry对ThreadLocal的引用为弱引用(WeakReference<ThreadLocal<?>>)，避免了 ThreadLocal 对象无法被回收的问题
# ThreadLocalMap 的 set 方法通过调用 replaceStaleEntry 方法回收键为 null 的 Entry 对象的值，以及 Entry 对象本身从而防止内存泄漏
static class ThreadLocalMap {
	static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    
    private Entry[] table;
}

# Thread 类中
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

#### 主要方法
```java
# 取得当前线程，然后通过getMap(t)方法获取到一个map，map的类型为ThreadLocalMap
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
# getMap中，是调用当期线程t，返回当前线程t中的一个成员变量threadLocals
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
# 维护map对象
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

### ThreadLocal使用场景
如上文所述，ThreadLocal 适用于如下两种场景

- 每个线程需要有自己单独的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

```java 
# 学习使用的测试代码
public static void main(String[] args) throws InterruptedException {
	CountDownLatch downLatch = new CountDownLatch(1);
	for (int i = 0; i < 5; i++) {
		final Thread t = new Thread() {
			@Override
			public void run() {
				System.out.println("当前线程:" + Thread.currentThread().getName() + ",已分配ID:" + ThreadId.get());
				try {
					downLatch.await();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println("End");
			}
		};
		t.start();
	}
	downLatch.countDown();
	System.out.println(1111);
}

static class ThreadId {
	// 一个递增的序列，使用AtomicInger原子变量保证线程安全
	private static final AtomicInteger nextId = new AtomicInteger(0);
	// 线程本地变量，为每个线程关联一个唯一的序号
	private static final ThreadLocal<Integer> threadId = new ThreadLocal<Integer>() {
		@Override
		protected Integer initialValue() {
			return nextId.getAndIncrement();// 相当于nextId++,由于nextId++这种操作是个复合操作而非原子操作，会有线程安全问题(可能在初始化时就获取到相同的ID，所以使用原子变量
		}
	};

	// 返回当前线程的唯一的序列，如果第一次get，会先调用initialValue，后面看源码就了解了
	public static int get() {
		return threadId.get();
	}
}
```

### 总结

- ThreadLocal 并不解决线程间共享数据的问题
- ThreadLocal 通过隐式的在不同线程内创建独立实例副本避免了实例线程安全的问题
- 每个线程持有一个 Map 并维护了 ThreadLocal 对象与具体实例的映射，该 Map 由于只被持有它的线程访问，故不存在线程安全以及锁的问题
- ThreadLocalMap 的 Entry 对 ThreadLocal 的引用为弱引用，避免了 ThreadLocal 对象无法被回收的问题
- ThreadLocalMap 的 set 方法通过调用 replaceStaleEntry 方法回收键为 null 的 Entry 对象的值（即为具体实例）以及 Entry 对象本身从而防止内存泄漏
- ThreadLocal 适用于变量在线程间隔离且在方法间共享的场景