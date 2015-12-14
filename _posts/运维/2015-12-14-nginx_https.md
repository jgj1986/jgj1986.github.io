---
title:  nginx配置htpps服务
date: 2015-12-14 12:12:28 +0800
tags: 运维
- a
- b
---

* toc
{:toc}

> 因为ios9要求使用https，而现在的服务又需要http的支持，需要对nginx中配置使用https需要有些更深入的了解。nginx官网上 [nginx配置htpps服务](http://nginx.org/en/docs/http/configuring_https_servers.html){:taget="_blank"}，说的比较透彻，此文做简单的意译。

## 基本配置

要使用https服务，那服务端比较要有相应的配置：监听的socket上支持ssl，指定服务证书和私钥。https使用的443端口，如果我们用443端口作为入口并设置监听，则在浏览器https后的地址栏就看不到443端口。

    server {
        listen              443 ssl;
        server_name         www.example.com;
        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        ...
    }

服务端证书 `ssl_certificate` 是公开的，会发送到连接服务端的各个客户端；私钥 `ssl_certificate_key`是保密的，访问权限受限（nginx的主进程可读）。上面还用的了 `ssl_protocals` 和 `ssl_ciphers` ，这个不是必须的。

## https服务端优化

ssl会消耗CPU资源，而CPU使用最重的是ssl的握手过程，有两个方法来减少这些握手过程：1，允许连接的keepalive，使用一个连接回复过个相应；2，在并行和随后的连接中复用ssl的会话参数(session parameters)，从而避免ssl的再握手。我们可以将这些默认的配置调整达到优化目的。

    worker_processes auto;

    http {
       ssl_session_cache   shared:SSL:10m; # 4000 -> 10M
        ssl_session_timeout 10m;           # 5 minutes -> 10m

        server {
            listen              443 ssl;
            server_name         www.example.com;
            keepalive_timeout   70;        # use this

            ssl_certificate     www.example.com.crt;
        ...

## ssl证书链

你可能会遇到一些浏览器对证书没有问题，而一些浏览器会提示证书可能存在风险，即使证书是由知名的证书机构签发的。这是因为签发的多个服务证书(server certificate)使用了一个中间证书(intermediate certificate)，而这中间证书没有在当前随受信任证书一起提供到浏览器中。在这种情况下，签发机构提供了一个大包的链路证书(chained certificates)用来合到单一的服务证书中。

    $ cat www.example.com.crt bundle.crt > www.example.com.chained.crt

在服务配置中就使用这个新的服务证书`www.example.com.chained.crt`。注意，服务证书要在前面，否则就会出现这样的错误信息

    SSL_CTX_use_PrivateKey_file(" ... /www.example.com.key") failed
       (SSL: error:0B080074:x509 certificate routines:
        X509_check_private_key:key values mismatch)

原文此处还有具体解析过程的说明，本文忽略。

## HTTP/HTTPS 服务

也可以配置一个服务支持HTTP 和 HTTPS 请求：

    server {
        listen              80;
        listen              443 ssl;
        server_name         www.example.com;

        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ...
    }

:-) 这个就是在用户系统中想要的。

## 基于名字的HTTPS服务

在单个ip上实现两个或多个https服务时，就会有个常见问题。假设我们如下配置

    server {
        listen          443 ssl;
        server_name     www.example.com;
        ssl_certificate www.example.com.crt;
        ...
    }

server {
    listen          443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    ...
}

受ssl协议的影响，会先建立ssl连接，然后在有http的请求，所以在使用https请求时，不知道使用哪个证书，只好使用默认的服务证书了。一个古老而最鲁棒的解决办法就是安置到不同的ip地址上

    server {
        listen          192.168.1.1:443 ssl;
        server_name     www.example.com;
        ssl_certificate www.example.com.crt;
        ...
    }

    server {
        listen          192.168.1.2:443 ssl;
        server_name     www.example.org;
        ssl_certificate www.example.org.crt;
        ...
    }

## 一个证书多个域名

可以支持一个IP多个域名的https，但是都有些缺点。还有一个方式，就是使用同一份证书，支持多个域名

    ssl_certificate     common.crt;
    ssl_certificate_key common.key;

    server {
        listen          443 ssl;
        server_name     www.example.com;
        ...
    }

    server {
        listen          443 ssl;
        server_name     www.example.org;
        ...
    }

## 其他

本文忽略，请查看原文。

