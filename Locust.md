# Locust 性能压力测试

## 概述
```shell
    1. 官网(https://locust.io/)
    2. 性能工具比较
            (1) LoadRunner: 商业性能测试工具，功能非常强大,使用也比较复杂,市面上常用
            (2) Jmeter: 开源性能测试工具，功能也很完善,是一个标准的性能测试工具
            (3) Locust: 功能较上面两个要少,但支持 python 扩展,里面有内置的 HttpLocust 库(支持 http/https 协议)
                        同时可以自己编写支持自定义的协议.LoadRunner 和 Jmeter 这类采用进程和线程的测试工具，
                        很难在单机上模拟出较高的并发压力。Locust 的并发机制采用协程（gevent）的机制。避免了系统级资源调度，
                        可以大幅提高单机的并发能力。
    3. 
        http://www.testclass.net/locust/introduce/?
```

## 安装
```shell
    1. Locust 依赖 python 库(查看 setup.py)
            (1) gevent 是在 Python 中实现协程的一个第三方库。协程，又称微线程（Coroutine）使用gevent可以获得极高的并发性能。
            (2) flask 是 Python 的一个 Web 开发框架。
            (3) Requests 用来做 HTTP 接口测试。
            (4) msgpack-python 是一种快速、紧凑的二进制序列化格式，适用于类似 JSON 的数据。
            (5) six 提供了一些简单的工具用来封装 Python2 和 Python3 之间的差异性。
            (6) pyzmq 如果你打算运行 Locust 分布在多个进程/机器，建议你安装 pyzmq
```
## 运行

```shell
1. 运行 locust 测试
    > locust [options] [LocustClass [LocustClass2 ... ]]
        如果 Locust 性能测试脚本内只有一个 LocustClass , 则命令行中可以不写.
    > locust -f uitd-testing.py
2. 访问 LOCUST 的 dashboard 127.0.0.1:8089
   (1) Number of users to simulate 设置模拟用户数
   (2) Hatch rate (users spawned/second) : 每秒启动虚拟用户数
   (3) 运行的结果参数:
            Type： 请求的类型，例如 GET/POST。
                Name：请求的路径。这里为百度首页，即：https://www.baidu.com/
                request：当前请求的数量。
                fails：当前请求失败的数量。
                Median：中间值，单位毫秒，一半的服务器响应时间低于该值，而另一半高于该值。
                Average：平均值，单位毫秒，所有请求的平均响应时间。
                Min：请求的最小服务器响应时间，单位毫秒。
                Max：请求的最大服务器响应时间，单位毫秒。
                Content Size：单个请求的大小，单位字节。
                Current RPS(reqs/sec)：是每秒钟请求的个数
   
3. 参数(option)
         -f 指定性能测试脚本文件
         --host 指定被测试应用的 URL 的地址 --host=https://www.baidu.com
     命令行运行 Locust 测试(locust -f load_test.py --host=https://www.baidu.com --no-web -c 10 -r 2 -t 1m)
         --no-web 表示不使用 Web 界面运行测试。
             -c 设置虚拟用户数。
             -r 设置每秒启动虚拟用户数。
             -t 设置设置运行时间。300s, 20m, 3h, 1h30m 
             
             --port=PORT : 如果采用 web 形式运行性能测试,指定 web 监听的端口号, 默认是 8089
             
         分布式部署
            --master : Locust 分布式模式使用，当前节点为 master 节点
            --slave  : Locust 分布式模式使用，当前节点为 slave 节点。
            --master-host=MASTER_HOST : 作为 slave 配置连接的 master 节点的主机或 IP 地址，
                                        只能与 --slave 节点一起运行时使用，默认为：127.0.0.1.
            --master-port=MASTER_PORT: (可选项)作为 slave 配置连接的 master 节点的 port, 只能与 --slave 节点一起运行时使用
                                        默认为：5557。slave 节点也将连接到这个端口 + 1 上的 master 节点, 也就是说
                                        slave 会同时连接 5557 和 5558
            --master-bind-host=MASTER_BIND_HOST
                    可选项，与 --master 一起结合使用。决定在 master 模式下将会绑定什么网络接口。默认设置为*(所有可用的接口)。
            
            --master-bind-port=5557
                    可选项，与 --master 一起结合使用。决定哪个端口 master 模式将会监听。默认设置为 5557。
                    注意 Locust 会使用指定的端口号，同时指定端口+1的号也会被占用。因此Locust 将会使用 5557 和 5558。
            
            --expect-slaves=EXPECT_SLAVES
                在 no-web 模式下启动 master 时使用。master 将等待 EXPECT_SLAVES 节点被连接,才能执行 Locust 测试
                                        
    4. Locust 分布式
        (1) 在一台独立的机器中运行 master, 该 master 节点不会模拟任何用户,只负责 slave 数据的统计,
           在 slave 机器中每个处理器内核运行一个 slave 实例(通过 --master-host 指定 master 的 ip)
        (2) 
            master 节点:
                > locust -f my_loucstfile.py --master
            slave 节点:
                > locust -f my_locustfile.py --slave --master-host=192.168.0.14
```


