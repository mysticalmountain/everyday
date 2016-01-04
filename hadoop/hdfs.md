# 概述
HDFS的分布式存储用于hadoop程序。HDFS集群主要由NameNode构成，它管理文件系统的metadata和nodedata,它存储真是的数据。
- hadoop包含HDFS和mapreduce。hdfs用于分布式存储和分布式处理在经济性的硬件上。mapreduce用于大的分布式处理程序。
- hdfs是高度可配的，仅在非常大的集群环境需要修改配置
- hadoop用java编写支持绝大多数平台
- hadoop支持shell命令与hdfs目录交互
- NameNode and Datanodes have built in web servers that makes it easy to check current status of the cluster.
