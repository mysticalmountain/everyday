[TOC]
## 2015-10
### 2015-10-14
#### 实时清分需要的基础服务
1. zookeeper：发布dubbo服务，多节点自动任务分布式事物锁
2. MQ：从队列中获取交易数据
3. 统一配置：基础配置初始化

### 2015-10-15
#### 实时清分数据迁移
1. 是否需要从MQ中读消息？
    不需要
2. 是否为核心提供反向交易记账dubbo服务？
    不需要
3. 是否需要记账完成后通知风控跑批？
    由自动日切机制完成
4. 历史表中交易是否有会计日？是否有记账流水号？手续费是否已经计算？
    无会计日，无记账流水号，手续费已经计算

*新建临时表存储手续费*
```sql
create table tmp_trans_clearing_fee(
    id varchar2(20) primary key not null,
    ACQ_MERCHANT_TRANS_FEE varchar2(20),
    ACQ_TRANS_FEE varchar2(20),
    BRAND_SERVICE_FEE varchar2(20),
    MERCHANT_TRANS_FEE varchar2(20),
    MERCHANT_ADDITIONAL_FEE varchar2(20),
    TRANS_REVENUE varchar2(20),
    ACQ_FEE_RATE varchar2(20),
    BRAND_SERVICE_FEE_RATE varchar2(20),
    MERCHANT_FEE_RATE varchar2(20),
    ADDITIONAL_FEE_RATE varchar2(20)
)
```
### 2015-10-19
#### 加班统计
:sweat:加班日期   2015-9-19（已调休）,2015-9-26（已倒休）
#### linux命令
zip name`date +%F`.zip dir
tar zcfv zk-test-`date +%F`.tar.gz zk-test/*
tar --exclude=logs/* --exclude=ac* -zcvf actual-clearing-`date +%y%m%d%H%M%S`.tar.gz ./*
#### 补记账
*交易历史记录和新计算的手续费比较*
```sql
select th.id,
       th.trans_type,
       th.trans_status,
       th.ACQ_MERCHANT_TRANS_FEE,
       trcf.ACQ_MERCHANT_TRANS_FEE, 
       th.ACQ_TRANS_FEE,
       trcf.ACQ_TRANS_FEE,
       th.BRAND_SERVICE_FEE,
       trcf.BRAND_SERVICE_FEE,
       th.MERCHANT_TRANS_FEE,
       trcf.MERCHANT_TRANS_FEE,
       th.MERCHANT_ADDITIONAL_FEE,
       trcf.MERCHANT_ADDITIONAL_FEE,
       th.TRANS_REVENUE,
       trcf.TRANS_REVENUE,
       th.amount,
       th.acq_fee_rate,
       trcf.acq_fee_rate,
       th.brand_service_fee_rate,
       trcf.brand_service_fee_rate,
       th.merchant_fee_rate,
       trcf.merchant_fee_rate,
       th.additional_fee_rate,
       trcf.additional_fee_rate,
       tcs.clearing_status,
       tcs.account_status
  from trans_history th
  left join tmp_trans_clearing_fee trcf on th.id = trcf.id
  left join trans_clearing_status tcs on th.id = tcs.trans_id
 where trans_type in
       ('sale', 'reversal_sale', 'sale_void', 'reversal_sale_void',
        'auth_comp', 'reversal_auth_comp', 'auth_comp_cancel',
        'reversal_auth_comp_cancel', 'refund')
   and trans_status in (1, 2, 3, 4)
   and th.trans_date_time between to_date('20150901', 'YYYYMMDD') and
       to_date('20150901', 'YYYYMMDD') + 1
```
*查询不匹配的记录*
```sql
--非银联通道
select th.id,
       th.trans_type,
       th.trans_status,
       th.ACQ_MERCHANT_TRANS_FEE,
       trcf.ACQ_MERCHANT_TRANS_FEE,
       th.ACQ_TRANS_FEE,
       trcf.ACQ_TRANS_FEE,
       th.BRAND_SERVICE_FEE,
       trcf.BRAND_SERVICE_FEE,
       th.MERCHANT_TRANS_FEE,
       trcf.MERCHANT_TRANS_FEE,
       th.MERCHANT_ADDITIONAL_FEE,
       trcf.MERCHANT_ADDITIONAL_FEE,
       th.TRANS_REVENUE,
       trcf.TRANS_REVENUE,
       th.amount,
       th.acq_fee_rate,
       trcf.acq_fee_rate,
       th.brand_service_fee_rate,
       trcf.brand_service_fee_rate,
       th.merchant_fee_rate,
       trcf.merchant_fee_rate,
       th.additional_fee_rate,
       trcf.additional_fee_rate,
       tcs.clearing_status,
       tcs.account_status
  from trans_history th
  left join tmp_trans_clearing_fee trcf on th.id = trcf.id
  left join trans_clearing_status tcs on th.id = tcs.trans_id
 where trans_type in
       ('sale', 'reversal_sale', 'sale_void', 'reversal_sale_void',
        'auth_comp', 'reversal_auth_comp', 'auth_comp_cancel',
        'reversal_auth_comp_cancel', 'refund')
   and trans_status in (1, 2, 3, 4)
   and (th.ACQ_MERCHANT_TRANS_FEE <> trcf.ACQ_MERCHANT_TRANS_FEE or
       th.ACQ_TRANS_FEE <> trcf.ACQ_TRANS_FEE or
       th.BRAND_SERVICE_FEE <> trcf.BRAND_SERVICE_FEE or
       th.MERCHANT_TRANS_FEE <> trcf.MERCHANT_TRANS_FEE or
       th.MERCHANT_ADDITIONAL_FEE <> trcf.MERCHANT_ADDITIONAL_FEE or
       th.TRANS_REVENUE <> trcf.TRANS_REVENUE or
       th.acq_fee_rate <> trcf.acq_fee_rate or
       th.brand_service_fee_rate <> trcf.brand_service_fee_rate or
       th.merchant_fee_rate <> trcf.merchant_fee_rate or
       th.additional_fee_rate <> trcf.additional_fee_rate)
   and th.trans_date_time between to_date('20150901', 'YYYYMMDD') and
       to_date('20150901', 'YYYYMMDD') + 1
   and acquirer_id <> 47 

--银联通道
select th.id,
       th.trans_type,
       th.trans_status,
       th.ACQ_MERCHANT_TRANS_FEE,
       trcf.ACQ_MERCHANT_TRANS_FEE,
       th.ACQ_TRANS_FEE,
       trcf.ACQ_TRANS_FEE,
       th.BRAND_SERVICE_FEE,
       trcf.BRAND_SERVICE_FEE,
       th.MERCHANT_TRANS_FEE,
       trcf.MERCHANT_TRANS_FEE,
       th.MERCHANT_ADDITIONAL_FEE,
       trcf.MERCHANT_ADDITIONAL_FEE,
       th.TRANS_REVENUE,
       trcf.TRANS_REVENUE,
       th.amount,
       th.acq_fee_rate,
       trcf.acq_fee_rate,
       th.brand_service_fee_rate,
       trcf.brand_service_fee_rate,
       th.merchant_fee_rate,
       trcf.merchant_fee_rate,
       th.additional_fee_rate,
       trcf.additional_fee_rate,
       tcs.clearing_status,
       tcs.account_status
  from trans_history th
  left join tmp_trans_clearing_fee trcf on th.id = trcf.id
  left join trans_clearing_status tcs on th.id = tcs.trans_id
 where trans_type in
       ('sale', 'reversal_sale', 'sale_void', 'reversal_sale_void',
        'auth_comp', 'reversal_auth_comp', 'auth_comp_cancel',
        'reversal_auth_comp_cancel', 'refund')
   and trans_status in (1, 2, 3, 4)
   and (th.MERCHANT_TRANS_FEE <> trcf.MERCHANT_TRANS_FEE or
       th.MERCHANT_ADDITIONAL_FEE <> trcf.MERCHANT_ADDITIONAL_FEE or
       th.merchant_fee_rate <> trcf.merchant_fee_rate or
       th.additional_fee_rate <> trcf.additional_fee_rate)
   and th.trans_date_time between to_date('20150901', 'YYYYMMDD') and
       to_date('20150901', 'YYYYMMDD') + 1
   and acquirer_id = 47
```

### 2015-10-26
#### 实时清分涉及表
DICT_CARDBIN
ACQ_MERCHANT
ACQ_MERCHANT_FEE_RATE
FEE_RATE_SETTING
CM_MERCHANT
MERCHANT_FEE_RATE
ACQUIRER_COST
TRANS_CURRENT
TRANS_CLEARING_STATUS
TRANS_HISTORY
ACQUIRER
SWITCH_DATE
JOB_STATUS
#### 删除性能历史数据
高阳   payadm
truncate table ACMTJNL;
truncate table ACMTCDJN;
truncate table ACTTACJN;
truncate table ACTTACJNDT;
truncate table ACTTINDT;
truncate table ACTTVCH;
truncate table WDCTORDR;

收单  acq_v3
truncate table trans_current;
truncate table trans_history;
truncate table trans_router_area; 
truncate table trans_clearing_status;
truncate table crd_settle_trans_record;
truncate table crd_settle_order;    

#### 数据迁移，导出8月  - 9月历史交易
```sql
select *
  from trans_history
 where trans_date_time between to_date('20150801', 'YYYYMMDD') and
       to_date('20150930', 'YYYYMMDD')
```

1. 可能要开户
2. 反向交易可能因为余额不足记账失败
3. 如何获取生产环境结算端

### 2015-10-29
#### 手续费比较
*211598062*
```sql
select * from cm_merchant where merchant_no = 'Z08000000447569';

select * from DICT_CARDBIN where id = 1524;

select * from trans_history where id = 211598062;

select *
  from cm_merchant cm
  left join merchant_fee_rate mfr on cm.id = mfr.merchant_id
  left join fee_rate_setting frs on mfr.fee_rate_setting_id = frs.id
 where mfr.cardbin_type = 0
   and cm.merchant_no = 'Z08000000447569';

select *
  from cm_merchant cm
  left join settle_setting ss on cm.settle_setting_id = ss.id
 where cm.merchant_no = 'Z08000000447569';
``` 
*211622088*
```sql
select * from cm_merchant where merchant_no = 'Z08000001189382';

select * from DICT_CARDBIN where id = 737;

select * from trans_history  where id = 211622088;

select *
  from cm_merchant cm
  left join merchant_fee_rate mfr on cm.id = mfr.merchant_id
  left join fee_rate_setting frs on mfr.fee_rate_setting_id = frs.id
 where mfr.cardbin_type = 0
   and cm.merchant_no = 'Z08000001189382';

select *
  from cm_merchant cm
  left join settle_setting ss on cm.settle_setting_id = ss.id
 where cm.merchant_no = 'Z08000001189382';
```

*211395029*
```sql
select * from cm_merchant where merchant_no = '909000001170376';

select * from DICT_CARDBIN where id = 741;

select * from trans_history  where id = 211395029;

select *
  from cm_merchant cm
  left join merchant_fee_rate mfr on cm.id = mfr.merchant_id
  left join fee_rate_setting frs on mfr.fee_rate_setting_id = frs.id
 where mfr.cardbin_type = 0
   and cm.merchant_no = '909000001170376';

select *
  from cm_merchant cm
  left join settle_setting ss on cm.settle_setting_id = ss.id
 where cm.merchant_no = '909000001170376';
```

#### 手续费比较（总结）
交易日期：20150929
总交易量：335449笔

商户手续费
手续费计算不匹配数量（消费）：997笔。交易状态为成功32笔，其余交易状态为2或3。经过查找生产日志（立新配合）32笔成功交易都在9月30日修改过商户手续费设置。其余交易为反向交易现有生产环境不做手续费计算。
手续费计算不匹配数量（消费撤销）：10笔。反向交易现有生产环境不做手续费计算。
手续费计算不匹配数量（消费冲正）：441笔。反向交易现有生产环境不做手续费计算。

商户附加手续费
手续费计算不匹配数量（消费）：392笔。该392笔交易附加手续费都为0，经过与立新分析，原因为此类交易并非D+0结算，所以附加手续费为0
手续费计算不匹配数量（消费撤销）：0笔
手续费计算不匹配数量（消费冲正）：0笔

通道交易费
生产所有当天交易通道交易费（ACQ_TRANS_FEE）为都为null
生产所有当天交易通道交易费（ACQ_MERCHANT_TRANS_FEE）存在3512条银联通道手续费不一致。经过与立新沟通，因为目前生产银联通道手续费计算为老版本，最新版本还未上线。所以可能存在不一致的情况。

品牌服务费
生产所有当天交易品牌服务费都为null 


#### 三农手续费计算修改
   /* 设置默认值 xiaoyun 2014-12-02 start */
    //特殊计费类型(银联规范) 00-无特殊计费, 01-周期计费, 02-微额打包, 03-固定比例, 04-县乡优惠, 05-大商户优惠
    String specialBillingType='00'
    //特殊计费档次(详见银联规范)
    String specialBillingLevel='0'
    /* 设置默认值 xiaoyun 2014-12-02 end */

