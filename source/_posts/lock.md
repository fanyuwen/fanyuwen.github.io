---
title: lock
date: 2021-04-20 23:33:50
tags: java lock
---
### java锁 ###
java对象内存布局
java对象在内存种存储的布局可以分为3块区域: **对象头** **实例数据** **对齐填充**

- *对象头*: 主要包括两部分1. Mark Word(标记字段), 2. Klass Pointer(类型指针). Klass Point是对象指向它的类元数据的指针,虚拟机通过这个指针来确定这个对象是哪个类的实例(即指向方法区类的模板信息). Mark Word用于存储对象自身的运行时数据
- *实例变量*: 存放类的属性数据信息,包括父类的属性信息,这部分内存按4字节对齐(64位 8字节)
- 填充数据: 由于虚拟机要求对象起始地址必须是8字节的整数倍. 填充数据不是必须存在的,仅仅是为了字节对齐

下面是32位虚拟机的 mark word布局

<table>
    <tr>
        <td rowspan="2" align="center" valign="middle">锁状态</td>
        <td colspan="2" align="center" valign="middle">25bit</td>
        <td rowspan="2" align="center" valign="middle">4bit</td>
        <td align="center" valign="middle">1bit</td>
        <td align="center" valign="middle">2bit</td>
    </tr>
    <tr>
        <td align="center" valign="middle">23bit</td>
        <td align="center" valign="middle">2bit</td>
        <td align="center" valign="middle">是否偏向锁</td>
        <td align="center" valign="middle">锁标志位</td>
    </tr>
    <tr>
        <td align="center" valign="middle">无锁</td>
        <td colspan="2" align="center" valign="middle">对象hashCode</td>
        <td align="center" valign="middle">分代年龄</td>
        <td align="center" valign="middle">0</td>
        <td align="center" valign="middle">01</td>
    </tr>
    <tr>
        <td align="center" valign="middle">轻量级锁</td>
        <td colspan="4" align="center" valign="middle">指向栈中锁记录的指针</td>
        <td align="center" valign="middle">00</td>
    </tr>
    <tr>
        <td align="center" valign="middle">重量级锁</td>
        <td colspan="4" align="center" valign="middle">指向互斥量(重量级锁)的指针</td>
        <td align="center" valign="middle">10</td>
    </tr>
    <tr>
        <td align="center" valign="middle">GC标记</td>
        <td colspan="4" align="center" valign="middle">空</td>
        <td align="center" valign="middle">11</td>
    </tr>
    <tr>
        <td align="center" valign="middle">偏向锁</td>
        <td align="center" valign="middle">线程ID</td>
        <td align="center" valign="middle">Epoch</td>
        <td align="center" valign="middle">分代年龄</td>
        <td align="center" valign="middle">1</td>
        <td align="center" valign="middle">01</td>
    </tr>
</table>
轻量级锁和偏向锁是Java 6对synchronized锁进行优化后新增加,重量级锁也就是通常说synchronized的对象锁,
锁标识位为10,其中指针指向的是monitor对象(_也称为管程或监视器锁_)的起始地址.每个对象都存在着一个**monitor**与之关联
在Java虚拟机(HotSpot)中,monitor是由**ObjectMonitor**实现的,其主要数据结构如下(位于HotSpot虚拟机源码ObjectMonitor.hpp文件,C++实现),
省略部分属性

```c++
class ObjectMonitor() {
    _count = 0;//记录数
    _recursions = 0;//锁的重入次数
    _owner = NULL;//指向持有ObjectMonitor对象的线程
    _WaitSet = NULL;//调用wait后,线程会被加入到_WaitSet
    _EntryList = NULL; //等待获取锁的线程,会被加入到该列表
}
```
- 新建(New),新建后尚未启动的线程
- 运行(Runable),Runnable包括了操作系统线程状态中的Running和Ready
- 无限期等待(Waiting),不会被分配CPU执行时间,要等待被其他线程显式的唤醒.例如调用没有设置Timeout参数的Object.wait()方法
- 限期等待(Timed Waiting),不会被分配CPU执行时间,不过无需等待其他线程显示的唤醒,在一定时间之后会由系统自动唤醒.例如调用Thread.sleep()方法
- 阻塞(Blocked),线程被阻塞了,"阻塞状态"与"等待状态"的区别是:"阻塞状态"在等待获取着一个排他锁,这个事件将在另外一个线程放弃这个锁的时候发生,而"等待状态"则是在等待一段时间,或者唤醒动作的发生.在程序等待进入同步区域的时候,线程将进入这种状态
- 结束(Terminated):线程结束执行

对于一个synchronized修饰的方法(代码块)来说:
1. 当多个线程同时访问该方法,那么这些线程会先被放进_EntryList队列,此时线程处于blocked状态
2. 当一个线程获取到了对象的monitor后,那么就可以进入running状态,执行方法,此时,ObjectMonitor对象的_owner指向当前线程,_count加1表示当前对象锁被一个线程获取
3. 当running状态的线程调用wait()方法,那么当前线程释放monitor对象,进入waiting状态,ObjectMonitor对象的_owner变为null,_count减1,同时线程进入_WaitSet队列,直到有线程调用notify()方法唤醒该线程,则该线程进入_EntryList队列,竞争到锁再进入_Owner区
4. 如果当前线程执行完毕,那么也释放monitor对象,ObjectMonitor对象的_owner变为null,_count减1

