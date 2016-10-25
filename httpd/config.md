
# 开启代理模块
```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_connect_module modules/mod_proxy_connect.so
LoadModule proxy_ftp_module modules/mod_proxy_ftp.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```
# 引入vhost文件
```
Include conf/extra/httpd-vhosts.conf
```
# 增加端口
```
Listen 8080
Listen 8081
```

# 配置httpd-vhosts.conf
```xml
<VirtualHost *:8080> 
       ServerAdmin test@test.com 
       ServerName 127.0.0.1:8080
       ProxyRequests Off 
       <Proxy *> 
           Order deny,allow 
           Allow from all 
       </Proxy> 
       ProxyPass / http://192.168.1.24:8080/
       ProxyPassReverse / http://192.168.1.24:8080/
</VirtualHost>

<VirtualHost *:8081> 
       ServerAdmin test@test.com 
       ServerName 127.0.0.1:8081
       ProxyRequests Off 
       <Proxy *> 
           Order deny,allow 
           Allow from all 
       </Proxy> 
       ProxyPass / http://192.168.1.24:8081/
       ProxyPassReverse / http://192.168.1.24:8081/
</VirtualHost>
```