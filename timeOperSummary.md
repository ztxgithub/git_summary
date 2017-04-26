# 基础知识

## shell命令相关的

- UTC/GMT：世界标准时间

- CST ：北京时间(China Standard Time) ，比世界标准时间早8小时，记为UTC+8

- 关系: CST = UTC/GMT +8 小时

## shell命令

- 显示该系统的当前时区

```shell

	> date -R  (以某种格式输出，其实里面包含了时区信息)
	Sat, 22 Apr 2017 16:58:38 +0800
	
	> date
	Sat Apr 22 17:38:34 CST 2017

```

- 将该系统的当前时区时间转化为UTC

```shell

	> date -u
	Sat Apr 22 09:39:06 UTC 2017

```

## 系统函数相关

- time_t:数据类型

```c

	time_t == long int
	
	含义：从UTC(coordinated universal time)时间1970年1月1日00时00分00秒(也称为Linux系统的Epoch时间)到当前时刻的秒数。  
	由于time_t类型长度的限制，它所表示的时间不能晚于2038年1月19日03时14分07秒(UTC)

```


- 设置系统的时区（将系统已某个特定的时区显示）

```shell

	1.确保 /usr/share/zoneinfo/ 目录下面有对应时区信息（例如/usr/share/zoneinfo/Asia/Shanghai）
	2.要在开机自启时 TZ='Asia/Shanghai'; export TZ

```

# 相关系统函数

- 从1970年1月1日00时00分00秒至今所经过的秒数

