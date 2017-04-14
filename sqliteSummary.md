# SQlite使用方法

- 按某种顺序限定显示范围


``` shell

SELECT expressions
FROM tables
[WHERE conditions]
[ORDER BY expression [ ASC | DESC ]]
LIMIT number_rows OFFSET offset_value;

返回 个数为number_rows条，从offset_value开始的记录

例如：按time降序限定第一条记录，选出capcity_array值
string sqlCmd = "SELECT capcity_array FROM "\
            + tableName + " ORDER BY time DESC LIMIT 1";
			
```
[链接1](https://www.techonthenet.com/sqlite/select_limit.php)
[链接2](http://www.cnblogs.com/wangxingliu/p/3512188.html)

- int sqlite3_changes(sqlite3*) 返回修改的行数

``` shell

功能：此函数返回最近完成的INSERT修改的行数，UPDATE插入的行数或DELETE语句删除的行数。 执行任何其他类型的SQL语句返回0。
			
```