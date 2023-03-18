## Java基础-显式锁




```
myLock.lock(); // a ReentrantLock object
try{
	critical section
}finally{
	myLock.unlock: // make sure the lock is unlocked even if an exception is thrown 
}
```



### 读写锁

下面是使用读/ 写锁的必要步骤:
1 )构 造 一 个 R e e n t r a n t Re a d Wr i t e L o c k 对 象 :
private ReentrantReadwriteLock rw = new ReentrantReadwriteLock0); 2 )抽取读锁和写锁:
private Lock readLock = rwl. readLock0: private Lock writeLock = rwl.writeLock0;
3 )对所有的获取 方法加读锁: public double getTotalBalance0)
readLock. lock0;
try {.
.. }
finally { readLock.unlock(); } ]
4 )对所有的修改 方法加写锁:
public void transfer(. .
).
{
API_java.util.concurrent.locks.ReentrantReadWriteLock5.0 • Lock readLock( )
得到一个可以被多个读操作共用的读锁，但会排斥所有写操作。
• Lock writelock( 得到一个写锁，排斥所有其他的该操作和写操作。
141 5 1 8 分什么我用s o p 和au s po n e 方考 1 - 0 。 初妢的Java版本定义了一个s10p方法用水终止一个线程，以及—个suspend 方法用*阳 塞一个线程直至另一个线程调用resumeo stop 和suspend 方法有一些共同点:都试图控制一
writeLock.lock0); try { . . . }
finally { writeLock.unlock0: 