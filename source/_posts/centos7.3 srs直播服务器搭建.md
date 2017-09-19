---
title: centos7.3 srs直播服务器搭建
date: YYYY-MM-DD hh:mm:ss
tags: 标签                    #多标签时以[tag1,tag2]格式填写
categories: 服务器             #多类别时以[category1,category2]格式填写
---

## centos7.3 srs直播服务器搭建

[官方文档](https://github.com/ossrs/srs/tree/2.0release)

### 下载源码到服务器上
```
# wget srs-2.0.tar.gz https://codeload.github.com/ossrs/srs/tar.gz/v2.0-r2
```
### 解压
```
# tar -xzvf srs2.0.tar.gz
```

### 编译
```
./configure && make
```
### 安装
```
make install
```
### 更改配置
#### 先备份一下默认配置
```
# mv srs.conf srs.conf.old
```
#### 新建配置文件
```
# vi srs.conf
```
#### 复制下面内容并保存
```
# main config for srs.
# @see full.conf for detail config.

listen              1935;
max_connections     1000;
srs_log_tank        file;
srs_log_file        ./objs/srs.log;
http_api {
    enabled         on;
    listen          1985;
}
http_server {
    enabled         on;
    listen          8080;
    dir             ./objs/nginx/html;
}
stats {
    network         0;
    disk            sda sdb xvda xvdb;
}
vhost __defaultVhost__ {
    #最小延迟打开，默认是打开的，该选项打开的时候，mr默认关闭。
    min_latency     on;
#Merged-Read，针对RTMP协议，为了提高性能，SRS对于上行的read使用merged-read，
即SRS在读写时一次读取N毫秒的数据
    mr {
        enabled     off;
        #默认350ms，范围[300-2000]
        #latency     350;
    }
    #Merged-Write,SRS永远使用Merged-Write，即一次发送N毫秒的包给客户端。这个算法
可以将RTMP下行的效率提升5倍左右,范围[350-1800]
    mw_latency      100;
    #enabled         on;
    #https://github.com/simple-rtmp-server/srs/wiki/v2_CN_LowLatency#gop-cache
    gop_cache       off;
    #配置直播队列的长度，服务器会将数据放在直播队列中，如果超过这个长度就清空到>最后一个I帧
    #https://github.com/simple-rtmp-server/srs/wiki/v2_CN_LowLatency#%E7%B4%AF%E7%A7%AF%E5%BB%B6%E8%BF%9F
    queue_length    10;
    #http_flv配置

    http_remux {
            enabled     on;
            mount [vhost]/[app]/[stream].flv;
            hstrs       on;
    }

    hls {
        enabled         on;
        hls_path        ./objs/nginx/html;
        hls_fragment    10;
        hls_window      60;
    }

    http_hooks {
        enabled         on;
        on_connect      http://www.chaisz.xyz/srstest/clients.php;
        on_close        http://www.chaisz.xyz/srstest/clients.php;
        on_publish      http://www.chaisz.xyz/srstest/streams.php;
        on_unpublish    http://www.chaisz.xyz/srstest/streams.php;
        on_play         http://www.chaisz.xyz/srstest/sessions.php;
        on_stop         http://www.chaisz.xyz/srstest/sessions.php;
    }
}
```
### 启动服务器
```
# ./objs/srs -c conf/srs.conf
```


### 其他知识
#### 查看端口号
```
# netstat -anp
```
#### 查看占用某端口的进程号
```
# netstat -anp | grep 1935
```
#### 关闭对应的应用程序
```
kill -9 22404
```
