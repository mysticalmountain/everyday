[TOC]

# 2016-01
## 2016-01-04
### 写出高质量代码的10个建议
引用资料：http://blog.jobbole.com/95989/#comment-155064
#### 打好基础
- 掌握好开发语言
- 熟悉开发平台
- 基础的数据结构与算法
- 基础的设计原则

#### 代码标准
#### 想好再写
#### 代码重构
#### 技术债务
#### 代码审查
#### 静态检查    
#### 单元测试
#### 充分自测
#### 善用开源

### 20个命令行工具监控 Linux 系统性能
- top —Linux系统进程监控
- vmstat — 虚拟内存统计
- lsof — 打开文件列表
- tcpdump — 网络数据包分析器
- netstat — 网络统计
- htop — 进程监控
- iotop — 监控 Linux 磁盘 I/O
- iostat — 输入/输出统计
- IPTraf —实时IP局域网监控
- Psacct 或者 Acct — 监视用户活动
- Monit — 程序和服务监测
- NetHogs — 监视每个进程的网络带宽
- iftop — 网络带宽监控
- Monitorix — 系统和网络监控
- Arpwatch — 以太网活动监控器
- Suricata — 网络安全监控
- VnStat PHP — 监测网络带宽
- Nagios — 网络/服务器监控
- Nmon — 监控Linux系统性能
- Collectl — 一体化性能检测工具

## 2016-01-28
### 可用性的理解

- 理解目标
业界对高可用的目标是几个9

| Level         | 每年岩机时间    | 每天岩机时间     |
| ------------- | --------------- | ---------------- |
| 9             | 36.5days        | 2.4hrs           |
| 99            | 3.65days        | 14min            |
| 99.9          | 8.76hrs         | 86sec            |
| 99.99         | 52.6min         | 8.6sec           |
| 99.999        | 5.25min         | 0.86sec          |
| 99.9999       | 31.5sec         | 8.6ms            |

- 拆解目标

几个9的目标比较抽象，需要对目标进行合理拆解，可以分解成如下两个子目标

1. 频率要低，即：减少出故障的次数
2. 时间要快，即：故障恢复时间要快
故障时不是解决或者定位到具体问题，而是快速恢复是第一要务，防止次生灾害，问题扩大。这就是要站在业务角度思考，而不是站在技术角度思考。

### 系统设计思路
- 大系统做小
把复杂系统拆成单一职责系统，并从单机、主备、集群、异地等架构方向扩展
- 基础通道做大
把基础通讯框架，带宽的高速路做大
- 流量分块
把用户流量按照某种模型拆分，让他们聚合在一个集群完成，闭环解决。

### 易运营
高可用的系统一定是可运营的，这种运营不是产品运营而是技术运营，指的是线上质量、流程能否运营。比如这个系统上线后，是否方便切换流量，是否方便开关，是否方便扩展。
这里有几个基本要求
1. 可限流
线上的流量永远有想不到的情况，在这种情况下系统的吞吐能力就非常重要，高并发的系统一般采取的策略是快速失败机制。比如系统QPS是5000，当10000的流量过来，我们能保证持续的5000，其他的5000快速失败
2. 无状态
需要要完全无状态，这样才能保证运维随便扩容，分配流量。
3. 降级能力

### 可测试 

### 降低发布风险
- 严格的发布流程
- 灰度机制


### 经验
1．珍惜每次真实高峰流量，建立高峰期流量模型。
2．珍惜每次线上故障复盘，上一层楼看问题，下一楼解决问题。
3．可用性不只是技术问题 
系统初期是：以开发为主；
系统中期是：开发＋DBA＋运维为主；
系统后期是：技术＋产品＋运维＋DBA ；
4．单点和发布是可用性最大的敌人


### 云计算
一种新型的IT交付模式，为企业节省IT成本，



zTree logoJQuery Tree 插件 zTree
http://www.ztree.me/v3/main.php

HTML5前端UI框架 ZUI
http://zui.sexy/

ECharts logoJavaScript图表库 ECharts
http://echarts.baidu.com/index.html

