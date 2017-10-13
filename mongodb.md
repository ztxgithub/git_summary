# mongodb 基本概念

``` shell

    1.慢查询: 运行比较慢的SQL语句
    2.mongodb System Collections(系统集合), <database>.system.profile保存有慢查询操作,同时该集合是固定大小的,
      文档超过了原有的大小会覆盖旧的的文档.
    
    
			
```

## mongodb 基本常识

``` shell

    1.所有的mongod和mongo实例(instances)中,对于每一个incoming的连接都会创建一个文件描述符和线程
    
    
			
```

# mongod使用方法

- systemctl restart mongod.service 启动失败

``` shell

    原因：1./var/lib/mongo及子目录文件所有者都应该使mongod用户
    （chown mongod:mongod /var/lib/mongo && chown -R mongod:mongod /var/lib/mongo/）
    
         2./var/log/mongodb及子目录文件所有者都应该使mongod用户
    （chown mongod:mongod /var/log/mongodb && chown -R mongod:mongod /var/log/mongodb）
    
         3./tmp/mongodb-27017.sock文件所有者都应该使mongod用户
     (sudo chown mongod:mongod /tmp/mongodb-27017.sock)
			
```

# mongodb 命令使用

- 切换到对应的数据库(database)

``` shell
    通过 mongostat 和 mongotop 可以看出 哪个db(数据库)的哪个集合(collection)进慢查询.


    1. 进入mongo shell
        $ mongo
       
    2. 输入help命令
        > help

        > use scloud-test (切换到对应的数据库,这个时候还要用户名和密码,才能对数据库进行操作)
        
        > db.auth("test", "YYtd_test")
        
        查看当前的数据库的mongo的查询情况
        > db.currentOP()
    
			
```

## mongodb 优化

- 慢查询日志

