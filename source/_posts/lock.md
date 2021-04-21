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

