# 1.distributed_framework
```
python分布式函数调度框架。适用场景范围超级广泛。

支持python内置Queue对象作为当前解释器下的消息队列。
支持sqlite3作为本机持久化消息队列。
支持pika包实现的使用rabbitmq作为分布式消息队列。
支持rabbitpy包实现的使用rabbitmq作为分布式消息队列。
支持amqpstorm包实现的使用rabbitmq作为分布式消息队列。
支持redis中间件作为分布式消息队列。（不支持消费确认，例如把消息取出来了，但函数还在运行中没有运行完成，
突然关闭程序或断网断电，会造成部分任务丢失【设置的并发数量越大，丢失数量越惨】，所以推荐mq）
支持mongodb中间件作为分布式消息队列。
支持nsq中间件作为分布式消息队列。
支持kafka中间件作为分布式消息队列。


源码实现思路100%遵守了oop的6个设计原则，很容易扩展中间件。
1、单一职责原则——SRP
2、开闭原则——OCP
3、里式替换原则——LSP
4、依赖倒置原则——DIP
5、接口隔离原则——ISP
6、迪米特原则——LOD
可以仿照源码中实现中间件的例子，只需要继承发布者、消费者基类后实现几个抽象方法即可添加新的中间件。



将函数名和队列名绑定，即可开启自动消费。

只需要一行代码就 将任何函数实现 分布式 、并发、 控频、断点接续运行、定时、指定时间不运行、
消费确认、重试指定次数、重新入队、超时杀死、计算消费次数速度、预估消费时间、
函数运行日志记录、任务过滤、任务过期丢弃等数十种功能。

和celery一样支持线程、gevent、eventlet 并发运行模式，大大简化比使用celery，很强大简单，已在多个生产项目和模块验证。
确保任何并发模式在linux和windows，一次编写处处运行。不会像celery在windwos上某些功能失效。

没有严格的目录结构，代码可以在各个文件夹层级到处移动，脚本名字可以随便改。不使用令人讨厌的cmd命令行启动。
没有大量使用元编程，使代码性能提高，减少代码拼错概率和降低调用难度。
并且所有重要公有方法和函数设计都不使用*args，**kwargs方式，全部都能使ide智能提示补全参数，一切为了ide智能补全。
```

# 1.1  pip安装方式
pip install function_scheduling_distributed_framework --upgrade -i https://pypi.org/simple 

# 2.具体更详细的用法可以看test_frame文件夹里面的几个示例。
 ```python
import time

from function_scheduling_distributed_framework import patch_frame_config, show_frame_config,get_consumer


# 初次接触使用，可以不安装任何中间件，使用本地持久化队列。正式墙裂推荐安装rabbitmq。
patch_frame_config(MONGO_CONNECT_URL='mongodb://myUserAdminxx:xxxx@xx.90.89.xx:27016/admin',

                       RABBITMQ_USER='silxxxx',
                       RABBITMQ_PASS='Fr3Mxxxxx',
                       RABBITMQ_HOST='1xx.90.89.xx',
                       RABBITMQ_PORT=5672,
                       RABBITMQ_VIRTUAL_HOST='test_host',

                       REDIS_HOST='1xx.90.89.xx',
                       REDIS_PASSWORD='yxxxxxxR',
                       REDIS_PORT=6543,
                       REDIS_DB=7,

                       NSQD_TCP_ADDRESSES=['xx.112.34.56:4150'],
                       NSQD_HTTP_CLIENT_HOST='12.34.56.78',
                       NSQD_HTTP_CLIENT_PORT=4151,

                       KAFKA_BOOTSTRAP_SERVERS=['12.34.56.78:9092'],
                       )

show_frame_config()

# 主要的消费函数，演示做加法，假设需要花10秒钟。
def f2(a, b):
    print(f'消费此消息 {a} + {b} ,结果是  {a + b}')
    time.sleep(10)  # 模拟做某事需要阻塞10秒种，必须用并发绕过此阻塞。


# 把消费的函数名传给consuming_function，就这么简单。
# 通过设置broker_kind，一键切换中间件为mq或redis等7种中间件或包。
# 额外参数支持超过10种控制功能，celery支持的控制方式，都全部支持。
# 这里演示使用本地持久化队列，本机多个脚本之间可以共享任务，无需安装任何中间件，降低初次使用门槛。
consumer = get_consumer('queue_test2', consuming_function=f2, broker_kind=6)  



# 推送需要消费的任务，可以变消费边推送。发布的内容字典需要和函数所能接收的参数一一对应，
# 并且函数参数需要能被json序列化，不要把自定义的类型作为消费函数的参数。
consumer.publisher_of_same_queue.clear()
[consumer.publisher_of_same_queue.publish({'a': i, 'b': 2 * i}) for i in range(100)]


# 开始从中间件循环取出任务，使用指定的函数消费中间件里面的消息。
consumer.start_consuming_message()

 ```
 
