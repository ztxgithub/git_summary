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

# mongo shell 命令

- 切换到对应的数据库(database)

``` shell

    > use scloud-test (切换到对应的数据库,这个时候还要用户名和密码,才能对数据库进行操作)
    
    > db.auth("test", "YYtd_test")
    
			
```