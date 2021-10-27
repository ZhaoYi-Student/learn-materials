### Synchronized

### java 对象组成
>- 对象头
>>- markword
>>>- 用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等
>>- klass point
>>>- 对象用来访问元数据的指针，jvm可以通过这个指针来确定这个对象是那个类的实例
>- 实例数据
>>- 将对象中的属性实例化
>- 对齐填充
>>- 占位符的作用，Hotspotvm的内存管理系统要求对象的起止地址必须为8byte，如果不满足条件的话则会通过占位符的形式填充

>- 主内存和工作内存之间交互过程
>>- lock（加锁）
>>- read（从主内存中读取信息）
>>- load（将主内存信息加载到工作内存中）
>>- use（使用这个信息）
>>- assign（操作并赋值）
>>- store（存储到工作内存中）
>>- write（再从工作内存写到主内存中）
>>- unlock（解锁）

![Alt text](https://img-blog.csdn.net/20141021095329497)

- 可见性
- 原子性
- 有序性
- 可重入性
- 不可中断性

### Synchronized 锁升级过程
- 无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁
- 无锁
- 偏向锁
-- 有研究表明，其实在大部分场景都不会发生锁资源竞争，并且锁资源往往都是由一个线程获得的。如果这种情况下，同一个线程获取这个锁都需要进行一系列操作，比如说CAS自旋，那这个操作很明显是多余的。偏向锁就解决了这个问题。其核心思想就是：一个线程获取到了锁，那么锁就会进入偏向模式，当同一个线程再次请求该锁的时候，无需做任何同步，直接进行同步区域执行。这样就省去了大量有关锁申请的操作。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果。
- 轻量级锁
-- 当有另外一个线程竞争获取这个锁时，由于该锁已经是偏向锁，当发现对象头 Mark Word 中的线程 ID 不是自己的线程 ID，销偏向锁状态，将锁对象markWord中62位修改成指向自己线程栈中Lock Record的指针（CAS抢）执行在用户态，消耗CPU的资源（自旋锁不适合锁定时间长的场景、等待线程特别多的场景），此时锁标志位为：00。
- 重量级锁
-- 若是重量锁，对象头中还会存在一个监视器对象，也就是Monitor对象。这个Monitor对象就是实现synchronized的一个关键。

synchronized的执行过程：
1. 检测Mark Word里面是不是当前线程的ID，如果是，表示当前线程处于偏向锁
2. 如果不是，则使用CAS将当前线程的ID替换Mard Word，如果成功则表示当前线程获得偏向锁，置偏向标志位1
3. 如果失败，则说明发生竞争，撤销偏向锁，进而升级为轻量级锁。
4. 当前线程使用CAS将对象头的Mark Word替换为锁记录指针，如果成功，当前线程获得锁
5. 如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。
6. 如果自旋成功则依然处于轻量级状态。
7. 如果自旋失败，则升级为重量级锁。

![Alt text](https://img-blog.csdnimg.cn/20200802155604236.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDkxMDM3Mg==,size_16,color_FFFFFF,t_70)

>- Monitor(对象)
>>- _count: 计数器
>>- _owner: 持有当前锁对象的线程
>>- _WaitSet: 队列。当一个线程调用了wait方法后，它会释放锁资源，进入WaitSet队列等待被唤醒
>>- _EntryList：队列。里面存放着所有申请该锁对象的线程。

### Lock
Lock接口
```
public interface Lock {

	//申请锁，如果没有申请到，则阻塞当前线程
	//屏蔽线程中断
    void lock();

	//与lock()方法作用类似，如果发生了线程中断，该线程会抛出InterruptedException异常
    void lockInterruptibly() throws InterruptedException;

	//尝试加锁，如果加锁成功，则返回true，失败返回false
	//无论加锁成功还是失败，该方法都会立即返回
    boolean tryLock();

	//尝试加锁，如果加锁成功，立即返回true，
	//如果失败则会阻塞当前线程time时间，如果在time期间内申请到了，则返回true，如果没有申请到，返回false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

	//解锁
	void unlock();

	//返回与锁相关的Condition对象
    Condition newCondition();
}
```

ReentrantLock
```
/**
 * true   设置为公平锁
 * false  设置为非公平锁
 */
public ReentrantLock(boolean fair){}

/**
 * FairSync
 */
 protected final boolean tryAcquire(int acquires) {
     final Thread current = Thread.currentThread();
     //调用父类方法获得属性state的值
     int c = getState();
     //c=0表示当前没有线程占用锁
     if (c == 0) {
       //hasQueuedPredecessors()方法用于检查队列中是否有线程等待锁
         if (!hasQueuedPredecessors() &&
             compareAndSetState(0, acquires)) {
             setExclusiveOwnerThread(current);
             return true;
         }
     }
     else if (current == getExclusiveOwnerThread()) {
         int nextc = c + acquires;
         if (nextc < 0)
             throw new Error("Maximum lock count exceeded");
         setState(nextc);
         return true;
     }
     return false;
 }

 /**
  * NonfairSync
  */
  final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}

```

### redis分布式锁
@link: https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/
