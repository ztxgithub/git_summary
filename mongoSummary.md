# mongod使用方法

- systemctl restart mongod.service 启动失败

``` shell

    原因：1./var/lib/mongo及子目录文件所有者都应该使mongod用户
    （chown mongod:mongod /var/lib/mongo && chown -R mongod:mongod /var/lib/mongo/）
    
         2./var/log/mongodb及子目录文件所有者都应该使mongod用户
    （chown mongod:mongod /var/log/mongodb && chown -R mongod:mongod /var/log/mongodb）
    
         3./var/log/mongodb及子目录文件所有者都应该使mongod用户
     (sudo chown mongod:mongod /tmp/mongodb-27017.sock)
			
```