``` shell
    
    1.Profiling级别说明
        0：关闭，不收集任何数据。
        1：收集慢查询数据，默认是100毫秒。
        2：收集所有数据
        
     前提条件:进入到特定的数据库中 use dbname
     
    2.开启Profiling和设置
        (1) 查看状态：级别和时间
            > db.getProfilingStatus()
            { "was" : 0, "slowms" : 100 }  关闭慢查询,超过100ms记录
        
        (2) 查看级别:
            > db.getProfilingLevel()
                0     代表关闭
                
        (3) 设置级别和时间
            > db.setProfilingLevel(1,200)
              { "was" : 0, "slowms" : 100, "ok" : 1 }
              
        不通过mongo shell:
        mongod --profile=1 --slowms=15
        
    3.关闭Profiling
        > db.setProfilingLevel(0)
        
    4. 修改“慢查询日志”的大小
        #关闭Profiling
            > db.setProfilingLevel(0)
                { "was" : 0, "slowms" : 200, "ok" : 1 }
                
        #删除system.profile集合
            > db.system.profile.drop()
               true
        #创建一个新的system.profile集合
            > db.createCollection( "system.profile", { capped: true, size:4000000 } )
                 { "ok" : 1 } 创建4000000bytes
                 
        #重新开启Profiling
            > db.setProfilingLevel(1)
                { "was" : 0, "slowms" : 200, "ok" : 1 }
                
    5.分析查询结果
        > db.system.profile.find().pretty()
        
        {
            "op" : "query",    #操作类型，有insert、query、update、remove、getmore、command   
            "ns" : "mc.user",  #操作的集合
            "query" : {        #查询语句
                "mp_id" : 5,
                "is_fans" : 1,
                "latestTime" : {
                    "$ne" : 0
                },
                "latestMsgId" : {
                    "$gt" : 0
                },
                "$where" : "new Date(this.latestNormalTime)>new Date(this.replyTime)"
            },
            "cursorid" : NumberLong("1475423943124458998"),
            "ntoreturn" : 0,   #返回的记录数。例如，profile命令将返回一个文档（一个结果文件），因此ntoreturn值将为1。limit(5)命令将返回五个文件，因此ntoreturn值是5。如果ntoreturn值为0，则该命令没有指定一些文件返回，因为会是这样一个简单的find()命令没有指定的限制。
            "ntoskip" : 0,     #skip()方法指定的跳跃数
            "nscanned" : 304,  #扫描数量
            "keyUpdates" : 0,  #索引更新的数量，改变一个索引键带有一个小的性能开销，因为数据库必须删除旧的key，并插入一个新的key到B-树索引
            "numYield" : 0,    #该查询为其他查询让出锁的次数
            "lockStats" : {    #锁信息，R：全局读锁；W：全局写锁；r：特定数据库的读锁；w：特定数据库的写锁
                "timeLockedMicros" : {     #锁
                    "r" : NumberLong(19467),
                    "w" : NumberLong(0)
                },
                "timeAcquiringMicros" : {  #锁等待
                    "r" : NumberLong(7),
                    "w" : NumberLong(9)
                }
            },
            "nreturned" : 101,        #返回的数量
            "responseLength" : 74659, #响应字节长度
            "millis" : 19,            #消耗的时间（毫秒）
            "ts" : ISODate("2014-02-25T02:13:54.899Z"), #语句执行的时间
            "client" : "127.0.0.1",   #链接ip或则主机
            "allUsers" : [ ],     
            "user" : ""               #用户
        }
        
    6.日常使用的查询
    
        #返回最近的10条记录
        db.system.profile.find().limit(10).sort({ ts : -1 }).pretty()
        
        #返回所有的操作，除command类型的
        db.system.profile.find( { op: { $ne : 'command' } } ).pretty()
        
        #返回特定集合
        db.system.profile.find( { ns : 'mydb.test' } ).pretty()
        
        #放回某个特定集合的大小
        db.system.profile.find( { ns : 'mydb.test' } ).count()
        
        #返回大于5毫秒慢的操作
        db.system.profile.find( { millis : { $gt : 5 } } ).pretty()
        
        #从一个特定的时间范围内返回信息
        db.system.profile.find(
                               {
                                ts : {
                                      $gt : new ISODate("2012-12-09T03:00:00Z") ,
                                      $lt : new ISODate("2012-12-09T03:40:00Z")
                                     }
                               }
                              ).pretty()
        
        #特定时间，限制用户，按照消耗时间排序
        db.system.profile.find(
                               {
                                 ts : {
                                       $gt : new ISODate("2011-07-12T03:00:00Z") ,
                                       $lt : new ISODate("2011-07-12T03:40:00Z")
                                      }
                               },
                               { user : 0 }
                              ).sort( { millis : -1 } )
                              
   

        
    
			
```




# 要从阿里云数据库下载恢复到线下的数据库

``` shell

    > cat hins2132121_data_20170906160327.ar |  
    mongorestore -h 192.168.0.6 --port 27017 -d  scloud-product -u product -p YYtd_product  
    --drop --gzip --archive -vvvv --stopOnError
    	
```

# 总结

- mongodb 函数调用成功不一定数据成功写到数据库中,一段时间内调用大量的数据库操作,会导致数据库崩溃.

- mongodb 内存cache问题

``` shell

    由于mongodb数据库的机制问题,从磁盘中查找将结果cache到内存中,如果查询的结果不用,则又会从磁盘中读数据,
    这样会导致IO压力大
    	
```

# c++ driver

## update函数结果

```c++
     auto result = update_one_document(fsu_collection, condition, update);
        if(!result) {
            log_e("update fsu_lgoinstate[%s][%s] fail", fsu_collection.c_str(), fsuid.c_str());
            return FUNC_FAIL;
        } else {
            log_d("update fsu_lgoinstate[%s][%s] success", fsu_collection.c_str(), fsuid.c_str());
        }
		
	update_one_document 不成功 result也不会为空
```