---
layout: post
title: "Android开源组件之DiskLruCache"
date: 2015-01-08 22:24:54 +0800
comments: true
categories: android应用层开发笔记 
---
>在Android应用程序开发过程中不可避免的要接触到内存数据Cache和外部存储介质数据的Cache。DiskLruCache
作为AOSP提供的一个外部存储介质数据的Cache，我们有必要对它进行深入的了解。
<p>
对一个开源组件的学习，首先必须掌握该组件最基本的使用方法。在熟练掌握基本使用方法后才能进一步根据组件
使用场景对组件的实现进行深入分析。本片博客将根据习惯，将从该组件的使用方法开始介绍，接下来将详细介绍
日志文件的格式，最后按照API的使用场景对组件的实现进行剖析。
</p>
<h1>DiskLruCache基本使用介绍</h1>
<p>
DiskLruCache做为外部存储介质的缓存，我们必须了解其基本的组成结构。DiskLruCache由一系列的entry组成，每
个entry对应与journal文件中的一行记录。而一个entry又是由若干个values组成，其实每个value代表着一个文件。
至于每个entry所拥有的文件个数则用户可以通过DiskLruCache的<code>open()</code>方法进行配置。
</p>
<p>
我们在进行编程的时候常用的是通过DiskLruCache的open()方法获得一个DiskLruCache实例（DiskLruCache的构造方
法是私有的）。在获取到DiskLruCache实例后，通过该类的edit()方法获取指定key对应的Editor进行读写操作或则
通过get()方法获取Snapshot实例进行批量的读操作。
</p>
	final int appVersion = 1;
	final int valueCount = 10;
	final int maxSizeInByte = 1024 * 1024 * 100;
	File cacheDir = new File(Environment.getExternalStorageDirectory(), "diskCache");
	DiskLruCache cache = DiskLruCache.open(cacheDir, appVersion, valueCount, maxSizeInByte);
	Editor editor = cache.edit("thumnail");
	FileOutputStream fos = editor.newOutputStream(1);
	.....
	editor.commit();

	FileInputStream fis = new FileInputStream(editorr.newInputStream(1));
	.....
