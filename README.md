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
    ![图片](https://github.com/Hiooary/hadoop_5.io/blob/master/images/1.PNG)
    ![图片](https://github.com/Hiooary/hadoop_5.io/blob/master/images/2.PNG)

# 3.NameNode 体系结构</br>
  * 是整个文件系统的管理节点。它维护着整个文件系统的文件目录树，文件/目录的元信息和每个文件对应的数据块列表。接收用户的操作请求 </br>
 (文件目录树： 为了检索速度快，最好放在内存中，为了持久保存，则写入硬盘中;  元信息： 除了文件内容本身的，涉及文件的信息，比如大小，权限...  ls 列出的信息都是元数据信息;  归根结底在硬盘上，但运行时在内存 )</br>
  * hdfs-default.xml 源码文件，eclipse中打开，可以查看存放位置: </br>
 <b>\<name>hadoop.tmp.dir\</name></b></br>
 <b>\<value>/usr/local/hadoop/tmp\</value></b></br>
  * 在hadoop 系统中也可以查看，</br>
    <b># cd /usr/local/hadoop/tmp </b></br>
    <b># ls </b>( 可以看到 dfs mapred 目录)</br>
    <b># cd dfs </b></br>
    <b># ls </b>( 可以看到 data name namesecondary 目录)</br>
    <b># cd name </b> ( 可以看到一些文件，其中有 in_use.lock)</br>
    <b># more in_use.lock </b>( 没有什么内容，这个文件表示 name 目录已经被 namenode 进程占用了，那么在此启动 namenode 的时候，进程会报错，无法进入)</br>
    <b># cd current  </b></br>
    <b># ls </b>( 这些是namenode存储数据的文件，如果多个进程同时编辑数据会有问题，所以只能允许一个 namenode 存在，所以 in_use.lock  存在，锁定，其他 namenode 进程无法进入)</br>
    
    文件包括：</br>
   * fsimage：(核心文件) 元数据镜像文件。存储某一时段NameNode内存元数据信息(hdfs-site.xml 的 dfs.name.dir 属性)，为了保障安全行，会进行备份，hdfs-default.xml中 </br>
    <b>\<name>dfs.name.dir\</name></b></br>
    <b>\<value>${hadoop.tmp.dir}/dfs/name\</value></b></br>
    不可以直接在文件上修改，复制到 hdfs-site.xml 中，将 <b>\<value>${hadoop.tmp.dir}/dfs/name\</value></b> 改为用<b> "," </b>分割的目录列表(逗号为英文状态，且不加空格)，数据会同时存到多个目录下(最好是多台机器多个磁盘上的多个文件夹，越分散越好)</br>
    * edits：操作日志文件</br>
      存放位置：</br>
      <b>\<name>dfs.name.edits.dir\</name></b></br>
      <b>\<value>${dfs.name.dir}\</value>  </b></br>
      事务文件(表示原子性操作，比如存取钱，如果其中一个操作失败，那么账不平，所以要么全部成功，要么全部失败)，假设从本地上传 1G 的文件到 hdfs，耗时10s，上传过程中网络中断或者源数据被删导致上传失败，这时 hdfs 上的部分文件是不完整的，不应该显示给用户。那么怎么保证这种一致性? -> 先写入 edits，上传100M,上传200M,...,保存上传的事务过程，如果失败，则不会告诉 fsimage(通过 SecondaryNamenode 进程)。NameNode 要接管用户的操作请求，要求快速响应，数据放入内存才快，cpu也尽量满足用户需求，所以交给第三方进程去合并</br>
    * fstime：保存最近一次 checkpoint 的时间</br>
		以上这些文件是保存在 linux 的文件系统中</br>
  
 <b>SecondaryNameNode</b></br>
   * HA的一个解决方案，但不支持热备，配置即可(见源码)</br>
   * 执行过程：从 NameNode 上下载元数据信息(fsimage.edits)，然后把二者合并，生成新的 fsimage，在本地保存，并将其推送到 NameNode ，同时重置 NameNode 的 edits</br>
   * 默认在安装在 NameNode 节点上，但是不安全</br>    
   
# 4.DataNode 体系结构</br>
  * 提供真实文件数据的存储服务</br>
	# 文件块(block)：最基本的存储单位。对于文件内容而言，一个文件的长度大小是size，那么从文件的 0 偏移开始，按照固定的大小，顺序对文件进行划分并编号，划分好的每一个块称一个 Block，HDFS 默认 Block 大小是 64MB，以一个 256MB 文件为例，共有256/64=4 个 Block </br>
	# 不同于普通文件系统的是，HDFS中，如果一个文件小于一个数据块的大小，并不占用整个数据块存储空间 </br>
	# Replication，多副本，默认是三个 (数据安全；副本越多会占用磁盘空间，所以根据实际情况定；默认三个，在配置文件中配置；目录没有副本，只有文件才有副本 </br>
  * 设置副本：                                                                                                       
       <b> # cd hadoop/conf </b></br>                                                      
       <b> # more hdfs-site.xml</b></br>                                                                    
      可以看到                                                                                                       
       <b> \<name>dfs.replication\</name></b> </br>                
       <b>\<value>1\</value></b> </br>
      这里设置的个数为1，没写则默认为3，这里做实验，就一个DataNode，所以设置为1。三个副本一般存在三个机器上：分别存放在 就近放在最近机架上的电脑上(确定)，同一机架上的另一台电脑(随机)，另一机架的另一台电脑(随机))</br>
  * NameNode 主节点只有一个，作为存储数据的 DataNode 有很多个，hdfs 可以存储海量数据，实际上指的是 DataNode 节点的快速扩展。真实的数据存储在 DataNode 中，对于目录的结构信息，元数据信息存储在 NameNode 中 。</br>
  * Block 是 linux 文件系统划分的一个概念 (形同 windows 的簇的概念，太大容易造成浪费，太小会导致频繁的IO操作，慢)，真正的文件是流，单向的，一个字节一个字节的数，数到64MB，则划分一个Block，一个大文件就会按照 64MB 划分为很多的 Block，NameNode 把 Block 存放于不同的 DataNode 上</br>
      (例如：一个文件 65MB ,会产生 2 个 Block，分别为 64MB,1MB，一个文件 2M，会产生 1 个 Block，占 2M 磁盘空间，2 个2M 的文件，产生 2 个 Block，占 4M)</br>
  *  源码中：                                                                                                       
     <b>\<name>dfs.data.dir\</name></b></br>      
     <b>\<value>${hadoop.tmp.dir}/dfs/data\</value></b></br>                                                    
      存放 Block 的路径:                                                                     
     <b># cd /usr/local/hadoop/tmp/dfs/data/current </b></br>                                        
     <b># ls  ( 可以看到 Block 块 )</b></br> 
     <b># ll  ( 会有 .meta 文件(校验数据，与数据文件成对) )</b></br> 
     <b># hadoop fs -rmr hdfs://hadoop0:9000/*  </b> (*表示通配) 删除所有的数据；再开启一个终端，用于上传(之前的那个终端用于显示)，上传占用实际文件大小空间，不足 64MB 只会占用一个块；文件超过 64MB ，会分块，几块加起来大小与原始文件一样</br>
     <b># cd /usr/local # ll 可以看到一些文件 </b></br>
     <b># hadoop fs -put xxx(文件) /</b> (上传到根目录)，上传完了去另一个终端查看，可以看到块，大小与上传之前的大小一样，手工上传的数据：不同的服务器上传，麻烦； 而且 hadoop fs -ls / 查询不到，因为 NameNode 管理目录树，它不知道，hdfs是不认的 </br>
 * 根据实际情况，确定 Block 的大小，复制 <b>\<name>dfs.block.size\</name> \<value>67108864\</value></b> 到 hdfs-site.xml 文件中修改 大小 </br>
