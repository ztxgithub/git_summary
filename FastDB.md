# FastDB

## 简要

```
1. FastDB 是基于面向对象的编程,将表中的行作为对象的实例，表则是作为这些对象的类，每一个查询的执行结果就是一个类的一组对象
2. 结构和数组可以用作记录成员。提供了一个特定的 exists 引用来定位数组中的元素
3. 
(1) dbDatabase::dbAllAccess，一旦某个进程使用该模式访问表，如果该进程使用了 insert、update、delete 等修改数据的操作，
    其他访问该库的进程的所有操作（包括 open、select）都会被阻塞，直到该操作提交或回滚
  (2)dbDatabase::dbConcurrentUpdate，使用该模式访问表，如果某个进程对数据进行修改性的操作，同时另外的进程使用
        dbDatabase::dbReadOnly 或者 dbDatabase::dbConcurrentRead 读取数据，不会出现阻塞的情况。但是dbDatabase::dbReadOnly 
        会把未提交的脏数据读出来；而 dbDatabase::dbConcurrentRead 则不会 
       (3) 如果多个进程都使用dbDatabase::dbConcurrentUpdate，实际效果和dbDatabase::dbAllAccess一样，一旦一个进程修改了数据，
        其他进程所有的操作（包括open、select）都将阻塞，直到该操作提交或回滚
(4) 如果你只需要访问数据，那么最好是使用 dbDatabase::dbReadOnly 或者是 dbDatabase::dbConcurrentRead。
    访问操作不会因为这个表被其他进程修改数据而阻塞。
(5) 如果一个表有不止一个使用者，那么涉及修改数据的进程应该使用 dbDatabase::dbConcurrentUpdate，
  而只读的进程使用则使用 dbDatabase::dbConcurrentRead。
(6) 如果表只有一个使用者，则可以根据需要，使用dbDatabase::dbAllAccess 或者 dbDatabase::dbReadOnly
(7) 所有的数据修改操作，如果有并发要求，一定要及时提交

```