DWZ logo国产jQuery UI框架 (jUI) DWZ
http://jui.org/



## 2016-02-02
### 办公IP分配
10.1.240.85
255.255.255.0
10.1.240.1
222.222.222.222

### 版本库
svn://115.29.147.186/01.%E7%BD%91%E7%BB%9C%E9%87%91%E8%9E%8D%E9%83%A8
svn://115.29.147.186/01.网络金融部
svn://115.29.147.186/阿里云/02.workspace
svn://115.29.147.186/%E9%98%BF%E9%87%8C%E4%BA%91/02.workspace
username:andongxu
password:



### 技术体系
技术架构
demo
代码规范
版本管理

开发环境，形成文档：
> mongodb安装、集群，数据分片
> redis安装、集群
> mysql安装、集群、数据分片
> MQ安装、集群
> zookeeper安装，集群

基础开发API
> 访问db
> 访问redis
> 访问MQ
> 访问ZK


## 2016-02-04

1.2.需求目标
	整合电子渠道以创新，(貌似部通顺)
1.7.12.柜面改造


系统层面：依托渠道整合平台

卡片类
	邦卡、开户、签约
	借记卡余额查询、开户信息查询、交易明细查询
	信用卡账单、余额等查询、开户信息查询、账单分期、积分查询

业务预约类
	预约取号
	预约大额取款
	申请办卡

营销类
	抽奖
	推荐有礼
	优惠专区
	优惠查询
	答题赢流量
	天天特惠

查询类
	网点
	公告

理财
	理财、基金购买
	金融行情
	存贷利率查询

生活类
	生活缴费
	门票类
	车票类
	医院挂号

通知类
	交易提醒

设置类
	设置某些开关


## 2016-02-05


maven-archetype-quickstart:1.0

mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart:1.0 -DgroupId=com.hdcb.nfd -DartifactId=nfd-common-redis


test update


join ttt

my modify

my featrue modify


## 2016-02-16
## 外部接口域
- 银银接口
- 人行支付
- 人行横向联调
- 人行支票影像
- 中间业务接口
- 银联接口
- 同业市场
- 身份核查
- SWIFT
- 短信平台
- 人行外币支付（新建）
### 问题
图标 1 、2 、 3 是吗意思？
为什么改造？现有系统有什么问题？
如何改造？是否有初步思路
是找外部厂商？还是自主开发？
系统设计和开发是由开发商负责？还是由自己负责？ 不同的方式对工作计划影响很大
目前系统是否都跑在我们机房？
2. 确认以下信息
	开发厂商、开发语言、对接系统、目前由谁负责运维、 	








## rocketmq

1.下载程序
2.tar -xvf alibaba-rocketmq-3.0.7.tar.gz 解压到适当的目录如/opt/目录
3.启动RocketMQ：进入rocketmq/bin 目录 执行
<!-- Crayon Syntax Highlighter v2.6.0 -->
 
 

nohup sh mqnamesrv &
1
nohup sh mqnamesrv&
<!-- [Format Time: 0.0011 seconds] -->
4.启动Broker，设置对应的NameServer
<!-- Crayon Syntax Highlighter v2.6.0 -->
 
 

nohup sh  mqbroker -n "127.0.0.1:9876" &
1
nohup sh  mqbroker-n"127.0.0.1:9876"&
<!-- [Format Time: 0.0010 seconds] -->



## 2016-02-22
### ACID
原子性(Atomicity)
> 原子性意味着数据库中的事务执行是作为原子。即不可再分，整个语句要么执行，要么不执行。

一致性(Consistency)
> 一致性,即在事务开始之前和事务结束以后，数据库的完整性约束没有被破坏

理解隔离性（Isolation)
> 隔离性。事务的执行是互不干扰的，一个事务不可能看到其他事务运行时，中间某一时刻的数据。

持久性（Durability)
> 持久性，意味着在事务完成以后，该事务所对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。

### 一致性
> 系统内
>> 事务、锁

> 系统间
>> 调单查询：在发现有未知数据时通过查询的方式恢复流水
>> 对账恢复：T+1针对T日的全量数据进行比对一致性，包括状态、金额等关键信息
>> 异常恢复：可以和掉单查询互为补充，由下游检测异常并进行恢复