<h1>DiskLruCache的journal文件格式介绍</h1>
<p>
在介绍DiskLruCache的journal文件格式之前，先了解一下DiskLruCache的文件构成，这有助于我们正确的理解journal
文件的格式。DiskLruCache所有的文件都在用户指定的一个目录内，该目录内包含了DiskLruCache管理所需的记录文件
也包含了存储数据的文件。
</p>
<h2>DiskLruCache管理所需的记录文件</h2>
<p>
其中DiskLruCache管理所需的记录文件包括这几个文件，journal文件、journal backup文件和journal tmp文件。其中
journal文件是这一部分着重需要说明的内容，其他两个文件的用途通过文件的名字可以很清楚的看出来。journal back
文件保存的是处于正确状态的journal文件的内容，一般情况下该文件是不存在的，只有在journal文件未成功更新到一
个新的状态值时才可能产生的。总的来说，这几个文件都是辅助journal文件从一个正确的状态切换到另一个正确的状态，
{% img center /images/rebuildJournal.png %}
从上面的代码可以看出journal tmp文件是在rebuild journal文件时写入的数据的文件，该文件存储的是下一个正确状态
的值，journal backup文件是上一个状态journal文件的值。在journal tmp文件成功改为journal文件的时候会将backup
文件删除。
</p>
<h2>DiskLruCache保存数据的文件</h2>
<p>
DiskLruCache是由entry构成的，而每个entry又是由若干个value构成的。其中每个value对应着两个文件，一个文件称为
clean文件，另一个文件称为dirty文件。这里的dirty文件和clean文件的关系与journal文件和journal tmp文件类似，
dirty文件是用户写入数据的文件，只有用户确认提交写入的数据时才会将dirty文件更名为clean文件。在文件系统中这
两个文件的名字有着固定的模式，dirty文件的名字通常是key.index.tmp而clean文件的名字通常是key.index。也就是说
用户通过Editor的<code>newOutputStream()</code>获取的OutputStream写入的数据实际上是写到dirty文件内，只有用户
调用<code>commit()</code>方法时才会将dirty文件更名为clean文件。
</p>
<h2>Journal文件格式和记录格式</h2>
<h3>Journal文件格式</h3>
<p>
与其他一些常见格式的文件类似，DiskLruCache文件也存在文件头的概念。DiskLruCache文件头由五行数据组成，这五行
数据标识了DiskLruCache journal文件的身份。他们的顺序分别是MAGIC（libcore.io.DiskLruCache）、VERSION（DiskLruCache
的版本）、appVersion、valueCount（每个entry包含的value的个数）。
</p>
<p>
这些数据的构建在<code>rebuildJournal()</code>方法内，该方法在整个组件中有两处地方被调用。他们分别是在用户首次
使用该cache时调用<code>open()</code>方法，和组件定时整理Cache使用的空间时调用。这两处地方是：
{% img center /images/diskLruCacheOpen.png %}
{% img center /images/cleanUpCallable.png %}
</p>
<h3>Journal文件的记录格式</h3>
<p>
DiskLruCache的journal文件除了header部分以外，其他的内容中每一行都代表着cache中一个entry的记录信息。这里的记录
信息包括记录状态、键值和其他状态值组成（例如字节数）, 这些数据通过空格字符分割开来。记录状态和记录的键值必须
符合一定的规范，其中记录的状态包括固定的几个值CLEAN、READ、DIRTY和REMOVE，而键值则必须由0-9、a-z和下划线构成
的并且整个key的长度不得超过64个字符。
</p>
<li>DIRTY状态：</li>
<p>
DIRTY状态表明当前的entry处于被创建或则更新的状态。正常情况下每个DIRTY状态记录的下一个记录是CLEAN或则REMOVE活动。
如果一个DIRTY状态的记录缺乏配对的CLEAN或则REMOVE记录则表明value的dirty文件需要被删除掉！
</p>
<li>CLEAN状态：</li>
<p>
CLEAN状态表明对应的entry被成功的commit并且现在可以进行读操作。CLEAN记录中会包含entry每个value对应的长度。
</p>
<li>REMOVE状态：</li>
<p>
REMOVE状态表明该记录已经被删除了。
</p>
<li>READ状态</li>
<p>
READ状态记录了entry访问的频率。
</p>
<h1>DiskLruCache的API场景实现分析</h1>
<p>
这里我们从调用API的角度按程序的执行逻辑逐步分析DiskLruCache的实现原理。这里将划分为三条路径进行分析，这三条路径
分别为DiskLruCache调用<code>edit()</code>方法获取Editor、调用<code>get()</code>方法获取Snapshot和调用<code>remove
()</code>方法删除指定的entry。
</p>
<li>DiskLruCache的open()方法</li>
<p>
在以上三条执行路径中均包含DiskLruCache实例获取的操作，为此必须将DiskLruCache实例获取的代码的逻辑搞清楚。由于
DiskLruCache的构造方法是私有的，所以获取DiskLruCache实例必须通过<code>open()</code>方法获取。<code>open()</code>
方法需要正确处理journal文件和journal backup文件的关系，如果directory下存在journal backup文件则需要根据journal文件
是否存在进行journal文件的恢复或则journal backup文件的删除。在journal文件存在的情况下需要根据文件里面的信息构建
entry的Lru映射关系，如果journal文件不存在则需要根据内存里entry的信息rebuild一个Lru的映射表。
{% img center /images/DiskLruCacheOpen.png %}
</p>
<li>DiskLruCache的edit()方法</li>
<p>
DiskLruCache的<code>edit()</code>方法的逻辑是根据传进来的key获取entry或则创建新的entry。如果无法获得entry对象则新
创建一个Entry实例，并将其currentEditor设置为新创建的Editor对象。如果可以获取到entry对象则判断其是否存在currentEditor
，如果存在该对象则说明该entry正在被访问当前请求无法满足。
{% img center /images/DiskLruCacheEdit.png %}
</p>
<li>DiskLruCache的get()方法</li>
<p>
DiskLruCahe的<code>get()</code>方法获取Snapshot对象，利用该对象进行读操作。<code>get()</code>方法获取key对应的Entry
，创建一系列FileInputStream对象，该对象是value的clean文件的FileInputStream。在journal文件内添加一条READ状态的记录并
且创建一个Snapshot对象用于返回。
</p>
{% img center /images/DiskLruCacheGet.png %}
<li>DiskLruCache的remove()方法</li>
<p>
DiskLruCache的<code>remove()</code>方法首先获得key对应的Entry，根据该Entry是否处于编辑状态进行控制。如果Entry处于编辑
状态那么不允许用户删除当前的Entry，否则删除该Entry的value对应的clean文件并重新计算DiskCache的size。如果成功删除该entry
的数据那么将在journal文件中添加一条REMOVE的记录。
{% img center /images/DiskLruCacheRemove.png %}
</p>