### 3.1运行中截图
![Image text](https://i.niupic.com/images/2019/08/09/_477.png)

### 3.2控频功能证明，由于截图是外网调度rabbitmq的消息有延迟，没有精确到函数每秒运行10次。

![Image text](https://i.niupic.com/images/2019/08/09/_462.png)


## 4.celery和这个框架比，存储的内容差异
### 4.1celery的
 ```
 {
  "body": "W1szLCA2XSwge30sIHsiY2FsbGJhY2tzIjogbnVsbCwgImVycmJhY2tzIjogbnVsbCwgImNoYWluIjogbnVsbCwgImNob3JkIjogbnVsbH1d",
   "content-encoding":  "utf-8",
   "content-type":  "application/json",
   "headers":  {
    "lang":  "py",
     "task":  "test_task\u554a",
     "id":  "39198371-8e6a-4994-9f6b-0335fe2e9b92",
     "shadow":  null,
     "eta":  null,
     "expires":  null,
     "group":  null,
     "retries":  0,
     "timelimit":  [
      null,
       null
    ],
     "root_id":  "39198371-8e6a-4994-9f6b-0335fe2e9b92",
     "parent_id":  null,
     "argsrepr":  "(3, 6)",
     "kwargsrepr":  "{}",
     "origin":  "gen22848@FQ9H7TVDZLJ4RBT"
  },
   "properties":  {
    "correlation_id":  "39198371-8e6a-4994-9f6b-0335fe2e9b92",
     "reply_to":  "3ef38b98-1417-3f3d-995b-89e8e15849fa",
     "delivery_mode":  2,
     "delivery_info":  {
      "exchange":  "",
       "routing_key":  "test_a"
    },
     "priority":  0,
     "body_encoding":  "base64",
     "delivery_tag":  "59e39055-2086-4be8-a801-993061fee443"
  }
}
  ```

### 4.2 此框架的消息很短，就是一个字典，内容的键值对和函数入参一一对应。
额外控制参数如重试、超时kill，由代码决定，
不需要存到中间件里面去。例如函数运行超时大小在本地代码修改后，立即生效。

不由中间件里面的配置来决定。
 ```   
{"a":3,"b":6}
  ```
  



# 5.常见问题回答
#### 5.1 你干嘛要写这个框架？和celery 、rq有什么区别？是不是完全重复造轮子为了装x？

  ```
 答：与rq相比，rq只是基于redis一种中间件键，并且连最基本的并发方式和并发数量都没法指定
，更不用说包括控频 等一系列辅助控制功能，功能比celery差太远了，也和此框架不能比较。

这个框架的最开始实现绝对没有受到celery框架的半点启发。
这是从无数个无限重复复制粘贴扣字的使用redis队列的py文件中提取出来的框架。


原来有无数个这样的脚本。以下为伪代码演示，实际代码大概就是这个意思。

while 1：
    msg = redis_client.lpop('queue_namex')
    funcx(msg)

原来有很多个这样的一些列脚本，无限次地把操作redis写到业务流了。
排除无限重复复制粘贴不说，这样除了redis分布式以外，缺少了并发、
 控频、断点接续运行、定时、指定时间不运行、
消费确认、重试指定次数、重新入队、超时杀死、计算消费次数速度、预估消费时间、
函数运行日志记录、任务过滤、任务过期丢弃等数十种功能。

所以这个没有借鉴celery，只是使用oop转化公式提取了可复用流程的类，然后慢慢增加各种辅助控制功能和中间件。
这个和celery一样都属于万能分布式函数调度框架，函数控制功能比rq丰富。比celery的优点见以下解答。

  ```

#### 5.2 为什么包的名字这么长，为什么不学celery把包名取成  花菜 茄子什么的？
  ```
  答： 为了直接表达框架的意思。
   ```
#### 5.3 支持哪些消息队列中间件。
   ```
   答： 支持python内置Queue对象作为当前解释器下的消息队列。
        支持sqlite3作为本机持久化消息队列。
        支持pika包实现的使用rabbitmq作为分布式消息队列。
        支持rabbitpy包实现的使用rabbitmq作为分布式消息队列。
        支持amqpstorm包实现的使用rabbitmq作为分布式消息队列。
        支持redis中间件作为分布式消息队列。（不支持消费确认，例如把消息取出来了，但函数还在运行中没有运行完成，
        突然关闭程序或断网断电，会造成部分任务丢失【设置的并发数量越大，丢失数量越惨】，所以推荐mq）
        支持mongodb中间件作为分布式消息队列。
```
 
 #### 5.4 各种中间件的优劣？
   ```
   答： python内置Queue对象作为当前解释器下的消息队列，
       优势：不需要安装任何中间件，消息存储在解释器的变量内存中，调度没有io，传输速度损耗极小，
             和直接一个函数直接调用另一个函数差不多快。
       劣势：只支持同一个解释器下的任务队列共享，不支持单独启动的两个脚本共享消息任务。  
             没有持久化，解释器退出消息就消失，代码不能中断。   没有分布式。
   
       persistqueue sqlite3作为本机持久化消息队列，
       优势： 不需要安装中间件。可以实现本机消息持久化。支持多个进程或多个不同次启动那个的脚本共享任务。
       劣势： 只能单机共享任务。 无法多台机器共享任务。分布式能力有限。
       
       redis 作为消息队列，
       优势： 真分布式，多个脚本和多态机器可以共享任务。
       劣势： 需要安装redis。 是基于redis的list数组结构实现的，不是真正的ampq消息队列，
              不支持消费确认，所以你不能在程序运行中反复随意将脚本启动和停止或者反复断电断网，
              这样会丢失一部分正在运行的消息任务，断点接续能力弱一些。
       
       mongo 作为消息队列：
       优势： 真分布式。是使用mongo col里面的的一行一行的doc表示一个消息队列里面的一个个任务。支持消费确认。
       劣势： mongo本身没有消息队列的功能，是使用三方包mongo-queue模拟的消息队列。性能没有专用消息队列好。
       
       rabbitmq 作为消息队列：
       优势： 真正的消息队列。支持分布式。支持消费确认，支持随意关闭程序和断网断电。连金融支付系统都用这个，可靠性极高。完美。
       劣势： 大多数人没安装这个中间件，需要学习怎么安装rabbitmq，或者docker安装rabbitmq。
       
       kafka 作为消息队列：
       优势：性能好吞吐量大，中间件扩展好。kafka的分组消费设计得很好，
            还可以随时重置偏移量，不需要重复发布旧消息。
       劣势：和传统mq设计很大的不同，使用偏移量来作为断点接续依据，需要消费哪些任务不是原子性的，
            偏移量太提前导致部分消息重复消费，太靠后导致丢失部分消息。性能好需要付出可靠性的代价。
            高并发数量时候，主题的分区如果设置少，确认消费实现难，框架中采用自动commit的方式。
            要求可靠性 一致性高还是用rabbitmq好。
            
       nsq 作为消息队列：
       优势：部署安装很简单，比rabittmq和kafka安装简单。性能不错。
       劣势：用户很少，比rabbitmq用户少了几百倍，导致资料也很少，需要看nsq包的源码来调用一些冷门功能。
       
```
 
 #### 5.5 比celery有哪些功能优点。
 ```
   答： 1） 如5.4所写，新增了python内置 queue队列和 基于本机的此计划消息队列。不需要安装中间件，即可使用。
        2） 性能比celery框架提高一丝丝。
        3） 公有方法，需要被用户调用的方法或函数一律都没有使用元编程，不需要在消费函数上加app.task这样的装饰器。
            例如不 add.delay(1,2)这样发布任务。 不使用字符串来import 一个模块。
            过多的元编程过于动态，降低不仅会性能，还会让ide无法补全提示，动态一时爽，重构火葬场不是没原因的。
        4） 全部公有方法或函数都能在pycharm下只能提示补全参数名称和参数类型。
            一切为了调用方便，例如get_consumer函数和AbstractConsumer的入参完全重复了，本来事项的时候可以使用*args **kwargs来省略入参，
            但这样会造成ide不能补全提示，此框架一切写法只为给调用者带来使用上的方便。不学celery让用户不知道传什么参数。
            如果拼错了参数，pycharm会显红，大大降低了用户调用出错概率。
         5）不使用命令行启动，在cmd打那么长的一串命令，容易打错字母。并且让用户不知道如何正确的使用celery命令，不友好。
            此框架是直接python xx.py 就启动了。
         6）框架不依赖任何固定的目录结构，无结构100%自由，想把使用框架写在哪里就写在哪里，写在10层级的深层文件夹下都可以。
            脚本可以四处移动改名。celery想要做到这样，要做额外的处理。
         7）使用此框架比celery更简单10倍，如例子所示。使用此框架代码绝对比使用celery少几十行。
         8）消息中间件里面存放的消息任务很小，简单任务 比celery的消息小了50倍。 消息中间件存放的只是函数的参数，辅助参数由consumer自己控制。
            消息越小，中间件性能压力越小。
         9）由于消息中间件里面没有存放其他与python 和项目配置有关的信息，这是真正的跨语言的函数调度框架。
            java人员也可以直接使用redis类rabbitmq类，发送jso参数到中间件，由python消费。celery里面的那种参数，高达几十项
            和项目配置混合了，java人员绝对拼凑不出来这种格式的消息结构。
         10）简单利于团队推广，不需要看复杂的celry 那样的5000页英文文档。
            
 ```


#### 5.6 框架是使用什么序列化协议来序列化消息的。

 ```
    答：框架默认使用json。并且不提供序列化方式选择，有且只能用json序列化。json消息可读性很强，远超其他徐内化方式。
    默认使用json来序列化和反序列化消息。所以推送的消息必须是简单的，不要把一个自定义类型的对象作为消费函数的入参，
    json键的值必须是简单类型，例如 数字 字符串 数组 字典这种。不可以是不可被json序列化的自定义类型的对象。
    
    用json序列化已经满足所有场景了，picke序列化更强，但仍然有一些自定义类型的对象的实例属性由于是一个不可被序列化
    的东西，picke解决不了，这种东西例如self.r = Redis（）  ,不可以序列化，就算能序列化也是要用一串很长的东西来
    表示这种属性，导致中间件要存储很大的东西传输效率会降低，这种完全可以使用json来解决，
    例如指定ip 和端口，在消费函数内部来使用redis。所以用json一定可以满足一切传参场景。
    
    如果json不能满足你的消费任务的序列化，那不是框架的问题，一定是你代码设计的问题。所以不打算准备加入其他序列化方式。
  ```
  

#### 5.6 框架如何实现定时？

 ```
    答：没有定时功能。
        安装三方shchduler框架，消费进程常驻后台，shchduler框架定时推送一个任务到消息队列，定时推送了消息自然就能定时消费。
  ```
  
#### 5.7 为什么强调是函数调度框架不是类调度框架？你代码里面使用了类，是不是和此框架水火不容了?
 ```
    答：一切对类的调用最后都是体现在对方法的调用。这个问题莫名其妙。
    celery rq huery 框架都是针对函数。
    调度函数而不是类是因为：
    1）类实例化时候构造方法要传参，类的公有方法也要传参，这样就不确定要把中间件里面的参数哪些传给构造方法哪些传给普通方法了。
       见5.8
    2） 这种分布式一般要求是幂等的，传啥参数有固定的结果，函数是无依赖状态的。类是封装的带有状态，方法依赖了对象的实例属性。
    
    框架如何调用你代码里面的类。
    假设你的代码是：
    class A():
       def __init__(x):
           self.x = x
        
       def add(y):
           print( self.x + y)
    
    那么你需要再写一个函数
    def your_task(x,y):
        return  A(x).add(y)
    然后把这个函数传给框架就可以了。所以此框架和你在项目里面写类不是冲突的。
  ```
  
  #### 5.8 是怎么调度一个函数的。
 ```
     答：基本原理如下
     
     def add(x,y):
         print(x + y)
         
     从消息中间件里面取出参数{"a":1,"b":2}
     然后使用  add(**{"a":1,"b":2}),就是这样运行函数的。
  ```
  #### 5.9 框架适用哪些场景？
 ```
      答：分布式 、并发、 控频、断点接续运行、定时、指定时间不运行、
          消费确认、重试指定次数、重新入队、超时杀死、计算消费次数速度、预估消费时间、
          函数运行日志记录、任务过滤、任务过期丢弃等数十种功能。
         
          只需要其中的某一种功能就可以使用这。即使不用分布式，也可以使用python内置queue对象。
          这就是给函数添加几十项控制的超级装饰器。是快速写代码的生产力保障。

  ```
  
  #### 5.10 怎么引入使用这个框架？
   ```
    答：先写自己的函数（类）来实现业务逻辑需求，不需要思考怎么导入框架。
        写好函数后把 函数和队列名字绑定传给消费框架就可以了。一行代码就能启动分布式消费。
        RedisConsmer('queue_name',consuming_function=your_function).start_consuming_message()
        所以即使你不想用这个框架了，你写的your_function函数代码并没有作废。
        所以不管是引入这个框架 、废弃使用这个框架、 换成celery框架，你项目的99%行 的业务代码都还是有用的，并没有成为废物。
  ```
   
  
  #### 5.11 怎么写框架？
   ```
    答：需要学习oop和设计模式。
        如果有杠精不信不服这句话的，你觉得可以使用纯函数编程，使用0个类来实现这样的框架。
        
        如果完全不理会设计模式，实现threding gevent evenlet 3种并发模式，加上9种中间件类型，实现分布式消费流程，
        需要反复复制粘贴扣字27次。代码绝对比你这个多。例如基于nsq消息队列实现任务队列框架，加空格只用了80行。如果完全反对oop，
        需要多复制好几千行来实现。
        
        包括完成7种中间件和3种并发模式，并且预留消息中间件的扩展。
        
        然后来和此框架 比较 实现框架难度上、 实现框架的代码行数上、 用户调用的难度上 这些方面。
  ```
   

  