由此看来,monitor对象存在于每个Java对象的对象头中(存储的是指针),synchronized锁便是通过这种方式获取锁的,也是为什么Java中任意对象可以作为锁的原因,同时也是notify/notifyAll/wait等方法存在于顶级对象Object中的原因

```assembly
monitorenter
monitorexit
ACC_SYNCHRONIZED
```

synchronized代码块是通过monitorenter和monitorexit两个指令实现的
synchronized方法是通过添加ACC_SYNCHRONIZED标识实现的

synchronized锁有四种状态,无锁,偏向锁,轻量级锁,重量级锁,这几个状态会随着竞争状态逐渐升级,
锁可以升级但不能降级,但是偏向锁状态可以被重置为无锁状态偏向锁(通过设置Mark Word倒数第三位的标志为0)

#### 偏向锁
因为经过HotSpot的作者大量的研究发现,大多数时候是不存在锁竞争的,常常是一个线程多次获得同一个锁,因此如果每次都要竞争锁会增大很多没有必要付出的代价,为了降低获取锁的代价,才引入的偏向锁.  
**升级过程**: 当线程1访问代码块并获取锁对象时,会在java对象头和栈帧中记录偏向的锁的threadID,因为偏向锁不会主动释放锁,因此以后线程1再次获取锁的时候,需要比较当前线程的threadID和Java对象头中的threadID是否一致,如果一致(还是线程1获取锁对象),则无需使用CAS来加锁、解锁;如果不一致(其他线程,如线程2要竞争锁对象,而偏向锁不会主动释放因此还是存储的线程1的threadID),那么需要查看Java对象头中记录的线程1是否存活,如果没有存活,那么锁对象被重置为无锁状态,其它线程(线程2)可以竞争将其设置为偏向锁;如果存活,那么立刻查找该线程(线程1)的栈帧信息,如果还是需要继续持有这个锁对象,那么暂停当前线程1,撤销偏向锁,升级为轻量级锁,如果线程1 不再使用该锁对象,那么将锁对象状态设为无锁状态,重新偏向新的线程.
#### 轻量级锁
轻量级锁考虑的是竞争锁对象的线程不多,而且线程持有锁的时间也不长的情景.因为阻塞线程需要CPU从用户态转到内核态,代价较大,如果刚刚阻塞不久这个锁就被释放了,那这个代价就有点得不偿失了,因此这个时候就干脆不阻塞这个线程,让它自旋这等待锁释放.  
**升级过程**: 线程1获取轻量级锁时会先把锁对象的对象头MarkWord复制一份到线程1的栈帧中创建的用于存储锁记录的空间(称为DisplacedMarkWord),然后使用CAS把对象头中的内容替换为线程1存储的锁记录(DisplacedMarkWord)的地址;
如果在线程1复制对象头的同时(在线程1CAS之前),线程2也准备获取锁,复制了对象头到线程2的锁记录空间中,但是在线程2CAS的时候,发现线程1已经把对象头换了,线程2的CAS失败,那么线程2就尝试使用自旋锁来等待线程1释放锁. 自旋锁简单来说就是让线程2在循环中不断CAS
但是如果自旋的时间太长也不行,因为自旋是要消耗CPU的,因此自旋的次数是有限制的,比如10次或者100次,如果自旋次数到了线程1还没有释放锁,或者线程1还在执行,线程2还在自旋等待,这时又有一个线程3过来竞争这个锁对象,那么这个时候轻量级锁就会膨胀为重量级锁.重量级锁把除了拥有锁的线程都阻塞,防止CPU空转.

|锁状态|优点|缺点|适用场景|
|----|----|----|----|
|偏向锁|加锁解锁无需额外的消耗,和非同步方法时间相差纳秒级别|如果竞争的线程多,那么会带来额外的锁撤销的消耗|基本没有线程竞争锁的同步场景|
|偏向锁|竞争的线程不会阻塞,使用自旋,提高程序响应速度|如果一直不能获取锁,长时间的自旋会造成CPU白白消耗|适用于少量线程竞争锁对象,且线程持有锁的时间不长,追求响应速度的场景|
|偏向锁|线程竞争不适用CPU自旋,不会导致CPU空转消耗CPU资源|线程阻塞,响应时间长|很多线程竞争锁,且锁持有的时间长,追求吞吐量的场景|

### 用锁的注意项
```java
//每次调用创建不同的锁,等于无锁
synchronized (new Object()){
    //do something
}
```
```java
//String、Boolean、包装类都采用了享元模式,即值在一定范围内,对象都是同一个.所以看似用了不同的对象,但其实用的是同一个对象,
//会导致一个锁被多个地方使用
private Integer count;
synchronized(count){
    //do something
}
```
```java
//常规对象锁
private final Object lock = new Object();
//静态对象锁
private static final Object lock = new Object();
```