---
layout: post
title: "Shell命令之备份与恢复"
date: 2014-11-05 23:58:38 +0800
comments: true
categories: Shell命令 
---
>备份是生活中比不可少的操作，如果在灾难发生时没有必要的备份那你将会欲哭无泪。Linux系统的Shell提供了强有力
的备份工具dump,为此我们有必要来了解一下这个很酷的小伙子。@@
<p>
Linux上有个强大的备份工具，那就是使用dump命令。dump命令的功能十分强大，该命令不仅可以备份整个文件系统而且
还可以备份指定的目录。不过dump命令在备份目录时存在一定的限制，那就是只能完整的备份整个目录，无法制定等级
进行备份。
</p>
<p>
下面将具体介绍使用dump进行文件系统备份和目录备份，以及他们的区别：  
</p>
<li>使用dump命令备份整个文件系统：</li>
<p>
使用dump命令备份整个文件系统时可以制定等级进行备份，也就是说我们可以在第一次备份指定文件系统时进行完整备份，
在以后对同一个文件系统进行备份时只需要备份当前文件系统与以前备份之间差异就可以。这样就可以大大减轻备份文件
对系统空间的使用量。
</p>

						dump	[-Suvj#] [-f 生成的备份文件]  待备份文件  
								-S 查看备份指定文件需要的磁盘空间  
								-u 将本次进行备份的时间记录到/etc/dumpdates文件内  
								-v 查看dump备份过程中的信息  
								-# #代表数字0到9，这个参数一般在制定等级备份时使用  
								-f 备份产生的文件名称  
							
						dump -W	<<== 该命令查看/etc/fstab内具有dump设置的分区是否备份过

						例子：	$ df -h	<<== 找出最小的文件系统进行备份处理
							   	$ dump -0vj -f root.dump0 /		<<== 对root目录进行备份, 在这里
																	 root目录是一个文件系统  
								$ touch new_file.txt			<<== 在根目录下新建一个文件  
								$ dump -1vj -f root.dump1 /		<<== 制定等级进行备份  

<li>使用dump命令备份指定目录：</li>
<p>
使用dump命令备份目录存在一定的限制，例如无法使用-u、-#参数并且所有的备份数据必须在该目录下。  
</p>

						dump	[-Svj] [-f 生成的备份文件]	待备份文件  
								-S	查看备份指定文件需要的磁盘空间  
								-v  显示备份过程中的信息  
								-j	数据使用bzip2进行压缩  
								-f  备份产生的文件名字  

						例子：	$ dump -S /etc	<<== 查看备份etc目录所需的磁盘空间  
								$ dump -0vj -f etc.dump /etc	<<== 对etc目录进行压缩备份  


<li>使用restore命令进行恢复操作：</li>
		
		restore用法：			restore -t [-f dumpfile] [-h]	<<== 查看dump文件  
								restore -C [-f dumpfile] [-D 挂载点]	<<== 比较dump与实际文件  
								restore -i [-f dumpfile]				<<== 进入互动模式，可以  
																			 进行部分文件还原  
								restore -r [-f dumpfile]		<<== 还原整个文件系统  

								其中-i一般进行目录的还原，而-r一般是进行文件系统的还原, -D参数表示  
							查看挂载点与dump内的不同文件  

		例子：		$ restore -t -f etc.dump		<<== 查看etc.dump内备份含有的数据  
					$ restore -i -f etc.dump		<<== 进入交互模式可以对目录进行部分还原   
<p>
其中restore的交互模式中可以使用add file、delete file和extract进行部分文件恢复的添加、删除和最终执行恢复操作。  
</p>
