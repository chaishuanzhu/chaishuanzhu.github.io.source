---
title: MAMP环境PHP添加pcntl扩展
date: YYYY-MM-DD hh:mm:ss
tags: 标签                    #多标签时以[tag1,tag2]格式填写
categories: PHP            #多类别时以[category1,category2]格式填写
---

## MAMP环境PHP添加pcntl扩展

前言：
pcntl 介绍
pcntl 扩展可以支持 PHP 的多线程操作。（非Unix类系统不支持此模块）
phpize 介绍
phpize 可以用来给 PHP 动态的添加扩展。比如编译 PHP 时忘记添加某扩展，后来想添加该扩展，即可以使用 phpize 动态的添加该扩展。
本文将介绍如何使用 phpize 给 Mac下的集成环境 MAMP 添加 PHP 的 pcntl 扩展。类 Unix 的环境都可以使用本方法安装，注意灵活变通。

## 修改bash_profile文件切换Mac默认PHP版本为MAMP
把mac上的默认php修改为MAMP等扩展环境中的php 版本，也就是不使用 mac 自带的。步骤如下：
打开terminal。输入 
``` bash 
$ locate ~/.bash_profile 
```
找到我们需要编辑的文件的路径。
cd 到该目录下执行
``` bash 
$ sudo vim ~/.bash_profile
```
这个文件也有可能没有，如果没有就新建一个。
在vim中打开或新建好该文件后在文件末尾添加
``` bash
$ export PATH="/Applications/MAMP/bin/php/php7.0.8/bin:$PATH"
```
注：“=”号之后，“：”号之前的路径就是你要修改成的php的路径，我这里选取的是MAMP中自带的7.0.8版本。
保存后运行： 
``` bash
$ source ~/.bash_profile
```
可能会报错，不用管。
就大功告成了，接着我们验证一下，另开terminal，输入
``` bash 
$ which PHP
```
有没有发现默认php的路径已经修改好了呢。接下来就可以正常使用了，如果还不行，重启电脑就可以了。

安装：
下面演示的是给 MAMP的PHP 7.0.8版本添加 pcntl扩展。
下载和本地 PHP 版本对应的源码包，地址为：
http://www.php.net/releases/
然后按照如下步骤进行编译：
## 解压源码包并初始化目录
``` bash
$ tar -xzvf php-7.0.8.tar.gz
$ mv php-7.0.8 php
$ mkdir -p /Applications/MAMP/bin/php/php7.0.8/include
$ mv php /Applications/MAMP/bin/php/php7.0.8/include
```
## 检测系统配置
``` bash
$ cd /Applications/MAMP/bin/php/php7.0.8/include/php
$ ./configure
```
## 添加一些标志来告诉系统怎样编译。MAMP PHP已经建成这样，如果不这样做，编译的共享对象将无法工作。
``` bash
$ MACOSX_DEPLOYMENT_TARGET=10.10
$ CFLAGS="-arch i386 -arch x86_64 -g -Os -pipe -no-cpp-precomp"$ CCFLAGS="-arch i386 -arch x86_64 -g -Os -pipe"
$ CXXFLAGS="-arch i386 -arch x86_64 -g -Os -pipe"
$ LDFLAGS="-arch i386 -arch x86_64 -bind_at_load"
$ export CFLAGS CXXFLAGS LDFLAGS CCFLAGS MACOSX_DEPLOYMENT_TARGET
```
## 编译 pcntl.so 文件
``` bash
$ cd ext/pcntl
$ phpize
$ ./configure
$ make
```
## 将编译出来的扩展文件pcntl.so 移动到php的扩展目录
``` bash
$ cp modules/pcntl.so /Applications/MAMP/bin/php/php7.0.8/lib/php/extensions/no-debug-non-zts-20151012/   
```
## 向php.ini 文件中添加该扩展
``` bash
$ echo "extension=pcntl.so" >> /Applications/MAMP/bin/php/php7.0.8/conf/php.ini
```
## pcntl现在应该已经启用，要检查是否安装成功，只需运行：
``` bash
$ /Applications/MAMP/bin/php/php7.0.8/bin/php --ri pcntl
pcntl
pcntl support => enabled
```
如出现以上信息，则说明该扩展已安装成功。


## PHP动态编译出现Cannot find autoconf的解决方法 
``` bash
wget http://ftp.gnu.org/gnu/m4/m4-1.4.9.tar.gz
tar -zvxf m4-1.4.9.tar.gz
cd m4-1.4.9/
./configure && make && make install
cd ../
wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.62.tar.gz
tar -zvxf autoconf-2.62.tar.gz
cd autoconf-2.62/
./configure && make && make install
```
参考链接：
http://blog.csdn.net/lipeiran1987/article/details/54942896
http://www.jianshu.com/p/ec88a61a0fa8
http://www.cnblogs.com/wt645631686/p/6868476.html
