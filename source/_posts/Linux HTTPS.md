---
title: Linux HTTPS
date: YYYY-MM-DD hh:mm:ss
tags: 标签                    #多标签时以[tag1,tag2]格式填写
categories: 服务器             #多类别时以[category1,category2]格式填写
---

## Linux HTTPS

阿里云证书，结合这篇文章：http://blog.csdn.net/wlzx120/article/details/52597338
主要步骤：vim /etc/httpd/conf.d/ssl.conf     
阿里官方配置：

```
# 添加 SSL 协议支持协议，去掉不安全的协议
SSLProtocol TLSv1 TLSv1.1 TLSv1.2
# 修改加密套件如下
SSLCipherSuite ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4
# 证书公钥配置
SSLCertificateFile cert/public.pem
# 证书私钥配置
SSLCertificateKeyFile cert/214052098260625.key
# 证书链配置，如果该属性开头有 '#'字符，请删除掉
SSLCertificateChainFile cert/chain.pem

```

修改配置：

```

# 添加 SSL 协议支持协议，去掉不安全的协议
SSLProtocol TLSv1 TLSv1.1 TLSv1.2
# 修改加密套件如下
SSLCipherSuite ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4
# 证书公钥配置
SSLCertificateFile cert/214052098260625.pem
# 证书私钥配置 
SSLCertificateKeyFile cert/214052098260625.key 
# 证书链配置，如果该属性开头有 '#'字符，请删除掉 
SSLCertificateChainFile cert/chain.pem
# 根证书配置，
SSLCACertificateFile cert/public.pem

```