## 例子解释

```shell
    from locust import HttpLocust, TaskSet, task
    
    class UserBehavior(TaskSet):  ## UserBehavior 类继承 TaskSet 类
    
        @task(1)  ## @task() 修饰该方法为任务,1 表示一个 Locust 实例被挑选执行的权重，数值越大，执行频率越高。
                  ## 在当前 UserBehavior() 行为下只有一个 baidu() 任务,所以设置权重没有意义
        def baidu(self):
            self.client.get("/")
     
    class WebsiteUser(HttpLocust):
        task_set = UserBehavior  # 指向一个定义了的用户行为类
        min_wait = 3000  # 用户执行任务之间等待时间的下限, ms
        max_wait = 6000  # 用户执行任务之间等待时间的上限, ms
        
```

## Locust 类和方法

```shell
    1. HttpLocust 类(继承自 Locust 类)
        每一个模拟的用户都看做是 HttpLocust 的一个实例.每个实例都有一个 client 属性, 能够用于生成 HTTP 请求的实例
        属性:
            task_set: 指向一个 TaskSet 类, TaskSet 类定义了每个用户的行为
            min_wait: 用户执行任务之间等待时间的下限, ms, 默认是 1000 ms, 可以被 TaskSet 类覆盖
            max_wait: 用户执行任务之间等待时间的上限, ms, 默认是 1000 ms, 可以被 TaskSet 类覆盖
            host: 如果是 Web 服务的测试，host 相当于是提供 URL 前缀的默认值，如果在命令行中指定了 --host 选项，
                  则以命令行中指定的为准。如果不是 Web 服务测试，使用默认的 None 。
            stop_timeout: 设置 Locust 多少秒后超时，如果为 None ,则不会超时
            weight: 一个 Locust 实例被挑选执行的权重，数值越大，执行频率越高。在一个 locustfile.py 文件中可以同时定义多个 
                    HttpLocust 子类，然后分配他们的执行权重, 如果 weight 没有被赋值,并且在命令行中没有指定对应的特定的 
                    locust 类, 则会随机调用 Locust 实例. 同时也可以在命令行指定 Locust 实例, 
                    > locust -f locust_file.py WebUserLocust MobileUserLocust
        实例:
            from locust import HttpLocust, TaskSet, task
            
            class UserTask(TaskSet):
            
                @task
                def tc_index(self):
                    self.client.get("/")
            
            class UserOne(HttpLocust):
                task_set = UserTask
                weight = 1
                min_wait = 1000
                max_wait = 3000
                stop_timeout = 5
                host = "https://www.baidu.com"
            
            class UserTwo(HttpLocust):
                weight = 2
                task_set = UserTask
                host = "https://www.baidu.com"
                
        运行:
            > locust -f load_test.py UserOne UserTwo
    2. TaskSet 类
        (1) TaskSet 类定义每个用户的任务集合，测试任务开始后，每个 Locust 用户会从 TaskSet 中随机挑选一个任务执行，
            然后随机等待 HttpLocust 类中定义的 min_wait和 max_wait 之间的一段时间，执行下一个任务
        (2) TaskSet 类内部 self.client 等同于 self.locust.client
        (3) 生成 GET 请求
                response = self.client.get("/about")  
                print "Response status code:", response.status_code  
                print "Response content:", response.content 
        (4) 生成 post 请求
                response = self.client.post("/login", {"username":"testuser", "password":"secret"})  
        (5) http 人工控制一个请求的响应为成功还是失败
                with client.get("/", catch_response=True) as response:  
                    if response.content != "Success":
                        response.failure("Got wrong response")  # 失败
                        response.success() # 成功
                        
        (6) TaskSet 的嵌套
                class ForumPage(TaskSet):
                    @task(20)
                    def read_thread(self):
                        pass
          
                    @task(1) 
                    def new_thread(self):
                        pass
                
                    @task(5)
                    def stop(self):
                        self.interrupt()
                
                class UserBehaviour(TaskSet):
                    tasks = {ForumPage:10}
                    
                    @task
                    def index(self):
                        pass
                        
                其中 UserBehaviour 任务集合嵌套了 ForumPage 任务集合, 那么 ForumPage 任务集合中就一定要定义 interrupt 
                这个函数,跳出 ForumPage 任务集,执行 UserBehaviour 中其他的任务.
        (7)
            属性:
                tasks : 效果与 @task 一样,实际上使用 @task 就是填充 tasks 属性, tasks 可以是<callable(函数名), int> 的字典表,
                        如果 tasks 是 [...] list 的形式, 则里面的任务是随机的调用.如果 tasks 是 dict 的形式 
                        {key1:value1, key2:value2}, 则 value1 代表 key1函数调用的权重
                实例:
                    def my_task(l):
                        pass
                    def another_task(l):
                        pass
                    
                    class MyTaskSet(TaskSet):
                        tasks = [my_task]
                        tasks = {my_task: 3, another_task: 1} # my_task 函数调用频率是 another_task 的 3 倍
                    
                
    3. Locust 类
        (1) 代表最底层的用户类.
        (2) 类的属性
                1. task_set, 每一个 Locust 类都必须有 task_set 属性,指向任务集
                2. min_wait, max_wait
                3. weight
    4. TaskSequence 类(顺序执行任务类)
        (1) 实例:
                class MyTaskSequence(TaskSequence):
                    @seq_task(1)     ## 第一个被调函数
                    def first_task(self):
                        pass
                
                    @seq_task(2)    ## 第二个被调函数
                    def second_task(self):
                        pass
                
                    @seq_task(3)
                    @task(10)      ## 第三个被调函数,并且要调 10 次
                    def third_task(self):
                        pass
    5. 
        (1) setup ,teardown 方法支持 Locust 类 和 TaskSet 类, setup ,teardown 只执行一次, setup 是在 tasks 任务
            运行之前启动的, 而 teardown 则是在所有任务完成后运行的.
        (2) on_start, on_stop 应用与 TaskSet , on_start 被调用是一个模拟用户执行 TaskSet class, 当 TaskSet 停止时
            on_stop 被调用.
        (3) 调用顺序
                1. Locust setup
                2. TaskSet setup
                3. TaskSet on_start
                4. TaskSet tasks…
                5. TaskSet on_stop
                6. TaskSet teardown
                7. Locust teardown   
    6. events 类(可以基于其他的通信协议)
         from locust import events
         request_success 方法和 request_failure 方法可以嵌入到 locust 框架的统计中(statistics)
         实例:
             locust 统计失败的结果
             events.request_failure.fire(request_type="tcpsocket", name="connect", 
                                         response_time=total_time, exception=e)
                                         
             locust 统计成功的结果
             events.request_success.fire(request_type="tcpsocket", name="connect", 
                                         response_time=total_time, response_length=0)
             
         
```