交易金额大于1000 手续费封顶2.4元

通道成本手续费
specialBillingType=04  && specialBillingLevel=1 为三农

商户手续费不影响

手续费计算：三农商户（系统户）交易金额大于1000元手续费为2.4元。金额小于1000元按0.24%计算。

如何判断商户为三农商户：表acq_merchant，列special_Billing_Type = 04 && special_Billing_Level = 1

天津银联费率0.3%

### 2015-11-03
#### 补入账 会计日与交易日
会计日     | 交易日
---------- | -------------
2015-10-19 | 2015-09-29
2015-10-20 | 2015-09-28
2015-10-21 | 2015-09-27
2015-10-22 | 2015-09-21
2015-10-23 | 2015-09-22
2015-10-24 | 2015-09-23
2015-10-25 | 2015-09-24
2015-10-30 | 2015-09-07
2015-10-31 | 2015-09-08
2015-11-01 | 2015-09-09


2015-10-23 ：17:43

#### 结算单比较
会计日     | 交易日
---------- | -------------
2015-10-23 | 2015-09-22

*总笔数*

记账       | boss
---------- | -------------
105347     | 106220

- [x] 补入账总结
会计日     | 交易日 
---------- | -------------
2015-10-30 | 2015-09-07
2015-10-31 | 2015-09-08

