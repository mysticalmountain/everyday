# 安装
1. 创建yum目录
```
sudo mkdir /var/yum
```
2. 复制系统软件包到/var/yum
3. 安装软件包`createrepo`
```
sudo rpm -ivh createrepo-0.4.4-2.fc6.noarch.rpm
```
*注意：先检查软件包是否已经安装*`rpm -qa createrepo`
4. 创建yum仓库
```
createrepo -g /var/yum/repodata/comps-rhel5-server-core.xml /var/yum/
```
5.配置本地yum客户端
```
vim /etc/yum.repo.d/local.repo

输入如下内容
[base] #（yum块区域）
name=liusuping.com # (名字可以随便起)
baseurl=file:///var/yum #（搜索路径，必须指向你本机的yum源路径）
gpgcheck=0 #（gpgcheck是gpg验证是否开启的选项，1是开启，0是不开启，一般情况可以关掉）
enabled=1 #(是否启用，0为不启用，1为启用，过没这一项，就是启用）
```
6. 查看是否安装成功
```
yum list
```
7. 安装软件包
            ```
            yum install xxx
            ```
