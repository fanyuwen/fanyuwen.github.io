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

