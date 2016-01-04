hbase用于随机的实时的读或写大数据，目标用于托管非常大的表格，数十亿级别的行和百万级别的列。模仿google bigtable的非关系型数据库。  
# features
- 线性和模块化的可伸缩性
- 严格一致的读和写
- 自动化和可配置的分享表

# Data Model
## Table
表由行组成
## Row
行由一个主键和多个列组成，按主键的字符顺序排序。
# Column
列由column family and a column qualifier组成
# Column Family
一系列列和它们的值的物理存储点，通常考虑性能的原因，每个column family有一组存储属性，例如：它的值是否缓存在内存，数据是否被压缩，key是否被编码。表中的每行可能有相同的column family，因此给定行可能不存储任何数据。
# Column Qualifier

# Architecture
## NoSQL
hbase是NoSQL数据库，所以不支持结构化查询语言，列类型，索引，触发器。但支持线性和模块化扩展。
## When Should I Use HBase
hbase不是适应所有的问题
第一：有数百万或数亿的数据
第二：确认可以不使用RDBMS的列类型、索引、事务、高级的查询语言
第三：确保你有足够的硬件，HDFS不要少于5个数据节点