高阳付款日：2015-11-01
BOSS结算单日：2015-09-09

高阳总记录数：101812，高阳总金额：1,219,438,148.37
boss总记录数：103713，boss总金额：1,401,037,834.18

经过结算单记录比较
商户结算金额匹配的商户总数：100139，金额：1,174,285,186
商户结算金额不匹配的商户总数：759

boss结算商户有但高阳结算单无总数：2815
boss结算商户无但高阳结算单有总数：914

金额差异分析
      随机抽取金额差异商户



结算单中商户，boss有但高阳无分析

结算单中商户，boss无但高阳有分析


### 2015-11-04
#### JVM 监控
```java
jstack 21711
jmap -permstat pid
jmap -dump:format=b,file=dumpFileName
jmap -heap 21711
jstat -gc 21711 250 4
jhat -port 9998 /tmp/dump.dat
```
Yong Generation
  Eden
  Surviver
    From
    To
Old
Permament
### JVM启动参数
```java
JAVA_OPTS="-server -Xmx1G -Xms1G -XX:MaxPermSize=256M -XX:PermSize=256M -Xss256k
            -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled
            -XX:+UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=0
            -XX:+CMSClassUnloadingEnabled -XX:LargePageSizeInBytes=128M
            -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly
            -XX:+HeapDumpOnOutOfMemoryError
            -XX:+PrintGCDetails
            -XX:+PrintGCDateStamps
            -Xloggc:$APP_HOME/logs/gc.log
            -Duser.timezone=Asia/Shanghai -Dfile.encoding=UTF-8 -Dlocal=false"

2012-11-15T16:57:06.118+0800: 2.089: [GC 2.089: [ParNew: 104960K->13056K(118016K), 0.1720037 secs] 104960K->48670K(511232K), 0.1721610 secs] [Times: user=0.28 sys=0.01, real=0.17 secs] 
2012-11-15T16:57:07.618+0800: 3.583: [GC 3.583: [ParNew: 117987K->13056K(118016K), 0.0986891 secs] 153601K->64983K(511232K), 0.0988357 secs] [Times: user=0.14 sys=0.01, real=0.09 secs] 
2012-11-15T16:57:10.696+0800: 6.669: [GC 6.669: [ParNew: 118016K->13056K(118016K), 0.0685054 secs] 169943K->78453K(511232K), 0.0686582 secs] [Times: user=0.14 sys=0.00, real=0.08 secs] 
2012-11-15T16:57:12.524+0800: 8.490: [GC 8.490: [ParNew: 118016K->11244K(118016K), 0.0525525 secs] 183413K->83007K(511232K), 0.0527229 secs] [Times: user=0.08 sys=0.00, real=0.05 secs]

以其中一例分析：

2012-11-15T16:57:12.524+0800: 8.490: [GC 8.490: [ParNew: 118016K->11244K(118016K), 0.0525525 secs] 183413K->83007K(511232K), 0.0527229 secs] [Times: user=0.08 sys=0.00, real=0.05 secs]

8.490 ：表示虚拟机启动运行到8.490秒是进行了一次monor Gc ( not Full GC)

ParNew :表示对年轻代进行的GC，使用ParNew 收集器

118016K->11244K(118016K) ：118016K  年轻代收集前大小，11244K 收集完以后的大小 ， 118016K  当前年轻代分配的总大小

0.0525525 secs ：表示对年轻代进行垃圾收集是，用户线程暂停的时间，即此次年轻代收集花费的时间

183413K->83007K(511232K) :JVM heap堆收集前后heap堆内存的变化

0.0527229 secs ：整个JVM此次垃圾造成用户线程的暂停时间。
```

