---
title: HopSpot虚拟机对象
notshow: false
date: 2021-07-20 15:22:01
tags: [Java,JVM]
categories: Java
---
<meta name="referrer" content="no-referrer" />

--- 
# 一.对象的创建

虚拟机遇到new指令，先检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并检查这个符号引用代表的类是否已经被加载、解析和初始化过。如果没有，那必须先执行相应的{% post_link Java类加载机制 类加载 %}过程。

## 1.分配内存

类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可以完全确定。

  

## 2.内存分配方式

- 指针碰撞：如果Java堆中的内存规整，用过的内存放在一边，空闲的放在另外一边，中间放一个指针作为分界点的指示器。分配内存就是把指针向空闲空间挪动一段与对象大小相等的距离。
- 空闲列表：如果Java堆中的内存不规整，无法使用指针碰撞的方法，需要维护一个表，记录哪些内存块是可用的，在分配时从列表中找到一块足够大的内存块来划分给对象实例，然后更新列表记录。选择哪种分配方式由Java堆是否规整决定，而**Java堆是否规整由所采用的垃圾收集器是否带有压缩整理功能决定。**

## 3.并发问题

创建对象是很频繁的行为，虚拟机采用如下两种方式来保证线程安全：

- CAS配上失败重试：CAS是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。
- TLAB： 将内存分配安排在每个线程独有的空间进行，每个线程首先在堆内存中分配一小块内存，称为本地分配缓存(TLAB : Thread Local Allocation Buffer)。分配内存时，只需要在自己的分配缓存中分配即可，由于这个内存区域是线程私有的，所以不会出现并发问题。当对象大于TLAB中的剩余内存或TLAB的内存已用尽时，再采用上述的CAS进行内存分配。虚拟机是否使用TLAB，可以通过`-XX:+/-UseTLAB`参数来设定。

## 4.初始化零值

内存分配完成后，虚拟机将分配到的内存空间初始化为零值（不包括对象头），这一步保证了对象的实例字段在Java代码中可以不赋初始值就使用。

## 5.对象头设置

初始化零值完成之后，虚拟机要对对象进行必要的设置，例如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希吗、对象的 GC 分代年龄等信息。 这些信息存放在对象头中。

另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

## 6.执行init方法

从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，init 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 init 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

---
# 二.对象的内存布局

在 Hotspot 虚拟机中，对象在内存中的布局可以分为3块区域：

- 对象头
- 实例数据
- 对齐填充

Hotspot虚拟机的对象头包括两部分信息

- 第一部分用于存储对象自身的自身运行时数据（哈希码、GC分代年龄、锁状态标志等等）
- 另一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。

实例数据部分是对象真正存储的有效信息，也是在程序中所定义的各种类型的字段内容。包括父类继承的内容和子类中定义的内容。

对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。如果对象实例数据部分没有对齐的话，就需要通过对齐填充来补全。

---     
# 三.对象的访问定位

Java程序通过栈上的reference数据操作堆上的具体对象。主流的访问方式由使用句柄和直接指针两种。

- 句柄：如果使用句柄的话，那么Java堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息。
- 直接指针： 如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而reference 中存储的直接就是对象的地址。

这两种对象访问方式各有优势：

- 使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改
- 使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。（大部分是使用第二种方法来访问的）

**HotSpot使用直接指针的方式进行对象访问。**