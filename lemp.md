# `Docker` `LEMP` --文档

@(Docker--项目)


# 目录


[I.`dockerfile` 结构](#idockerfile结构)  
- [`nginx.conf`](#nginxconf)  
- [`index.html`](#indexhtml)  
- [`php-fpm.conf`](#php-fpmconf)  
- [`phpinfo`](#phpinfo)  
- [连接`mysql`测试脚本](#连接mysql测试脚本)  

[II.`lemp`结构](#iilemp结构)  
- [`mysql` `Dockerfile`](#mysql-dockerfile)  
- [`mysql` `supervisord.conf`](#mysql-supervisordconf)  
- [`nginx` `Dockerfile`](#nginx-dockerfile)  
- [`nginx` `supervisord.conf`](#nginx-supervisordconf)  
- [`php` `Dockerfile`](#php-dockerfile)  
- [`php` `supervisord.conf`](#php-supervisordconf)  

[III.`docker-compose`结构](#iiidocker-compose结构)  
- [`docker-compose.yml`](#docker-composeyml)  
- [`docker` `images`](#docker-images)  

[IV.`lemp` 部署](#ivlemp-部署)  
- [`docker-compose`部署`lemp`](#docker-compose部署lemp)  
- [查看`lemp`运行状态](#查看lemp运行状态)  
- [查看`lemp` `IP地址`](#查看lemp-ip地址)  
- [`ssh`登录`mysql`](#ssh登录mysql)  
- [`mysql`添加授权用户](#mysql添加授权用户)  

[V.`lemp`测试](#vlemp测试)  
- [`nginx`测试](#nginx测试)  
- [`php`测试](#php测试)  
- [`php`连接`MySQL`测试](#php连接mysql测试)  


I.`dockerfile`结构
===
**[回目录](#目录)**

```bash
[root@greately ~]# tree dockerfile/
dockerfile/
├── mysql
├── nginx
│   ├── conf
│   │   ├── fastcgi.conf
│   │   ├── fastcgi_params
│   │   ├── koi-utf
│   │   ├── koi-win
│   │   ├── mime.types
│   │   ├── nginx.conf
│   │   ├── scgi_params
│   │   ├── uwsgi_params
│   │   └── win-utf
│   └── html
│       └── index.html
└── php
    ├── conf
    ├── etc
    │   ├── php-fpm.conf
    │   ├── php.ini
    │   ├── supervisord.log
    │   └── supervisord.pid
    └── webphp
        ├── jason.php
        └── pinfo.php

8 directories, 16 files
```

`nginx.conf`
---
**[回目录](#目录)**

```bash
[root@greately conf]# cat nginx.conf 

user root;
worker_processes  1;

error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        charset utf-8;

        #access_log  logs/host.access.log  main;

        location / {
            root   /var/www/html;
            index  index.php index.html index.htm;
#	    proxy_pass http://proxy_pass; 
       }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
#        location ~ \.php$ {
#            proxy_pass  http://172.17.0.3:80;
#         }

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            root           /var/www/html/webphp;
            fastcgi_pass   172.17.0.3:9000;
            fastcgi_index  index.php;
#            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi.conf;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

#    upstream php {
#	 server 172.17.0.3:9000;
#     }
}
```
<font color=red>**关键点**</font>：`fastcgi_pass   172.17.0.3:9000;`

`index.html`
---
**[回目录](#目录)**

```vbscript-html
[root@greately html]# cat index.html 
<title>Dockerfile Nginx</title>
<h1>This is test<font color=red> docker-compose</font> !</h1>
```

`php-fpm.conf`
---
**[回目录](#目录)**

```bash
[root@greately etc]# cat php-fpm.conf
...//省略
pid = run/php-fpm.pid
user = php
group = php
listen = 0.0.0.0:9000
pm.max_children = 50
pm.start_servers = 20
pm.min_spare_servers = 5
pm.max_spare_servers = 35
...//省略
```

`phpinfo`
---
**[回目录](#目录)**

```bash
[root@greately webphp]# cat pinfo.php 
<?php
phpinfo();
?>
```

连接`mysql`测试脚本
---
**[回目录](#目录)**

```bash
[root@greately webphp]# cat jason.php 
<?php
$link=mysql_connect('172.17.0.2','jason','123123');
if($link) echo "<h1> Congratulations! connected</h1>";
mysql_close();
?>
```

II.`lemp`结构
===
**[回目录](#目录)**


```bash
[root@greately ~]# tree lemp
lemp
├── mysql
│   ├── authorized_keys
│   ├── cmake-2.8.12.tar.gz
│   ├── Dockerfile
│   ├── mysql-5.5.38.tar.gz
│   └── supervisord.conf
├── nginx
│   ├── authorized_keys
│   ├── Dockerfile
│   ├── get-pip.py
│   ├── nginx-1.6.2.tar.gz
│   └── supervisord.conf
└── php
    ├── authorized_keys
    ├── Dockerfile
    ├── get-pip.py
    ├── php-5.3.28.tar.gz
    └── supervisord.conf

3 directories, 15 files
```

`mysql` `Dockerfile`
---
**[回目录](#目录)**

```bash
[root@greately mysql]# cat Dockerfile 
FROM docker.io/centos:centos6
MAINTAINER The fifth group
RUN yum -y install gcc gcc-c++ ncurses-devel python-setuptools openssh-server
RUN easy_install supervisor
RUN useradd admin
RUN echo "admin:admin" | chpasswd
RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN mkdir -pv /var/run/sshd
RUN mkdir -pv /home/admin/.ssh
RUN sed -i 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd
ADD authorized_keys /home/admin/.ssh/authorized_keys
ADD cmake-2.8.12.tar.gz /usr/local/
WORKDIR /usr/local/cmake-2.8.12
RUN ./configure && gmake && gmake install
ADD mysql-5.5.38.tar.gz /usr/local/
WORKDIR /usr/local/mysql-5.5.38
RUN cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_EXTRA_CHARSETS=all -DSYSCONFDIR=/etc
RUN make && make install
RUN /bin/cp support-files/my-medium.cnf /etc/my.cnf
RUN /bin/cp support-files/mysql.server /etc/rc.d/init.d/mysqld
RUN chmod +x /etc/rc.d/init.d/mysqld && chkconfig --add mysqld && chkconfig mysqld on
RUN echo "PATH=$PATH:/usr/local/mysql/bin" >> /etc/profile &&  source /etc/profile
RUN ln -sv /usr/local/mysql/bin/ /usr/bin/mysql
RUN groupadd mysql
RUN useradd -M -s /sbin/nologin mysql -g mysql
RUN chown -R mysql:mysql /usr/local/mysql
RUN /usr/local/mysql/scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql > /dev/nul
COPY supervisord.conf /etc/supervisord.conf
EXPOSE 3306
EXPOSE 22
CMD ["/usr/bin/supervisord","-c","/etc/supervisord.conf"]
```

`mysql` `supervisord.conf`
---
**[回目录](#目录)**

```bash
[root@greately mysql]# cat supervisord.conf 
[supervisord]
nodaemon=true

[program:sshd]
command=/usr/sbin/sshd -D

[program:mysql]
command=/usr/local/mysql/bin/mysqld_safe

autostart=true
autorestart=true
priority=5
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

`nginx` `Dockerfile`
---
**[回目录](#目录)**

```bash
[root@greately nginx]# cat Dockerfile 
FROM docker.io/centos:centos6
MAINTAINER the fifth groups
RUN yum -y install pcre-devel zlib-devel gcc gcc-c++ openssh-server
ADD get-pip.py ./
RUN python get-pip.py
RUN pip install supervisor
RUN echo "root:123123" | chpasswd
RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN mkdir -pv /var/run/sshd
RUN mkdir -pv /root/.ssh
RUN sed -i 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd
ADD authorized_keys /root/.ssh/authorized_keys
COPY nginx-1.6.2.tar.gz /usr/local/
RUN useradd nginx -M -s /sbin/nologin
WORKDIR /usr/local/
RUN tar zxf /usr/local/nginx-1.6.2.tar.gz
WORKDIR /usr/local/nginx-1.6.2
RUN ./configure --prefix=/usr/local/nginx --user=nginx --group=nginx && make && make install
RUN ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin
ADD supervisord.conf /etc/supervisord.conf
EXPOSE 80 22
CMD ["/usr/bin/supervisord","-c", "/etc/supervisord.conf"]
```

`nginx` `supervisord.conf`
---
**[回目录](#目录)**

```bash
[root@greately nginx]# cat supervisord.conf 
[supervisord]
nodaemon=true

[program:sshd]
command=/usr/sbin/sshd -D

[program:nginx]
command=/usr/local/nginx/sbin/nginx -g "daemon off;"
autostart=true
autorestart=true
priority=5
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

`php` `Dockerfile`
---
**[回目录](#目录)**

```bash
[root@greately php]# cat Dockerfile 
FROM docker.io/centos:centos6
MAINTAINER the fifth groups
RUN yum -y install gd libxml2-devel libjpeg-devel libpng-devel mysql-devel gcc gcc-c++ openssh-server
ADD get-pip.py ./
RUN python get-pip.py 
RUN pip install supervisor
RUN useradd -M -s /sbin/nologin php
RUN echo "root:123123" | chpasswd
RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN mkdir -pv /var/run/sshd
RUN mkdir -pv /root/.ssh
RUN sed -i 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd
ADD authorized_keys /root/.ssh/authorized_keys
ADD php-5.3.28.tar.gz /usr/local/
WORKDIR /usr/local/php-5.3.28/
RUN /bin/cp /usr/lib64/mysql/libmysqlclient.so.16.0.0 /usr/lib/libmysqlclient.so
RUN ./configure --prefix=/usr/local/php --with-gd --with-zlib --with-mysql --with-mysqli --with-mysql-sock --with-config-file-path=/usr/local/php --enable-mbstring --enable-fpm --with-jpeg-dir=/usr/lib && make && make install
WORKDIR /usr/local/php/etc/
RUN cp php-fpm.conf.default php-fpm.conf
RUN mkdir -pv /var/www/html/webphp
RUN echo "<?phpinfo();?>" > /var/www/html/webphp/index.php
ADD supervisord.conf /etc/supervisord.conf
EXPOSE 22
CMD ["/usr/bin/supervisord","-c", "/etc/supervisord.conf"]
```
`php` `supervisord.conf`
---
**[回目录](#目录)**

```bash
[root@greately php]# cat supervisord.conf 
[supervisord]
nodaemon=true

[program:sshd]
command=/usr/sbin/sshd -D

[program:php]
command=/usr/local/php/sbin/php-fpm -F
autostart=true
autorestart=true
priority=5
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

III.`docker-compose`结构
===
**[回目录](#目录)**

```bash
[root@greately ~]# tree docker-compose/
docker-compose/
└── docker-compose.yml

0 directories, 1 file
```

`docker-compose.yml`
---
**[回目录](#目录)**

```bash
[root@greately docker-compose]# cat docker-compose.yml 
web_nginx:
  image: centos6:nginx_8
  restart: always
  volumes:
    - /root/dockerfile/nginx/conf/:/usr/local/nginx/conf/
    - /root/dockerfile/nginx/html/:/var/www/html/
  ports:
    - 80:80
  links:
    - web_php

web_php:
  image: centos6:php_12
  restart: always
  volumes:
    - /root/dockerfile/php/etc/:/usr/local/php/etc/
    - /root/dockerfile/php/webphp/:/var/www/html/webphp/
  expose:
    - 9000
  links:
    - web_mysql

web_mysql:
  image: centos6:mysql_5
  restart: always
  ports:
    - 22 
  expose:
    - 3306
```
`docker` `images` 
---
**[回目录](#目录)**

```bash
[root@greately ~]# docker images 
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos6             tomcat_6            7f9fc6de00a4        2 hours ago         975.5 MB
centos6             php_12              c807625a4ef5        9 hours ago         668.1 MB
centos6             php_11              31205f6b654c        9 hours ago         668.1 MB
centos6             nginx_8             f5f4113db768        9 hours ago         383 MB
centos6             php_10              675f18d33d61        9 hours ago         668.1 MB
centos6             test_root_ssh       d952fd987b69        10 hours ago        296.8 MB
centos6             mysql_5             a45df092bff1        11 hours ago        1.892 GB
centos6             mysql4              ac25a438bfab        21 hours ago        1.892 GB
centos6             nginx_6             d15591780d42        45 hours ago        341.9 MB
centos6             php_9               c592b343fad6        2 days ago          668.1 MB
centos6             tomcat7_success     cb8bc5b09229        9 days ago          666.6 MB
centos              nginx               ece6c63adc9e        2 weeks ago         660.4 MB
centos              php                 fcf999e9f431        2 weeks ago         483.5 MB
docker.io/tomcat    latest              30d95ba23356        3 weeks ago         355.3 MB
docker.io/mysql     latest              18f13d72f7f0        3 weeks ago         383.4 MB
docker.io/centos    centos6             cf2c3ece5e41        3 months ago        194.6 MB
```
`compose`中使用的`image`是在对应的路径下面执行

`docker build -t "centos6:xxx" .`生成的



IV.`lemp` 部署
===
**[回目录](#目录)**

`docker-compose`部署`lemp`
---
**[回目录](#目录)**

```bash
[root@greately docker-compose]# docker-compose up -d 
Creating dockercompose_web_mysql_1
Creating dockercompose_web_php_1
Creating dockercompose_web_nginx_1
```
查看`lemp`运行状态
----
**[回目录](#目录)**

```bash
[root@greately docker-compose]# docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                             NAMES
81330f98e985        centos6:nginx_8     "/usr/bin/supervisord"   4 seconds ago       Up 3 seconds        22/tcp, 0.0.0.0:80->80/tcp        dockercompose_web_nginx_1
7ec397bde24f        centos6:php_12      "/usr/bin/supervisord"   5 seconds ago       Up 4 seconds        22/tcp, 9000/tcp                  dockercompose_web_php_1
0098f963ae25        centos6:mysql_5     "/usr/bin/supervisord"   5 seconds ago       Up 5 seconds        3306/tcp, 0.0.0.0:32776->22/tcp   dockercompose_web_mysql_1
```

查看`lemp` `IP`地址
---
**[回目录](#目录)**

>宿主机

```bash
[root@greately ~]# ifconfig 
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0
        inet6 fe80::42:faff:fed8:403e  prefixlen 64  scopeid 0x20<link>
        ether 02:42:fa:d8:40:3e  txqueuelen 0  (Ethernet)
        RX packets 1136  bytes 267880 (261.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1340  bytes 148122 (144.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eno16777736: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.142.17  netmask 255.255.255.0  broadcast 192.168.142.255
        inet6 fe80::20c:29ff:fe68:c48d  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:68:c4:8d  txqueuelen 1000  (Ethernet)
        RX packets 13399  bytes 1119360 (1.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6148  bytes 1110501 (1.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

>nginx

```bash
[root@greately ~]# docker inspect --format '{{.NetworkSettings.IPAddress }}' dockercompose_web_nginx_1 
172.17.0.4
```

>php

```bash
[root@greately ~]# docker inspect --format '{{.NetworkSettings.IPAddress }}' dockercompose_web_php_1 
172.17.0.3
```
>mysql

```bash
[root@greately ~]# docker inspect --format '{{.NetworkSettings.IPAddress }}' dockercompose_web_mysql_1 
172.17.0.2
```

```bash
[root@greately ~]# cd ~/.ssh/
[root@greately .ssh]# cat known_hosts
172.17.0.2 ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAuagCJWIO7827LrxoReuo7xJ6ZsoA26c/ugVdO4GWeGVZ/SpskFqBFFRb79pGIVe8iamfUrhEQcVGxA2RlFgG6Vv7c/w3xS8iWky2NN9lMeuVrDu4cBOK4kgIi6p4gDAG7h0z8+RCcf3cCA3T3Mbtv+AFzO9TbWigLGqmxtoF7OCAFCxlCDBKDrrDrwSHaaiQCS4w7TXJ3winylhWFXyY53aYAZoMi35oo0hExCxpKDmfmGFlTTNR8EFUQaxPLcYVRG485bVsF5xagqyt+5FnV2bEtQdlgMmEnnXI4NVlJaq/ajdMU4j8yhnYjipOu1uR5mHDQGX01RQqbNr+tTb2bQ==
```
之前存储过证书

`ssh`登录`mysql`
---
**[回目录](#目录)**

```bash
[root@greately ~]# ssh admin@172.17.0.2 
[admin@0098f963ae25 ~]$ 
```

`mysql`添加授权用户
---
**[回目录](#目录)**

```bash
[admin@0098f963ae25 ~]$ mysql -uroot -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.38-log Source distribution

Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> grant all on *.* to 'jason'@'%' identified by '123123';
Query OK, 0 rows affected (0.01 sec)

mysql> exit
Bye
```

V.`lemp`测试
===
**[回目录](#目录)**


`nginx`测试
---
**[回目录](#目录)**

![](http://i.imgur.com/ITUHylV.jpg)

`php`测试
---
**[回目录](#目录)**

![](http://i.imgur.com/EF6LBdi.jpg)

`php`连接`MySQL`测试
---
**[回目录](#目录)**
![](http://i.imgur.com/H0vDIHK.jpg)



**[回目录](#目录)**


---