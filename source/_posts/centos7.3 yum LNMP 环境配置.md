---
title: centos7.3 yum LNMP 环境配置
date: YYYY-MM-DD hh:mm:ss
tags: 标签                    #多标签时以[tag1,tag2]格式填写
categories: 服务器             #多类别时以[category1,category2]格式填写
---

## centos7.3 yum LNMP 环境配置

服务器配置太低，装了gitlab后挂了，重新配置一下。
### **挂载数据盘**
```
df -h
```
![](http://upload-images.jianshu.io/upload_images/1386124-c6f18782a0f2affa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
只有系统盘了，挂载上原来的数据盘
```
fdisk -l
```
![](http://upload-images.jianshu.io/upload_images/1386124-42664cbc0db451fa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看到数据盘了/dev/vdb1
挂载上这个数据盘，
```
mkdir /data
mount /dev/vdb1 /data
```
![](http://upload-images.jianshu.io/upload_images/1386124-388f2dcf8b0bc7aa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后写入分区：
```
echo '/dev/vdb1 /data ext4 defaults 0 0' >> /etc/fstab
```
不写入分区表，重启后又要挂载的。
怎么知道分区类型是ext4，用这个命令：
```
df -hT
```
 
好的成功了！
 
### **安装nginx**
**首先更新系统软件**
```
# yum update
```
### 安装nginx
#### 安装nginx源
```
# yum localinstall http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```
#### 安装nginx
```
# yum install nginx
```
#### 启动nginx
```
# service nginx start

Redirecting to /bin/systemctl start  nginx.service
```
#### 访问http://你的ip/
如果成功安装会出来nginx默认的欢迎界面
![](http://upload-images.jianshu.io/upload_images/1386124-671c67a1062f2adb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
### **安装MySQL5.7.***
#### 安装mysql源
```
# yum localinstall http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
```
#### 安装mysql
```
# yum install mysql-community-server
```
#### 安装mysql的开发包，以后会有用
```
# yum install mysql-community-devel
```
#### 启动mysql
```
# service mysqld start

Redirecting to /bin/systemctl start  mysqld.service
```
#### 查看mysql启动状态
```
# service mysqld status
```
出现pid
证明启动成功
#### 获取mysql默认生成的密码
```
# grep 'temporary password' /var/log/mysqld.log
```
![](http://upload-images.jianshu.io/upload_images/1386124-e0dd162ec8014539.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 选中的就是密码。
#### 换成自己的密码
```
# mysql -uroot -p
```
Enter password:输入上页的密码，进入mysql
#### 更换密码
1
```
mysql>  ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPasdfs4!';
```
这个密码一定要足够复杂，不然会不让你改，提示密码不合法;
#### 退出mysql并试用下新密码
```
mysql> quit;mysql -uroot -p
```
确认密码正确
### **编译安装php7.0.0**
#### 下载php7源码包
```
# cd /root & wget -O php7.tar.gz http://cn2.php.net/get/php-7.0.1.tar.gz/from/this/mirror
```
#### 解压源码包
```
# tar -xvf php7.tar.gz
```
#### 进入目录
```
# cd php-7.0.1
```
#### **安装php依赖包**　
```
# yum install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
```
#### 编译配置，这里如果上一步的某些依赖包没有安装好，就会遇到很多configure error，我们一一解决，安装上相关软件开发包就可以

```
# ./configure \
--prefix=/usr/local/php \
--with-config-file-path=/etc \
--enable-fpm \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared \
--enable-soap \
--with-libxml-dir \
--with-xmlrpc \
--with-openssl \
--with-mcrypt \
--with-mhash \
--with-pcre-regex \
--with-sqlite3 \
--with-zlib \
--enable-bcmath \
--with-iconv \
--with-bz2 \
--enable-calendar \
--with-curl \
--with-cdb \
--enable-dom \
--enable-exif \
--enable-fileinfo \
--enable-filter \
--with-pcre-dir \
--enable-ftp \
--with-gd \
--with-openssl-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib-dir \
--with-freetype-dir \
--enable-gd-native-ttf \
--enable-gd-jis-conv \
--with-gettext \
--with-gmp \
--with-mhash \
--enable-json \
--enable-mbstring \
--enable-mbregex \
--enable-mbregex-backtrack \
--with-libmbfl \--with-onig \
--enable-pdo \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-zlib-dir \
--with-pdo-sqlite \
--with-readline \
--enable-session \
--enable-shmop \
--enable-simplexml \
--enable-sockets \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-wddx \
--with-libxml-dir \
--with-xsl \
--enable-zip \
--enable-mysqlnd-compression-support \
--with-pear \
--enable-opcache
```

configure error:
1.configure: error: xml2-config not found. Please check your libxml2 installation.
解决：
```
# yum install libxml2 libxml2-devel
```
 
2.configure: error: Cannot find OpenSSL's <evp.h>
解决：
```
# yum install openssl openssl-devel
```
 
3.configure: error: Please reinstall the BZip2 distribution
解决：
```
# yum install bzip2 bzip2-devel
```
 
4.configure: error: Please reinstall the libcurl distribution - easy.h should be in <curl-dir>/include/curl/
解决：
```
# yum install libcurl libcurl-devel
```
 
5.If configure fails try --with-webp-dir=<DIR> configure: error: jpeglib.h not found.
 
解决：
```
# yum install libjpeg libjpeg-devel
```
6.If configure fails try --with-webp-dir=<DIR>
checking for jpeg_read_header in -ljpeg... yes
configure: error: png.h not found.
解决：
```
# yum install libpng libpng-devel
```
 
7.If configure fails try --with-webp-dir=<DIR>
checking for jpeg_read_header in -ljpeg... yes
checking for png_write_image in -lpng... yes
If configure fails try --with-xpm-dir=<DIR>
configure: error: freetype-config not found.
解决：
```
# yum install freetype freetype-devel
```
8.configure: error: Unable to locate gmp.h
解决：
```
# yum install gmp gmp-devel
```
9.configure: error: mcrypt.h not found. Please reinstall libmcrypt.
解决：
```
  # yum install libmcrypt libmcrypt-devel
```
10.configure: error: Please reinstall readline - I cannot find readline.h
解决：
```
# yum install readline readline-devel
```
11.configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution
解决：
```
# yum install libxslt libxslt-devel
```
#### 编译与安装
```
# make && make install
```
这里要make好久，要耐心一下
#### 添加 PHP 命令到环境变量
```
# vim /etc/profile
```
在末尾加入
```
PATH=$PATH:/usr/local/php/bin
export PATH
```
要使改动立即生效执行
```
# source /etc/profile
```
查看环境变量
```
# echo $PATH
```
查看php版本
```
# php -v
```
#### **配置php-fpm**
```
# cp php.ini-production /etc/php.ini

# cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf# cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf

# cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm

# chmod +x /etc/init.d/php-fpm
```
#### 启动php-fpm
```
# /etc/init.d/php-fpm start
```
### **配置nginx虚拟机,绑定域名**
#### 添加配置文件
```
# vim /etc/nginx/conf.d/www.chaisz.xyz.conf
```
这里可以把www.chaisz.xyz改成自己的域名

把下面的内容复制到www.chaisz.xyz.conf里
```
server{
    listen 80;
    server_name  www.chaisz.xyz;
    root /data/www/; # 该项要修改为你准备存放相关网页的路径
    location / {
        index  index.php index.html index.htm;
         #如果请求既不是一个文件，也不是一个目录，则执行一下重写规则
         if (!-e $request_filename)
         {
            #地址作为将参数rewrite到index.php上。
            rewrite ^/(.*)$ /index.php/$1;
            #若是子目录则使用下面这句，将subdir改成目录名称即可。
            #rewrite ^/subdir/(.*)$ /subdir/index.php/$1;
         }
    }
    #proxy the php scripts to php-fpm
    location ~ \.php {
            include fastcgi_params;
            ###pathinfo支持start
            #定义变量 $path_info ，用于存放pathinfo信息
            set $path_info "";
            #定义变量 $real_script_name，用于存放真实地址
            set $real_script_name $fastcgi_script_name;
           #如果地址与引号内的正则表达式匹配
            if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
                    #将文件地址赋值给变量 $real_script_name
                    set $real_script_name $1;
                    #将文件地址后的参数赋值给变量 $path_info
                    set $path_info $2;
            }
            #配置fastcgi的一些参数
            fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
            fastcgi_param SCRIPT_NAME $real_script_name;
            fastcgi_param PATH_INFO $path_info;
            ####pathinfo支持end
        fastcgi_intercept_errors on;
        fastcgi_pass   127.0.0.1:9000;
    }

    location ^~ /data/runtime {
    return 404;
    }
    location ^~ /application {
    return 404;
    }

    location ^~ /simplewind {
    return 404;
    }
}
#************如果不需要支持https，以下内容不需要***********
#************支持https需要ssl证书 
server {
    listen 443;
    server_name localhost;
    ssl on;
    root /data/www/;
    index index.html index.htm;
    ssl_certificate   /etc/nginx/ssl/214052098260625.pem;
    ssl_certificate_key  /etc/nginx/ssl/214052098260625.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        index  index.php index.html index.htm;
         #如果请求既不是一个文件，也不是一个目录，则执行一下重写规则
         if (!-e $request_filename)
         {
            #地址作为将参数rewrite到index.php上。
            rewrite ^/(.*)$ /index.php/$1;
            #若是子目录则使用下面这句，将subdir改成目录名称即可。
            #rewrite ^/subdir/(.*)$ /subdir/index.php/$1;
         }
    }
    #proxy the php scripts to php-fpm
    location ~ \.php {
            include fastcgi_params;
            ###pathinfo支持start
            #定义变量 $path_info ，用于存放pathinfo信息
            set $path_info "";
            #定义变量 $real_script_name，用于存放真实地址
            set $real_script_name $fastcgi_script_name;
            #如果地址与引号内的正则表达式匹配
            if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
                    #将文件地址赋值给变量 $real_script_name
                    set $real_script_name $1;
                    #将文件地址后的参数赋值给变量 $path_info
                    set $path_info $2;
            }
            #配置fastcgi的一些参数
            fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
            fastcgi_param SCRIPT_NAME $real_script_name;
            fastcgi_param PATH_INFO $path_info;
            ####pathinfo支持end
        fastcgi_intercept_errors on;
        fastcgi_pass   127.0.0.1:9000;
    }

    location ^~ /data/runtime {
    return 404;
    }

    location ^~ /application {
    return 404;
    }

    location ^~ /simplewind {
    return 404;
    }
    #******************如果不需要支持https，以上内容不需要
}
```

#### 重启nginx
```
# service nginx reload
```
#### 测试脚本
```
# vim /data/www/index.php
```
把下面的代码复制到这个文件 里
```
<?php
phpinfo();
```
#### 查看访问http://www.chaisz.xyz 和 https://www.chaisz.xyz
ok!收工！


[原文链接](http://www.cnblogs.com/feng18/p/6491386.html)
