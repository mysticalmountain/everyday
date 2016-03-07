分布式系统中完整的业务被拆分到多个服务系统中，所以无法使用RDBMS的事务保证整个业务功能的原子性。所以目前交通用的出发方式为采用柔性事务，主要可以分为：多阶段提交型、补偿型、异步确保型、最大努力通知型。

# 基于zookeeper的多阶段提交事务设计
## 整体结构
![struct](./images/struct.png)

整体结构分为：事务管理服务(Transaction Manager)、资源管理服务（Resource Manager)、业务处理模块（Business Process)三步分。
- Transaction Manager
协调系统控制全局事务，管理事务生命周期，并协调资源。
- Resource Manager
负责控制和管理实际资源系统
- Business Process
负责调用实际业务资源完成整个业务操作
