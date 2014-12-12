---
title:  go操作mysql、memcache、mongodb
date: 2014-12-12 21:12:28 +0800
tags: go
- a
- b
---

* toc 
{:toc}


> 发现go提供的操作msyql、memcache、mongodb的文档没有lua-ngx的好读，即实例不是很明确，此文列出自己操作时的实例，可让快速入手，但是优化以及错误处理等还有很多工作。


###操作mysql

[go-sql-driver/mysql Example]https://github.com/go-sql-driver/mysql/wiki/Examples 上提供了用go操作mysql的两个例子。
例子中关于查询，一个是一次只取一个结果，一个是一次去多个结果，而且使用了prepare的方法，防止sql注入攻击。  
此文中是参考上面例子，做的测试。

只取一个数据的例子

    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql"
    )

    func main() {
        db, err := sql.Open("mysql", "user:passwd@tcp(host:port)/db_name")
        if err != nil {
            panic(err.Error())
        }
        defer db.Close()

        stmtOut, err := db.Prepare("SELECT uid FROM mj_table limit 1")
        if err != nil {
            panic(err.Error())
        }
        var uid string
        err = stmtOut.QueryRow().Scan(&uid)
        if err != nil {
            Log.Info("err to Scan: %s", err)
            return
        }
        Log.Info("uid: %s", uid)
    }


取多个数据的例子

    ...
    rows, err := db.Query("select appid from mj_table limit 1")
    if err != nil {
        Log.Info("No Err in select msyql: %s", err)
        return
    }   
    // Get column names
    columns, err := rows.Columns()
    if err != nil {
        panic(err.Error()) 
    }   

    // Make a slice for the values
    values := make([]sql.RawBytes, len(columns))
    scanArgs := make([]interface{}, len(values))   // 必须要有此类型
    for i := range values {
        scanArgs[i] = &values[i]
    }

    // Fetch rows
    for rows.Next() {
        // get RawBytes from data
        err = rows.Scan(scanArgs...)
        if err != nil {
            panic(err.Error()) 
        }
        var value string
        for i, col := range values {
            if col == nil {
                value = "NULL"
            } else {
               value = string(col)
            }
            Log.Info("name: %s, value: %s ", columns[i], value)
        }
        Log.Info("-----------------------------------")
    }
    ...



###操作mongodb
[mgo](https://gopkg.in/mgo.v2){:target="_blank"}是go语言操作MongoDb的驱动，因为接口比较丰富，[官方文档](http://godoc.org/gopkg.in/mgo.v2){:target="_blank"}看着也就比较长。

下面代码提供了建立连接、插入数据、查询、删除的操作

    import (
    "time"
    "gopkg.in/mgo.v2"   // mgo的包
    "gopkg.in/mgo.v2/bson"
    )

    func main() {
        var (
            err error
        )
        //建立连接
        mongoURL := "192.168.1.140,192.168.1.141:20001,192.168.1.141:20002"  //可在配置文件中设置
        maxWait := time.Duration(10 * time.Second)
        sess, err = mgo.DialWithTimeout(mongoURL, maxWait)
        if err != nil {
            Log.Error("mgo.Dial err:%s", err)
            panic(err)
        }   
        sess.SetMode(mgo.Monotonic, true)
        defer sess.Close()
        
        //插入数据      
        //...构造要插入数据
        db_name := "db"
        table_name := "table"  // 即mongodb中的Collection
        dataBytes := []byte("{\"name\":\"jgj\"}")
        var intf map[string]interface{}
        err = json.Unmarshal(dataBytes, &intf)
            
        //...执行入库操作
        // new_sess := sess.Copy()  // 可以复制一份回话，用新回话做操作
        col := sess.DB(appid).C(classname)
        err = col.Insert(intf)
        if err != nil {
             Log.Error("db POST, db insert error, appid:%s, class:%s, data:%s, err:%s", appid, classname, string(dataBytes), err)
             return
        }
        
        // 获取数据
        var objectid interface{}
        objectid = string("5486b4e08bb3c2659c000001")   
        err = col.FindId(objectid).One(&intf)
        this.Data["json"] = intf
        // this.ServeJson() // 显示
        
        // 更新某个对象
        err = col.Update(bson.M{"_id": objectid, "updatedAt": oldUpdatedAt}, intf)

        // 删除某个对象
        err = col.RemoveId(objectid)
        
        // 查询
        selectParam := "{\"name\":true, \"age\":true}"
        var selectIntf map[string]bool
        err = json.Unmarshal([]byte(selectParam), &selectIntf)
        whereParam := "{\"or\":[{\"name\":\"jgj\"},{\"age\":\"29\"}]}"
        var whereIntf := map[string]interface{}{}
        err = json.Unmarshal([]byte(whereParam), &whereIntf)
        //...json to bson. 代码未提供
        ConvertBSON(whereIntf)
        sortParam = "name,-age"
        offset = 1
        limit = 10

        //...TODO 此处要有错误的判断
        query := col.Find(whereIntf)
        query = query.Sort(sortParam)
        query = query.Select(selectIntf)
        query = query.Skip(offset)
        query = query.Limit(limit)
        
        iter := query.Iter()
        var result []bson.M = make([]bson.M, 0)
        intf := bson.M{}
        for iter.Next(&intf) {
            result = append(result, intf)
            intf = bson.M{}
        }   
        iter.Close()    
        this.Data["json"] = result
        this.ServeJson()
    }


###操作memcache
[GoDoc中memcache的说明](https://godoc.org/github.com/bradfitz/gomemcache/memcache){:target="_blank"}提供了操作的一些函数，没有给出实际的例子。
下面是测试的小例子

    con := memcache.New("ip:host")
    if con == nil {
        Log.Info("Failed to connect Memcache")
    }

    item := &memcache.Item{Key:"jgj",Value:[]byte("test_value"),Expiration:0 }
    err := con.Set(item)
    if err != nil {
        Log.Info("failed to set item: %s", err)
    }

    item1, err := con.Get("jgj")
    if err != nil {
        Log.Info("Failed to get item %s ", err)
        return
    }
    Log.Info("get value: %s", item1.Value)


