[TOC]

#　Fastdfs install
## 安装libfastcommon
```shell
tar -xvf libfastcommon-1.0.7.tar.gz  
cd libfastcommon-1.0.7  
./make.sh  
sudo ./make.sh install
```
输出
```
mkdir -p /usr/lib64
install -m 755 libfastcommon.so /usr/lib64
mkdir -p /usr/include/fastcommon
install -m 644 common_define.h hash.h chain.h logger.h base64.h shared_func.h pthread_func.h ini_file_reader.h _os_bits.h sockopt.h sched_thread.h http_func.h md5.h local_ip_func.h avl_tree.h ioevent.h ioevent_loop.h fast_task_queue.h fast_timer.h process_ctrl.h fast_mblock.h connection_pool.h /usr/include/fastcommon
```

> 注意安装过程中的输出信息，如果没有报错就表示libfastcommon安装成功了。 由于libfastcommon.so默认安装到了/usr/lib64/libfastcommon.so，而FastDFS主程序设置的lib目录是/usr/local/lib，所以需要设置软连接：

```shell
sudo ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
```

## 安装fastdfs
```shell
tar -zxvf fastdfs-5.04.tar.gz
cd fastdfs-5.04
./make.sh
sudo ./make.sh install
```

### 复制配置文件
```shell
cd /etc/fdfs
sudo cp client.conf.sample client.conf
sudo cp storage.conf.sample storage.conf
sudo cp tracker.conf.sample tracker.conf
```

### 配置tracker.conf
```shell
sodo vi tracker.conf

bind_addr=172.21.11.131
base_path=/home/assist/fastdfs
```
> bing_add = ip (不可写localhost 或者　127.0.0.1)

### 启动tracker服务
```shell
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
```

查看是否已经启动
```shell
[assist@ASSIST logs]$ ps -ef | grep fdfs
assist    2723     1  0 16:40 ?        00:00:00 /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf

[assist@ASSIST logs]$ netstat -anp | grep 22122
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 172.21.11.133:22122         0.0.0.0:*                   LISTEN      2723/fdfs_trackerd  

```

> 如果启动失败，去目录　～/fastdfs/logs 查看日志

### 配置storage
```shell
sudo vi /etc/fdfs/storage.conf

bind_addr=172.21.11.133
base_path=/home/assist/fastdfs
store_path0=/home/assist/fastdfs
tracker_server=172.21.11.133:22122
```
　
### 启动storage服务
```shell
[assist@ASSIST fdfs]$ /usr/bin/fdfs_storaged /etc/fdfs/storage.conf
[assist@ASSIST fdfs]$ ps -ef | grep fdfs
assist    2723     1  0 16:40 ?        00:00:00 /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
assist    2806     1  0 16:57 ?        00:00:00 /usr/bin/fdfs_storaged /etc/fdfs/storage.conf
assist    2816  1855  0 16:57 pts/2    00:00:00 grep fdfs
```

### 配置client
```shell
sudo vi /etc/fdfs/client.conf

base_path=/home/assist/fastdfs
tracker_server=172.21.11.133:22122
```

### 测试上传
1. 上传
```shell
[assist@ASSIST fdfs]$ /usr/bin/fdfs_upload_file /etc/fdfs/client.conf tracker.conf.sample 
group1/M00/00/00/rBULhVc64s6AaB3dAAAbvu2aF8c.sample
```
2. 查看文件是否存在
```shell
[assist@ASSIST fdfs]$ ll ~/fastdfs/data/00/00/
-rw-r--r-- 1 assist bank 7102 5ÔÂ  17 17:22 rBULhVc64s6AaB3dAAAbvu2aF8c.sample
```

> 文件名相同，所以上传成功