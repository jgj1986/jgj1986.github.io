---
title: lua-ngx操作mysql
date: 2014-11-28 11:12:28 +0800
tags: web
- a
- b
---

* toc 
{:toc}

> 本文是参考 [openresty对mysql的操作文档](https://github.com/openresty/lua-resty-mysql){:target=:"_blank"}，对mysql做添加、查询操作，因业务的需要，使用mysql的事务操作，并做测试，可以很好的支持。

###mysql查询操作

clone下来lua-resty-mysql的仓库 `https://github.com/openresty/lua-resty-mysql.git`，在nginx配置中lua路径中包含mysql.lua；或者将mysql.lua放在统一的路径下。可参考文冠的博客[nginx与lua安装教程](http://blog.kissdata.com/2014/11/14/nginx-lua-install.html){:target="_blank"}。
在实际操作时，发现很多的查询都具有相同的特性，于是封装了一个函数，放在nginx的初始化文件中，如下：

    function fetch_db_data(sql, limit)
    local mysql = require "mysql"
    local db, err = mysql:new()
    if not db then
        monitor(id, "ERROR", "FAILED_INIT_MYSQL", "")  -- 监控函数，用于告警
        ngx.exit("700")
    end
    db:set_timeout(1000) -- 1 sec
    local ok, err, errno, sqlstate = db:connect {
        host = "127.0.0.1", port = 3306,
        database = "mjyun", user = "root",
        password = "xxxxx",
        max_packet_size = 1024 * 1024 }

    if not ok then
        monitor(id, "ERROR", "FAILED_CONNECT_MYSQL", "")
        ngx.exit("701")
    end

    --fetch data--
    if 0 == limit or "" == limit then
        data, err, errno, sqlstate = db:query(sql)
    else
        data, err, errno, sqlstate = db:query(sql, limit)
    end

    local ok, err_1 = db:set_keepalive(10000, 100)
    if not ok then
        ngx.log(ngx.ERR, "failed to set keepalive: " )
    end
    return data, err, errno, sqlstate
    end
    
当需要查询时，拼接出sql语句直接调用该函数，如：

    version_sql = "select version from app_kv_ver where appid = \'" .. id .."\'"
    version, err, errno, sqlstate = fetch_db_data(version_sql, 1)

注意，如果id直接是接口传递的参数，这样就有sql注入的危险，需要对id做验证或其他防范措施。关于sql注入攻击详情，可参考前面的博客 [网络攻击与防范](http://blog.woshifengzi.com/2014/11/11/%E7%BD%91%E7%BB%9C%E6%94%BB%E5%87%BB%E4%B8%8E%E9%98%B2%E8%8C%83.html){:target="_blank"}。

因为[github](https://github.com/openresty/lua-resty-mysql){:target=:"_blank"}上已经有比较详细的介绍，此过程中用户的函数或方法就不在做过多的介绍。


###mysql的事务操作

项目中有这样的需求：用户请求更新自己应用的某个key值，且更新这个应用的版本号，这个过程要保证更新的原子性。如此计划采用[mysql的事务操作](http://dev.mysql.com/doc/refman/5.0/en/commit.html){:target="_blank"}。

    START TRANSACTION [WITH CONSISTENT SNAPSHOT]
    BEGIN [WORK]
    COMMIT [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
    ROLLBACK [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
    SET autocommit = {0 | 1}
    
在lua-resty-mysql的例子有，`db.query()`可以一次执行多条sql语句，于是做了如下的测试：

    transaction_sql = "START TRANSACTION;"
    insert_kv_sql = "insert into app_kv (appid, app_key, app_val) values (\'mj_test\', \'key1\', \'value1\') ON duplicate KEY UPDATE app_val = \'value2\';"
    insert_ver_sql = "insert into app_kv_ver(appid, version) values(\'mj_test\', 0) ON duplicate KEY UPDATE version = version + 1;"
    commit_sql = "COMMIT;"
    
    kv_data, err, errno, sqlstate = fetch_db_data(transaction_sql .. insert_kv_sql .. insert_ver_sql .. commit_sql, 4)

如果将 `insert_kv_sql` 或 `insert_ver_sql` 故意改错，则整个过程都不成功，可以达到目的。 :)


###编码问题

如果mysql中存有汉字，查询时会有乱码，操作网上的讨论 [issues-20](https://github.com/openresty/lua-resty-mysql/issues/20){:target="_blank"}，将mysql服务端的编码改为utf-8。

    $ cat mysql.conf
    [mysqld]
    skip-character-set-client-handshake
    collation-server=utf8_unicode_ci
    character-set-server=utf8

        
