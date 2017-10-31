# mongodb 基本概念

``` shell

    1.慢查询: 运行比较慢的SQL语句
    2.mongodb System Collections(系统集合), <database>.system.profile保存有慢查询操作,同时该集合是固定大小的,
      文档超过了原有的大小会覆盖旧的的文档.
      
    3.mongodb是一个非关系性数据库,它通过最大限度的利用内存资源来提高性能,同时它的存储方式是以文档的形式,这样获取数据的方式
      就比较方便,可以通过key-value的方式来获取.在传统数据库中对数据库的操作需要提前设计好数据库表的类型和字段,对尝试插入不符合
      的数据时,传统的数据库是不会接受以确保数据的完整性.而mongodb则是动态的
      
    4.范式化设计:数据库表中只存放单一功能的数据,查询数据主要是通过关联的方式,这样可能会影响查询的性能,但数据库的修改和删除则
                比较方便.
                
      反范式化设计:尽可能得将一个数据所关联的所有信息都保存在一张表中,这样查询的性能较好,但修改数据就比较麻烦.
      
      
    
    
			
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
            "ntoreturn" : 0,   #返回的记录数。例如，profile命令将返回一个文档（一个结果文件），因此ntoreturn值将为1。
                               limit(5)命令将返回五个文件,因此ntoreturn值是5。
                               如果ntoreturn值为0,则该命令没有指定一些文件返回,
                               因为会是这样一个简单的find()命令没有指定的限制.
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

- 执行计划函数explain

``` shell
    
    > db.GuoDong_signal.find({code:"371602000100600010406001001"}).explain("executionStats")
    
    expalin的参数:
        1."queryPlanner"
        2."executionStats"
        3."allPlansExecution"
        
    explain()的返回值有：
    
    stage的分类:
        COLLSCAN : for a collection scan (对一个集合中进行所有文档扫描)
        IXSCAN  : for scanning index keys 
        FETCH : for retrieving documents (检索文档)
    
    keysExamined值:索引被查找的次数
        在stage为IXSCAN的阶段时,如果查找的条件的值在一个连续的范围内,那么keysExamined的值只等于in-bounds keys,如果查找的
        条件是多个不连续的值,那么索引执行查找程序会检测一个out-of-bounds 的索引,然后跳到下一个in-bounds key值.
        那么keysExamined的值in-bounds keys + out-of-bounds keys,
        例如 db.keys.find( { x : { $in : [ 3, 4, 50, 74, 75, 90 ] } } ).explain( "executionStats" )
            先查询3,4,再查询5,发现5不是要找的值,直接跳到50,接着检查51,不是要找的值,直接跳到74,
            这样3, 4, 50, 74, 75, 90 是in-bounds keys, 而 5,51,76,91 是out-of-bounds keys.
            所有 keysExamined = 6 + 4 = 10
    
    
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "scloud-product.GuoDong_signal",  //运行查询的指定集合名
                "indexFilterSet" : false,  //在查询中是否使用索引过滤
                "parsedQuery" : {
                        "code" : {
                                "$eq" : "371602000100600010406001001"
                        }
                },
                "winningPlan" : {  //查询优化选择的计划文档
                        "stage" : "FETCH",  //查询阶段,如果stage有子stage ,该子stage为inputStage
                        "inputStage" : {  //子过程的文档
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "code" : 1
                                },
                                "indexName" : "signalCode",
                                "isMultiKey" : false,  //本次查询是否使用了多键、复合索引
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 1,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "code" : [
                                                "[\"371602000100600010406001001\", \"371602000100600010406001001\"]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]  //被查询优化备选并被拒绝的计划数组
        },
        "executionStats" : {  //被选中执行计划和被拒绝执行计划的详细说明,对于写操作不会对数据库进行修改
                "executionSuccess" : true,  //是否成功
                "nReturned" : 1,  //符合查询条件的文档数
                "executionTimeMillis" : 12,  //计划选择和查询执行所需的总时间（毫秒数）
                "totalKeysExamined" : 1,     //扫描的索引总数
                "totalDocsExamined" : 1,     //扫描的文档总数
                "executionStages" : {      //显示执行成功细节的查询阶段树
                        "stage" : "FETCH",  //如果stage有子stage ,该子stage为inputStage
                        "nReturned" : 1,
                        "executionTimeMillisEstimate" : 20,
                        "works" : 2,  //指定查询执行阶段执行的“工作单元”的数量
                        "advanced" : 1,  //返回的中间结果数
                        "needTime" : 0,   //未将中间结果推进到其父级的工作周期数
                        "needYield" : 0,  //存储层要求查询系统产生的锁的次数
                        "saveState" : 1,
                        "restoreState" : 1,
                        "isEOF" : 1,       //指定执行阶段是否已到达流结束
                        "invalidates" : 0,
                        "docsExamined" : 1,
                        "alreadyHasObj" : 0,
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "nReturned" : 1,
                                "executionTimeMillisEstimate" : 20,
                                "works" : 2,
                                "advanced" : 1,
                                "needTime" : 0,
                                "needYield" : 0,
                                "saveState" : 1,
                                "restoreState" : 1,
                                "isEOF" : 1,
                                "invalidates" : 0,
                                "keyPattern" : {
                                        "code" : 1
                                },
                                "indexName" : "signalCode",
                                "isMultiKey" : false,  //本次查询是否使用了多键、复合索引
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 1,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "code" : [
                                                "[\"371602000100600010406001001\", \"371602000100600010406001001\"]"
                                        ]
                                },
                                "keysExamined" : 1,
                                "dupsTested" : 0,
                                "dupsDropped" : 0,
                                "seenInvalidated" : 0
                        }
                }
        },
        "serverInfo" : {
                "host" : "localhost.localdomain",
                "port" : 27017,
                "version" : "3.2.11",
                "gitVersion" : "009580ad490190ba33d1c6253ebd8d91808923e4"
        },
        "ok" : 1
}

			
```

## Query Plans

``` shell
    mongodb查询优化plan会根据可用的索引制定出最有效的查询plan,query planner业务逻辑是有没有存在查询计划,
    如果存在,那么评估一下该plan的性能是否符合要求,如果符合则根据该plan产生结果报告,如果不符合要求,则产生多个Query Plans
    从中筛选出符合性能要求query plan,再执行该plan产生结果报告.
    
    Plan Cache Flushes(查询计划更新)
        通过对索引和集合的操作都会导致query plan 被重新制定,如果mongod重启或则关闭,其query plan不会保存.
        
    Index Filters(索引过滤器)
        index filter 存在在query shape中,那么optimizer(优化程序)就只会考虑索引过滤器里的索引,来产生对应的query plan.
    这个时候hint()无效.

