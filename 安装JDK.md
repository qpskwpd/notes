### 安装JDK

1. 解压JDK至`/usr/local/jdk`

2. 配置环境变量`vim /etc/profile`
   
    ```shell
    JAVA_HOME=/usr/local/jdk11/jdk-11
    CLASSPATH=$JAVA_HOME/lib/
    PATH=$PATH:$JAVA_HOME/bin
    export JAVA_HOME CLASSPATH PATH
    ```
    
3. 更新profile
    `source /etc/profile`
    
4. 查看版本信息
    `java -version`

### 查看已安装JDK信息

  	`rpm -qa | grep java`

### 删除JDK

  	`rpm -e -nodeps 查询的jdk信息`



### 安装MySQL

1.  解压MySQL包

2.  `rpm -ivh mysql-community-common-8.0.20-1.el8.x86_64.rpm`

3.  `rpm -ivh mysql-community-libs-8.0.20-1.el8.x86_64.rpm`

4.  `rpm -ivh mysql-community-client-8.0.20-1.el8.x86_64.rpm`

5.  `rpm -ivh mysql-community-server-8.0.20-1.el8.x86_64.rpm` 

6. 启动`mysql service mysqld start`

7. 查找初始密码 `cat /var/log/mysqld.log`

8. 登录 `mysql -uroot -p`

9. 修改密码 `ALTER USER 'root'@'localhost' IDENTIFIED BY 'Mysql@wpd222.com';`

### 开启远程访问

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '密码';`

`grant all privileges on 数据库名.表名 to '用户名' @'host名' identified by '密码';`

- 创建账户

  `create user 'root'@'172.16.10.203' identified by  'password'`

- 赋予权限，with grant option这个选项表示该用户可以将自己拥有的权限授权给别人
  `grant all privileges on *.* to 'root'@'172.16.10.203' with grant option`

- 改密码&授权超用户，`flush privileges` 命令本质上的作用是将当前user和privilige表中的用户信息/权限设置从mysql库(MySQL数据库的内置库)中提取到内存里
  `flush privileges;`

  ————————————————
  版权声明：本文为CSDN博主「common_util」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
  原文链接：https://blog.csdn.net/shenhonglei1234/java/article/details/84786443



### Tomcat资源部署

将war包放入webapps目录下



### 安装Redis

1. 解压redis包

2. 编译`make`

3. 执行src中的两个文件即可



### 安装Maven

1. 解压

2. 配置环境变量，编辑/etc/profile

   ```
   export MAVEN_HOME=/usr/local/maven/apache-maven-3.6.3
   export PATH=$MAVEN_HOME/bin:$PATH
   ```

3. source /etc/profile



### 安装Nginx

1. 安装依赖包

   ```shell
   yum install -y pcre pcre-devel
   
   yum install -y zlib zlib-devel
   
   yum install -y openssl openssl-devel
   ```

2. 执行配置文件

   ```shell
   ./configure --prefix=/usr/local/nginx --pid-path=/var/run/nginx/nginx.pid --lock-path=/var/lock/nginx.lock --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-http_gzip_static_module --http-client-body-temp-path=/var/temp/nginx/client --http-proxy-temp-path=/var/temp/nginx/proxy --http-fastcgi-temp-path=/var/temp/nginx/fastcgi --http-uwsgi-temp-path=/var/temp/nginx/uwsgi --http-scgi-temp-path=/var/temp/nginx/scgi
   ```

3. 编译`make`

4. 安装`make install`

5. 创建目录

   `mkdir /var/temp/nginx/client -p`

6. 进入`/user/local/nginx/sbin`中启动

7. 停止 `./nginx -s stop或quit` 重新加载`./nginx -s reload`

### Nginx部署静态资源

1. 将资源目录放入nginx目录下
2. 修改conf目录下nginx.conf中server部分

### Nginx作为虚拟主机部署多个资源

在conf目录下nginx.conf中添加新的server部分

可通过端口方式/域名方式区分多个资源的访问路径

### Nginx反向代理&负载均衡

    upstream tomcat-travel{
            server 192.168.208.200:8080 weight=2;#只能写到端口号,因此tomcat项目需要部署为ROOT
            server 192.168.208.200:8081 weight=1;
            server 192.168.208.200:8082 weight=1;
            #这里可以配置多个server
        }
    server {
        listen       80;
        server_name  localhost;
    
        #charset koi8-r;
    
        #access_log  logs/host.access.log  main;
    
        location / {
            #root   index;
            proxy_pass http://tomcat-travel;
            index  index.html index.htm;
        }
        ...
    }



### 设置防火墙端口开放

`firewall-cmd --zone=public --add-port=80/tcp (永久生效再加上 --permanent)`



### 修改ip地址

- 有的系统有sysconfig文件夹，有的系统是在network文件夹

1. `vim /etc/sysconfig/network-scripts/ifcfg-ens160`

2. ```properties
   TYPE=Ethernet
   PROXY_METHOD=none
   BROWSER_ONLY=no
   BOOTPROTO=static //dhcp动态主机配置协议
   DEFROUTE=yes
   IPV4_FAILURE_FATAL=no
   IPV6INIT=yes
   IPV6_AUTOCONF=yes
   IPV6_DEFROUTE=yes
   IPV6_FAILURE_FATAL=no
   IPV6_ADDR_GEN_MODE=stable-privacy
   NAME=ens160
   UUID=c83e05b8-2c20-4bc9-b634-93547ecf1944
   DEVICE=ens160
   ONBOOT=yes //开机启动连接
   IPADDR=192.168.208.200 //设置的静态ip
   NETMASK=255.255.255.0 //子网掩码
   GATEWAY=192.168.208.2 //网关 需要找对
   DNS1=202.97.131.178 //当前网络所使用的dns服务器
DNS2=202.99.216.113
   ```
   



1. `vim /etc/network/interfaces`

2. ```yml
   #auto ens33 #网卡名称
   #iface ens33 inet dhcp #自动动态配置ip
   
   auto ens33 #网卡名称
   iface ens33 inet static #静态ip
   address 192.168.1.194 #ip地址
   netmask 255.255.255.0 #子网掩码
   gateway 192.168.1.1 #网关
   ```

3. 查看dns地址`/etc/resolv.conf`

4. 如果`/etc/resolvconf/resolv.conf.d/base`中没有信息需要把dns地址添加进来。