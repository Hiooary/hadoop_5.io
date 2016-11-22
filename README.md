# 1. 分布式文件系统</br>
  * 数据量越来越多，在一个操作系统管辖的范围存不下了，那么就分配到更多的操作系统管理的磁盘中，但是不方便管理和维护，因此迫切需要一种系统来管理多台机器上的文件，这就是分布式文件系统</br>
  * 是一种允许文件通过网络在多台主机上分享的文件系统，可让多机器上的多用户分享文件和存储空间</br>
  * 通透性，让实际上是通过网络来访问文件的动作，由程序与用户看来，就像是访问本地的磁盘一般</br>
  * 容错，即使系统中某些节点脱机，整体来说系统任然可以维持运作而不会有数据损失</br>
  * 分布式文件管理系统很多，hdfs只是其中一种，适用于一次写入多次查询的情况(可以删除了重写)，不支持并发写情况，小文件不适合( Block 越多对 NameNode 的内存压力越大)</br>
  
# 2. HDFS的shell操作</br>
  * (开启虚拟机，打开终端，验证hadoop程序是否运行：jps (出现5个java进程))</br>
	   对hdfs的操作方式：<b> hadoop fs -xxx </b> </br>
       <b>例如：</b> hadoop fs -ls / 看根目录下的内容	hadoop fs -lsr / 递归看根目录下的内容 </br>
	   注意：这个命令查的是 hdfs 的目录， # ls 才是查看 linux 的目录 </br>
	 * 常用命令： hadoop fs -mkdir /d1 在 hdfs 下创建文件夹</br>
       mkdir /dl  在 linux 下创建文件	</br>
	   hadoop fs -put /root/install.log(源，来自linux)  /d1(目的地，hdfs) 从 linux 上传数据到 hdfs d1文件夹下  (再次上传，会提示已经存在，不会默认覆盖)</br>
	   hadoop fs -put /root/install.log(源) /d2(目的地) (如果目的 hadoop地路径 d2 不存在时，路径名称默认表示文件名称)</br>
	   hadoop fs -put /root/install.log /d2/abc         (同上)	</br>
	   cd Desktop， hadoop ls -get /d1/abc(源) .(目的的->当前目录)， ls  ->显示abc  (从 hdfs 的源下载数据到 linux 的特定目录下)</br>
	   hadoop fs -text /d1/abc (查看 hdfs 中的文件)</br>
	   hadoop fs -rm /d1/abc   (删除文件)	</br>
    hadoop fs -rmr /d1/abc (删除文件夹)</br>
	   hadoop 查看命令	</br>
    hadoop fs 查看命令</br>
    hadoop fs -help xxx(查看帮助文档)</br>
    ls 遍历文件	mkdir 创建文件	cp 复制...</br>
		