### CAP
一致性 Consistenc
可用性 Avalibility
分区容忍性 Tolerance of network partition
CAP理论认为三点不可能同时满足，证明如下：
假设系统出现网络分区为 G1 和 G2 两个部分，在一个写操作 W1 后面有一个读操作 R2 ， W1 写 G1 ， R2 读取 G2 ，由于 G1 和 G2 不能通信，如果读操作 R2 以终结的话，必定不能读取写操作 W1 的操作结果。


### GAPS
GAPS即General Application Preposed System，综合应用前置系统，简称大前置系统。
各种交易发起渠道集中、统一的中间接入系统，把各种终端设备的前置系统和外围系统与银行业务主机系统分离，在系统上集中实现到相关的不同业务子系统的交易路由，是银行开展一般业务交易发起终端和后台帐务主机间的枢纽控制主机。

## 2016-03-02

层次清晰
职责分明
容易扩展
不同技能的开发人员有不同职责（如UED和程序员）## 2016-03-02

在首席架构师眼里，架构的本质是……
http://www.oschina.net/news/71136/nature-of-architecture

## 2016-03-07
### 柔性事务
- 两阶段型
- 补偿型
- 异步确保型
- 最大努力通知型


## 2016-03-22
## 与易诚互动讨论架构

单点登陆
基础服务
安全服务
存贷服务
增值缴费
投资理财
营销服务

## 行方负责
用户中心、单点登陆、安全中心



# 2016-03-23
1. 支付的页面由睿民开发，页面采用H5规范
2. 用户中心(包含个人、企业、商户)、用户鉴权，单点登陆、由互联网金融部负责，提供两种接口规范（JSON，RPC）
3. 内管功能部分 柜员、机构、角色、权限由互联网金融部主导
4. 单点登陆存在疑问，需要清宝后续确定
5. 核心接口由易诚互动统一封装并对外提供调用接口
6. 与银联或财付通等第三方接口由互联网金融部统一封装并提供调用接口
7. 二维码相关接口由锐民统一提供,最终由易诚互动与睿民协商
8. 短信由易诚互动封装并提供调用接口
9. 图片统一由睿民统一封装并提供接口
10. 银联网关类接口是由睿民或互联网金融部提供待确定
11. 睿民开发的支付页面是否采用native + h5 的模式？，睿民可以打成SDK,该包可以嵌入到手机银行
12. 文件服务器采用 nginx由睿民开发 



# 2016-03-30
sublime text 3 中文输入发
http://www.jianshu.com/p/bf05fb3a4709

# 2016-03-31
中间业务系统：商业银行不运用或较少运用自己的资金，以中间人的身份为客户提供代收付和其他委托事项，提供各类金融服务并收取手续费的业务。

## 招标流程
1. 厂商初次交流
> 成果
>> 功能范围
>> 报价参考
>> 厂商实施经验、产品成熟度
>> 技术平台成熟度、技术平台扩展性、技术平台先进性
2. 厂商二次交流
> 成果
>> 明确的功能范围
>> 项目预算
>> 形成汇报材料mZ
3. 整理项目需求和需求评审
4. 准备立项材料
5. 发起立项申请

# 2016-04-05

睿民嵌入在手机银行的页面采用H5的方式
郑永和姚栋共同确认单点登陆跳转流程

# 2016-04-18
172.172.1.11	3600
172.172.1.21	3600




## 工作安排
软令牌不还要了
- 用户中心
- 安全中心
- 单点登陆

### 5.30前任务
功能细化
系统设计
接口设计
	接口设计
	接口文档（提供给睿民）
开发计划
开发规范

周三前用户中心搞完，包括代码和设计

### 5.30后任务
- redis支持http接口
- 消息推送

### 用户中心
类型：个人、商户、企业
#### 个人注册
手机号检查　
用户信息保存	
动态码生成
动态码验证

绑定卡片
认证方式
	手机号　＋　动态验证码
	卡号　＋　密码
	usbkey
积分
限额