``` 

## mongodb 索引

``` shell

    当你往某各个集合插入多个文档后,每个文档在经过底层的存储引擎持久化后,会有一个位置信息,通过这个位置信息,
    就能从存储引擎里读出该文档.例如,person集合里包含插入了4个文档，假设其存储后位置信息如下
    
    位置信息	文档
        pos1	{“name” : “jack”, “age” : 19 }
        pos2	{“name” : “rose”, “age” : 20 }
        pos3	{“name” : “jack”, “age” : 18 }
        pos4	{“name” : “tony”, “age” : 21}
    	
    假设现在有个查询 db.person.find( {age: 18} ), 查询所有年龄为18岁的人,这时需要遍历所有的文档（『全表扫描』）,
    首先根据位置信息读出文档,然后在文档中对比age字段是否为18.当然如果只有4个文档,全表扫描的开销并不大,
    但如果集合文档数量到百万,甚至千万上亿的时候,对集合进行全表扫描开销是非常大的,一个查询耗费数十秒甚至几分钟都有可能.
    
    如果想加速 db.person.find({age: 18})就可以考虑对person表的age字段建立索引.
    
        > db.person.createIndex( {age: 1} )  // 按age字段创建升序索引
        
    建立索引后,MongoDB会额外存储一份按age字段升序排序的索引数据,索引结构类似如下，
    索引通常采用类似btree的结构持久化存储,以保证从索引里快速（O(logN)的时间复杂度）找出某个age值对应的位置信息,
    然后根据位置信息就能读取出对应的文档.
        AGE ----	位置信息
        18	----     pos3
        19	----     pos1
        20	----     pos2
        21	----     pos4
        
    索引就是将某个(某些)字段顺序的组织起来,这样可以按某个算法查找该字段,并得出对应的文档.
    
    至少能优化如下场景的效率：
    
        查询,比如查询年龄为18的所有人
        更新/删除,将年龄为18的所有人的信息更新或删除,因为更新或删除时,需要根据条件先查询出所有符合条件的文档,所以本质上还是在优化查询
        排序,将所有人的信息按年龄排序，如果没有索引，需要全表扫描文档，然后再对扫描的结果进行排序
        
    MongoDB默认会为插入的文档生成_id字段(如果应用本身没有指定其他字段),_id是文档唯一的标识.
    
    查询集合的索引信息
       > db.GuoDong_signal.getIndexes()
       [
               {
                       "v" : 1,
                       "key" : {
                               "_id" : 1
                       },
                       "name" : "_id_",
                       "ns" : "scloud-product.GuoDong_signal" 
               },
               {
                       "v" : 1,      // 索引版本
                       "key" : {     // 索引的字段及排序方向
                               "code" : 1   //对该字段进行升序
                       },
                       "name" : "signalCode",                     // 索引的名称
                       "ns" : "scloud-product.GuoDong_signal"     // 集合名
               }
       ]
        
        
        
    2.索引的分类
        (1) 单字段索引(Single Field Index)
                > db.person.createIndex( {age: 1} )  // 按age字段创建升序索引
                > db.person.createIndex( {age: -1} )  // 按age字段创建降序索引
                
        (2) 复合索引(Compound Index)
                它针对多个字段联合创建索引,先按第一个字段排序,第一个字段相同的文档按第二个字段排序,依次类推,
                如下针对age, name这2个字段创建一个复合索引
                
                > db.person.createIndex( {age: 1, name: 1} ) 
                
            上述索引对应的数据组织类似下表"
                
                AGE,NAME  ----	位置信息
                18,adam	  ----   pos5
                18,jack	  ----   pos3
                19,jack	  ----   pos1
                20,rose	  ----   pos2
                
            复合索引能满足的查询场景比单字段索引更丰富,不光能满足多个字段组合起来的查询,
            比如db.person.find( {age： 18, name: "jack"} ),也能满足匹配复合索引前缀的查询,
            这里{age: 1}即为{age: 1, name: 1}的前缀,所以类似db.person.find( {age： 18} )的查询也能通过该索引来加速；
            但db.person.find( {name: "jack"} )则无法使用该复合索引.
            如果经常需要根据『name字段』以及『name和age字段组合』来查询，则应该创建如下的复合索引
                > db.person.createIndex( {name: 1, age: 1} ) 
                
            age字段的取值很有限,即拥有相同age字段的文档会有很多；而name字段的取值则丰富很多,拥有相同name字段的文档很少；
            显然先按name字段查找,再在相同name的文档里查找age字段更为高效,索引列颗粒越小越好,颗粒就是在索引列中重复数据
            的数量.
                > db.person.createIndex({name:1,age:1})
                
        (3) 多key索引(Multikey Index）
                当索引的字段为数组["AAA","BBB"]时,创建出的索引称为多key索引,多key索引会为数组的每个元素建立一条索引,
                比如person表加入一个habbit字段（数组）用于描述兴趣爱好,需要查询有相同兴趣爱好的人就可以利用habbit字段的
                多key索引。
                    {"name" : "jack", "age" : 19, habbit: ["football, runnning"]}
                    > db.person.createIndex( {habbit: 1} )  // 自动创建多key索引
                    > db.person.find( {habbit: "football"} )
                    
        (4)文档索引:等同于复合索引
                {name:"xyz",metro:{city:"New York",state:"NY"}}
            > db.person.createIndex( {metro: 1} ) == db.person.createIndex({metro.city: 1, metro.state: 1}) 
    
     hint()函数:
        要查询的内容中包含多个字段且都有索引，我们可以使用hint()函数来强制使用特定索引,这是一个优化方法.
     
```

- 数据库的设计

``` shell
    要了解这个集合是频繁写,还是频繁读.

``` 


## mongostat shell 命令

``` shell

    1.监控mongod的内存情况
        > db.serverStatus().mem
            {
                    "bits" : 64,
                    "resident" : 97,  物理内存单位 MB
                    "virtual" : 432,  虚拟内存 MB
                    "supported" : true,
                    "mapped" : 0,
                    "mappedWithJournal" : 0
            }
    	
    2. 查看连接数
        > db.serverStatus().connections
        
    3. 容量大小
    
        > db.stats(scale)
           其中 参数scale是可选择的, scale = 1024,则结果是以KB为单位
        > db.stats()
            {
                    "db" : "scloud-test2",  数据库名
                    "collections" : 24,    集合的数量
                    "objects" : 694,       该数据库中所有documents数量(查询所有collectons)
                    "avgObjSize" : 828.3876080691642,  每个documents的平均大小,单位Byte
                    "dataSize" : 574901,  单位Byte
                    "storageSize" : 778240,
                    "numExtents" : 0,
                    "indexes" : 23,   该数据库的所有索引数
                    "indexSize" : 569344, 单位Byte
                    "ok" : 1
            }
``` 

- mongo 命令

``` shell

    1.监控mongod的内存情况
        > db.serverStatus().mem
            {
                    "bits" : 64,
                    "resident" : 97,  物理内存单位 MB
                    "virtual" : 432,  虚拟内存 MB
                    "supported" : true,
                    "mapped" : 0,
                    "mappedWithJournal" : 0
            }
    	
    2. 查看连接数
        > db.serverStatus().connections 
        
     > db.serverStatus() 如果出现not authorized on admin to execute command serverstatus 1.0,
     可以选择先切换到admin数据库中,
        > db.createUser(
          {
            user: "admin",
            pwd: "password",
            roles: [ { role: "root", db: "admin" } ]
          }
        );
        > show users
        
        再使用db.serverStatus()
        
        
        
    3. 容量大小
    
        > db.stats(scale)
           其中 参数scale是可选择的, scale = 1024,则结果是以KB为单位
        > db.stats()
            {
                    "db" : "scloud-test2",  数据库名
                    "collections" : 24,    集合的数量
                    "objects" : 694,       该数据库中所有documents数量(查询所有collectons)
                    "avgObjSize" : 828.3876080691642,  每个documents的平均大小,单位Byte
                    "dataSize" : 574901,  单位Byte
                    "storageSize" : 778240,
                    "numExtents" : 0,
                    "indexes" : 23,   该数据库的所有索引数
                    "indexSize" : 569344, 单位Byte
                    "ok" : 1
            }
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

# Storage (mongod 的存储方式)

## WiredTiger Storage Engine

```c++
    在版本3.2以后,默认的存储方式采用WiredTiger Storage Engine
    
    1.Document Level Concurrency(文档级别的并发)
	    WiredTiger机制对数据库的写操作是基于文档级别的并发,这就意味着可以有多个客户端同时对同一个collection的不同
	    documents(进行操作).
	    
	2. Snapshots and Checkpoints(快照和检查点)
	    
	     WiredTiger使用了多版本并发控制,当数据开始一个操作时,WiredTiger对当时的数据进行了快照,保存了在内存中的相同
	     视图(数据).
	     当要写入磁盘时,WiredTiger将Snapshots里的所有数据一致的写到磁盘的文件中,同时作为Checkpoint的数据一定是
	     可持久性的数据,Checkpoint可以充当数据恢复点.创建checkpoints间隔是60秒或则journal data.容量达到
	     2 gigabytes(千兆字节).可以通过 $ mongod --syncdelay <value> 来改变频率,不过不建议.
	     
	     Snapshots and Checkpoints 是为了保证mongo异常退出时,数据可以恢复.
	     
	     
```

