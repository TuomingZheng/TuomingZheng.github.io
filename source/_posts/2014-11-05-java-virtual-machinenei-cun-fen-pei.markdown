---
layout: post
title: "Java Virtual Machine内存分配"
date: 2014-11-05 23:44:35 +0800
comments: true
categories: Java虚拟机学习笔记
---
>Java虚拟机内存管理的两个方面是对象内存的分配和对象内存的回收，在之前的笔记中已经记录
了Java虚拟机的对象内存回收方面的基本内容。
<p>
	对象的分配主要是在Java虚拟机的堆空间进行分配，一般是分配在新生代的Eden区上，如果启动了本地线程
分配缓存，将按线程分配在TLAB上，少数情况下也可能分配在老年代上。
</p>
<h1>对象分配在Eden区上</h1>
<p>
在大多数情况下，对象在新生代Eden区分配，在Eden区没有足够的空间时虚拟机将进行一次Minor GC。在发生Minor
GC时如果Survivor没有足够的空间容纳Eden区没有被回收的对象时将直接分配到老年代。
<li>Minor GC</li>
<p>Minor GC是新生代垃圾收集动作，在新生代的Java对象大都具有朝生夕死的特点，所以Minor GC发生比较频繁。</p>
<li>Major GC</li>
<p>Major GC是老年代发生的垃圾收集动作，在老年代垃圾收集动作发生时一般伴随至少一次的Minor GC。</p>
</p>
<h1>大对象进入老年代</h1>
<p>
Java大对象指需要连续大量内存空间的Java对象，大对象容易导致内存还有不少空间时就提前触发垃圾收集来获得足够的空间
来容纳他们。Java虚拟机提供参数"-XX:PretenureSizeThreshold"指明当对象所需的内存空间大于这个值时将直接在老年代分
配。
</p>
<h1>长期存活的对象进入老年代</h1>
<p>
虚拟机给每个对象定义一个对象年龄计数器，如果对象在Eden区出生并经过第一次Minor GC后仍然存活并且能够被Survivor容纳
的话，将被移动到Survivor并且对象年龄设置为1。对象在Survivor中内经历一次Minor GC，对象年龄计数器加一，在对象年龄
计数器值到一定值时将移动到老年代。
</p>
<h1>动态对象年龄判定</h1>
<p>
如果Survivor空间中相同年龄的所有对象大小总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，
无需等到MaxTenuringThreshold中要求的年龄。
</p>
