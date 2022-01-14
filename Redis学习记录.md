[toc]

***

## 第一章

1. Redis介绍
   - 开源的、高性能的、基于K-V的缓存与存储系统
   - 胜任消息队列、任务队列等
2. 特性
   - 存储结构
     - 以字典K-V结构进行存储，同时支持通过TCP协议访问key值
     - 支持一下数据类型
       - 字符串类型
       - 散列类型
       - 列表类型
       - 集合类型
       - 有序集合类型
     - 相比MySQL等关系型数据库更为直观，更方便的操作方式
   - 内存存储与持久化、
     - 所有数据都存在内存中，速度远超硬盘 [^ 单机每秒10w]
     - 内存数据异步写入磁盘
   - 功能丰富
     - 可做缓存、队列等
     - 设置存货时间[^ TTL]
     - 限定数据占用最大内存空间[^ 自动淘汰]
     - 列表类型键实现队列，支持阻塞读取
     - “发布/订阅”消息模式
   - 简单稳定
     - 语句方便
		- 多种客户端库
		- C语言开发

## 第二章

1. Redis可执行文件说明

   | 文件名          | 说明 |
   | --------------- | :--- |
   | redis-server   | Redis服务器 |
   | redis-cli      | Redis命令行客户端 |
   | redis-benchmark | Redis性能测试工具 |
   | redis-check-aof|AOF文件修复工具|
   |redis-check-dump|RDB文件检查工具|
   |redis-sentinel|Sentinel服务器|
2. 多类型回复

   - 状态回复

     - redis> **PING**

       PONG

   - 错误回复

     - redis> **ERRORCOMMEND**

       (error) ERR unknown command 'ERRORCOMMEND'

   - 整数回复

     - **INCR foo**

       (integer) 1

   - 字符串回复

     - **GET foo**

       "1"

     - **GET noexistes**

       (nil)

   - 多行字符串回复

     - KEYS *

       "bar"

       "foo"

3. 多数据库

   - Redis提供了多个用来存储数据的字典，客户端可以指定将数据存储在那个字典当中。Redis默认支持16个。可以更改配置databases来修改这一数字

     - SELECT 1

       OK

       [1] > GET foo

       (nil)

   - 不支持自定义数据库名，都以编号命名

   - 不支持不同数据库不同密码，要么客户端都可以访问，要么都不能访问

## 第三章

1. 通用命令

   - KEYS pattern

     - 查看符合条件的key值，支持glob风格通配符(?,*,[],\x)

       **KEYS会遍历redis中的所有key，数量较多时会影响性能**

   - EXISTS key

     - 判断一个键是否存在，存在返回1，否则返回0

   - DEL key

     - 可以删除一个或者多个key，返回删除键的个数

   - TYPE key

     - 查看对应KEY的类型，返回值为redis的五种数据类型

2. 字符串类型

   - 赋值与取值

     - SET key value

     - GET key

       **当key不存在时返回空**

   - 递增数字

     - INCR key

       对不存在的key，创建key并自增，对非整数类型会报错

       redis的所有命令都是原子操作

   - 其他

     - key的命名规则    对象类型:对象ID:对象属性.···

     - ```mermaid
       graph LR
       A[获取自增ID] --> B(序列化) 
       B --> C[Set对应key的数据]
        D[存储数据]
       ```
       
     - ```mermaid
       graph LR
       A[Get对应key数据] --> B[反序列化] 
       B --> C[Incr对应ID]
       	获取数据
       ```
     
   - 其他命令

     - INCR key increment 增加指定的数字
     - DECR key [decrment] 自减1或者自减指定的数字
     - INCRBYFLOAT key incrment 增加指定浮点数
     - APPEND key value 向key尾部添加value并返回追加后value总长度
     - STRLEN key 获取字符串长度
     - MGET key1[value] key2 [value] ···· 获取/设置多个key的值

3. 散列类型
4. 列表类型
5. 集合类型
6. 有序集合类型

## 第四章

## 第五章

## 第六章

## 第七章

## 第八章

## 第九章

[^ 单机每秒10w]: Redis 在单个计算机上支持每秒获取10w的key
[^ TTL]: Time To Live , 生存时间到期后，会自动删除key
[^ 自动淘汰]: 在数据达到空间限制后，会根据一定的规则自动淘汰不需要的键
```

```