### 2015-11-6
#### 插入排序算法
- [x] 示例
```java
void InsertSort(int[] a, int n) {
    for (int i = 1; i < n; i++) {
        if (a[i] < a[i - 1]) {               //若第i个元素大于i-1元素，直接插入。小于的话，移动有序表后插入
            int j = i - 1;
            int x = a[i];        //复制为哨兵，即存储待排序元素
            a[i] = a[i - 1];           //先后移一个元素
            while (x < a[j]) {  //查找在有序表的插入位置
                a[j + 1] = a[j];
                if (j == 0) {
                    break
                }
                j--;         //元素后移
            }
            a[j + 1] = x;      //插入到正确位置
        }
    }
}
```
- [x] 时间复杂度
1 + 2 + 3 + 4 + ... n

### 2015-11-10
#### HashMap
1 << 4
1 -> 00000001
1 << 4 -> 00010000

### 2015-11-11
#### zookeeper应用场景
 命名服务
 配置管理
 集群管理
 分布式锁

### 2015-11-19
/master
/workers
/assign
/tasks
/status



手续费计算
记账

两个独立的模块，相互之间无直接依赖

### 2015-11-23
#### 实时清分改造
- 交易可以选择性入账，可以灵活的配置规则，（规则考虑多维度）
入账前检查
基础：交易类型、交易状态
业务：通道规则、商户规则

