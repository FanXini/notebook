### <center>hadoop 基本操作指令</center>

1. 格式化 Namenode

   > **hadoop namenode -format**

2. 开启hadoop集群指令

   > **start-all.sh | 先start-dfs.sh 再 start-yarn.sh**

3. 在hdfs里创建目录

   > **hadoop fs -mkdir /目录名**

4. 查看hdfs的文件列表

   > **hadoop fs -ls /**

5. 将本地文件上传到hdfs中

   > - **hadoop fs -put   /本地目录/文件名	/目标目录/文件名**
   > 	 **hadoop fs -copyFromLocal  /本地目录/文件名	/目标目录/文件名**

6. 将hdfs上面的文件下载到本地

   >- **hadoop fs -get /hdfs目录/文件名  /本地目录/文件名**
   >- **hadoop fs  -copyTotLocal  /本地目录/文件名   /目标目录/文件名**

7. 查看文件内容

   > + **hadoop fs -cat filename**
   > + **hadoop fs -text filename**
   > + **hadoop fs -tail filename**

8. 删除hdfs中的文件

   > **hadoop fs -rm filename**

9. 删除hadoop中的文件夹

   > **hadoop fs -rm -r dirname**