# 安装nginx和fasdfs nginx module

## 安装 pcre
```shell
./configure
make
sudo make install
```

## 安装 zlib
```shell
./configure
make
sudo make install
```

## 安装nginx
```shell
./configure --prefix=/usr/local/nginx --add-module=/home/yq/fastdfs-nginx-module/src
make
sudo make install
```

## 配置nginx
```shell
sudo vi /usr/local/nginx/conf/nginx.conf
在server节点加入下面的配置
location /group1/M00{
    root /home/assist/fastdfs/data;
    ngx_fastdfs_module;
}

user root


cp fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/

base_path=/home/assist/fastdfs
racker_server=172.21.11.133:22122
url_have_group_name = true
store_path0=/home/assist/

cp ~/fastdfs-5.04/conf/http.conf /etc/fdfs/
cp ~/fastdfs-5.04/conf//mime.types /etc/fdfs/

sudo ln -s /home/assist/fastdfs/data /home/assist/fastdfs/data/M00
```

## 启动nginx
```shell
sudo /usr/local/nginx/sbin/nginx

ps -ef | grep nginx

    启动: ./nginx
    重启：./nginx -s reload
    停止：./nginx -s stop
```





# 安装问题解答
## 问题fastdfs-nginx-module/src//ngx_http_fastdfs_module.c 找不到目录
检查fastdfs-nginx-module config
```shell
vi fastdfs-nginx-module/config
删除下行的local
CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/"
```

# 启动nginx问题解答
## 问题 /usr/local/nginx/sbin/nginx: error while loading shared libraries: libpcre.so.1: cannot open shared object file: No such file or directory
```shell
ln -s /usr/local/lib/libpcre.so.1 /lib64/
```

## 启动日志报错
```
[2016-05-18 11:43:57] ERROR - file: shared_func.c, line: 960, open file /etc/fdfs/mod_fastdfs.conf fail, errno: 2, error info: No such file or directory
[2016-05-18 11:43:57] ERROR - file: /home/assist/hdyhserver/software/fastdfs-nginx-module/src/common.c, line: 155, load conf file "/etc/fdfs/mod_fastdfs.conf" fail, ret code: 2
```