- 考虑监控



Handler 增加 check list，可以为handler注册多个check，交易前先执行check list

#### UML各种关系说明
UML中描述对象和类之间相互关系的方式包括：依赖（Dependency），关联（Association），聚合（Aggregation），组合（Composition），泛化（Generalization），实现（Realization）等。
- [x] 依赖（denpendency）
元素A的变化会影响元素B，但反之不成立，那么B和A间的关系为依赖，B依赖A；类属关系和实现关系在语义上讲也是依赖关系，但由于其有更特殊的用途，所以被单独描述。uml中用带箭头的虚线表示Dependency关系，箭头指向被依赖元素。
- [x] 泛化（Generalization）
通常所说的继承（特殊个体 is kind of 一般个体）关系，不必多解释了。uml中用带空心箭头的实线表示Generalization关系，箭头指向一般个体。
- [x] 实现（Realize）
元素A定义一个约定，元素B实现这个约定，则B和A的关系是Realize，B realize A。这个关系最常用于接口。uml中用空心箭头和虚线表示Realize关系，箭头指向定义约定的元素。
- [x] 关联（Association）
元素间的结构化关系，是一种弱关系，被关联的元素间通常可以被独立的考虑。uml中用实线表示Association关系，箭头指向被依赖元素。
- [ ] 聚合（Aggregation）
关联关系的一种特例，表示部分和整体（整体 has a 部分）的关系。uml中用带空心菱形头的实线表示Aggregation关系，菱形头指向整体。
- [ ] 组合（Composition）
组合是聚合关系的变种，表示元素间更强的组合关系。如果是组合关系，如果整体被破坏则个体一定会被破坏，而聚合的个体则可能是被多个整体所共享的，不一定会随着某个整体的破坏而被破坏。uml中用带实心菱形头的实线表示Composition关系，菱形头指向整体。