用户信息采集多少？
	姓名、性别、年龄、身份证号、手机号、电子邮箱、ECIF客户号、核心客户号


关注微信公共账号自动注册为游客类型用户

#### 短信验证码生成
> 系统号 + 交ID + 手机号

#### 短信你验证码验证

#### 个人登陆
> 用户名　＋　登陆密码
> 手机号　＋　动态验证码



#### 密码找回
> 密保问题支持公共和自定义



### 安全中心


接口文档　安东旭　整理


### 单点登陆




功能细化、优先级、时间点


内管用户中心（易诚）


董泽杉：１８６６１７５５２２３
汤林海：




南天交流资料
需求我知道桂龙不知道	
找传奇说加密的事情
王垚
岳辉


https://192.168.1.76/svn/hdyh/dubbo_fairy_uscenter


王芳：13483080179


王宏睿：18501033410


代码与易诚互动保持一直

公共报文头加　　
	系统ｉｐ   
	客户　　ＩＰ
	USERid  可为空

注册
	手机号
	email
	身份证号
	卡号

	返回，用户所有信息

用户信息
	微信号
	微博号
	QQ号

	传真
	客户级别
	注册ip
	用户状态
	别名
	注册时间

用户信息查询，３个接口，用户信息，用户所有信息。保存一个接口，修改一个接口


动态验证码输入和输出都增加唯一标志


登陆验证
	账号ＩＤ


密保问题查询
	userid 默认　０，输入userid, 查询公共问题和用户私有问题

问题答案查询，只支持一个一个查询

	最多只支持３个问题

密码设置和重置


# 银联交流

现场半相关手续，材料准备


测试环境

接入要求：比如提供ip


联调时间、联调配合人

接入方式：全渠道采用互联网方式（可以专线接入，但是不建议）
清算方式：Ｔ＋１，清算文件采用原有方式，两个机构号为两个文件
业务范围：代收、代付（新模式，银联在邯郸银行开户）、快捷支付、网关支付、退款（原手续费退货，本身交易无手续费）

代收代付：业务有限制，限制范围

业务对口人，阜阳：15232198077
技术对口人：陈姚：18203229290



如果是机构发验证码的特殊要求，流程，接口这些还不清楚？
	


新申请机构号，约10个工作日


需要我行确认
	人行备付金账户开在那个行？
	如果开在其他行需要代理清算协议


系统测试
	需要在机构号、

兴业：费励志



json


# 2016-04-22

## 现有问题整理


# 2016-04-26
## svn
**启动脚本**
```shell
svnserve -d -r ~/dev/subversion-repos --listen-port 8082
```

-d 后台启动
-r 仓库目录
--listen-port 指定端口

**创建仓库**
```shell
vnadmin create /usr/local/svn/newrepos
```

## fastdfs
**启动tracker**
```shell
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
```
**启动storage**
```shell
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf
```
**测试上传**
```shell
/usr/bin/fdfs_test /etc/fdfs/client.conf upload filename
```

**监控**
```shell
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```

ANFBGDE9PJ



无卡取现预约
无卡取现取消
无卡取现查询
＝＝＝＝＝＝＝＝＝没日志

无卡取现密码是否必输

交易明细查询(5435)，借是否代表收入，贷是否代表支出　（有描述）

短信平台
＝＝＝＝＝＝＝＝＝无接口也无日志

	本行借记卡账户验证	本行借记卡账户验证	支付实施厂商睿民提出需求，与账户通用信息查询、校验接口属于同一接口，可以共用
	快捷支付（无磁无密）		支付实施厂商睿民提出需求
	退款		支付实施厂商睿民提出需求
	交易状态查询		支付实施厂商睿民提出需求，与大小额转账结果查询接口接口属于同一接口，可以共用


	5446
	5447

	
sjcx

/cib/log/08

//18831077512 苗晓华


970882116

172.172.1.32

18031

5477081019999cbfe@@@@2010738000544002865999999999999999999999999313127000013313127000013201605121001                00000000000000202511

622960876041307515                            
20160512
20160513                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        :        0      02016051217425499990076





向晟祥您好：