### 前言

解决功能新问题：JAVA  jsp  rdsms  tomcat  html  linux  jdbc  svn
解决发展性问题：strtus  spring  springmvc  hibernate  mybatis
解决性能问题：NOSQL  java 线程  hadoop  nginx  mq  elasticsearch

### 1.NoSQL

NoSQL特点

非关系型数据库，不依赖业务逻辑数据库存储，以简单key-value存储

适用于

- 高并发读写
- 海量数据读写
- 数据可扩展

不适用于 

- 事务存储 以及复杂数据库



NoSQL优点

1. 缓存数据库，完全在内存中，速度快，数据结构简单

2. 减少io操作，数据库和表拆分，虽然破坏业务逻辑，即外加一个缓存数据库，提高数据库速度，也可以用专门的存储方式，以及针对不同的数据结构存储

### 2.Redis 

默认6379端口号

与memcache三点不同，支持多数据类型，持久化，单线程+多路io口复用

高速缓存 可持久化 开源key-value存储系统 支持多个类型集合 不同方式的排序 实现主从操作等

#### 软件安装

```shell
sudo apt update
```

```shell
sudo apt install redis-server
```

```shell
sudo systemctl status redis-server

Output:
● redis-server.service - Advanced key-value store
   Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-11-28 14:15:23 PST; 27s ago
     Docs: http:# redis.io/documentation,
           man:redis-server(1)
 Main PID: 2024 (redis-server)
    Tasks: 4 (limit: 2359)
   Memory: 6.9M
   CGroup: /system.slice/redis-server.service
           └─2024 /usr/bin/redis-server 127.0.0.1:6379

```

```shell
# 修改配置文件，开启远程访问
sudo vim /etc/redis/redis.conf


# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#注释掉此行
# bind 127.0.0.1 ::1

# 修改此配置为 no
protected-mode no
```

```shell
# 修改完配置后重启redis服务
sudo systemctl restart redis-server
```

```shell
# 查看服务端口情况
ss -an | grep 6379
# or
ps -ef | grep redis
```



#### 配置文件详解

```shell
# 给本地远程注释，可以远程
# bind 127.0.0.1 -::1 # 给本地远程注释，可以远程

# 保护模式改为no，可以远程访问
protected-mode no 

# 后台启动为yes
daemonize yes 

# 保存进程号的路径
pidfile /var/run/redis_6379.pid 
```

访问密码的查看和取消

```shell
# 将其注释去掉
requirepass foobared
```

或者使用命令行

```shell
config set requirepass "foobared"

auth foobared
```

设置之后每一次redis-cli进入的时候

还需要敲auth 123456才可进行访问



### 3.Redis 数据类型

**key值键位**

key值的操作：

```shell
# 查看当前库所有key
keys *

# 设置key值与value
set key value 

# 判断key是否存在
exists key 

# 查看key是什么类型
type key 

# 删除指定的key数据
del key 

# 根据value选择非阻塞删除
unlink key 

------仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作。

# 为给定的key设置过期时间
expire key 10 10秒钟：

# 查看还有多少秒过期，-1表示永不过期，-2表示已过期
ttl key 
```



库的选择：

```shell
# 命令切换数据库
select

# 查看当前数据库的key数量
dbsize

# 清空当前库
flushdb 

# 通杀全部库
flushall 
```



#### 3.1 string字符串



一个key对应一个value

二进制安全的，即可包含任何数据

value最多可以是512m



**基本命令：**

```shell
# 设置key值
set key value 

# 查询key值
get key 

# 将给定的value追加到原值末尾
append key value 

# 获取值的长度
strlen key 

# 只有在key不存在的时候，设置key值
setnx key value 

# 将key值存储的数字增1，只对数字值操作，如果为空，新增值为1
incr key 

# 将key值存储的数字减1，只对数字值操作，如果为空，新增值为1
decr key 

# 将key值存储的数字减1
decr key 

# 将key值存储的数字增减如步长
incrby/decrby key <步长> 
```

补充：

原子操作 不会被打断，从开始到结束

单线程不会被打断

多线程很难说，被打断的就不是原子操作



补充额外的字符串参数：

```shell
# 同时设置一个或者多个key-value
mset key value key value…

# 同时获取一个或多个value
mget key key…

# 同时设置一个或者多个key-value.当且仅当所有给定key都不存在
msetnx key value key value…

# 获取key的起始位置和结束位置的值
getrange key <起始位置> <结束位置> 

# 将value的值覆盖起始位置开始
setrange key <起始位置> value 

# 设置键值的同时,设置过期时间
setex key <> value 

# 用新值换旧值
getset key value 
```



#### 3.2 list列表

常用命令:

```shell
# 从左或者右插入一个或者多个值(头插与尾插)
lpush/rpush key value value…

# 从左或者右吐出一个或者多个值(值在键在,值都没,键都没)
lpop/rpop key 

# 从key1列表右边吐出一个值,插入到key2的左边
rpoplpush key1 key2 

# 按照索引下标获取元素(从左到右)
lrange key start stop 

# 获取所有值
lrange key 0 -1 

# 按照索引下标获得元素
lindex key index 

# 获取列表长度
llen key 

# 在value的前面插入一个新值
linsert key before/after value newvalue 

# 从左边删除n个value值
lrem key n value 

# 在列表key中的下标index中修改值value
lset key index value 
```



#### 3.3 set集合

字典，哈希表
自动排重且为无序的
常用命令:

```shell
# 将一个或者多个member元素加入集合key中,已经存在的member元素被忽略
sadd key value value… 

# 取出该集合的所有值
smembers key 

# 判断该集合key是否含有改值
sismember key value 

# 返回该集合的元素个数
scard key 

# 删除集合中的某个元素
srem key value value 

# 随机从集合中取出一个元素
spop key 

# 随即从该集合中取出n个值，不会从集合中删除
srandmember key n 

# 将一个集合a的某个value移动到另一个集合b
smove <一个集合a><一个集合b>value 

# 返回两个集合的交集元素
sinter key1 key2 

# 返回两个集合的并集元素
sunion key1 key2 

# 返回两个集合的差集元素（key1有的，key2没有）
sdiff key1 key2 
```



#### 3.4 hash哈希

键值对集合，特别适合用于存储对象类型
常用命令：

```shell
# 给key集合中的filed键赋值value
hset key field value 

# 集合field取出value
hget key1 field 

# 批量设置hash的值
hmset key1 field1 value1 field2 value2 

# 查看哈希表key中，给定域field是否存在
hexists key1 field 

# 列出该hash集合的所有field
hkeys key 

# 列出该hash集合的所有value
hvals key 

# 为哈希表key中的域field的值加上增量1 -1
hincrby key field increment 

# 将哈希表key中的域field的值设置为value，当且仅当域field不存在
hsetnx key field value 
```



#### 3.5 Zset有序集合

没有重复元素的字符串集合，按照相关的分数进行排名
常用命令：

```shell
# 将一个或多个member元素及其score值加入到有序key中
zadd key score1 value1 score2 value2 

# 返回有序集key，下标在start与stop之间的元素，带withscores，可以让分数一起和值返回到结果集。
zrange key start stop (withscores) 

# 返回有序集key，所有score值介于min和max之间（包括等于min或max）的成员。有序集成员按score的值递增次序排列
zrangebyscore key min max(withscores) 

# 同上，改为从大到小排列
zrevrangebyscore key max min （withscores）

# 为元素的score加上增量
zincrby key increment value 

# 删除该集合下，指定值的元素
zrem key value 

# 统计该集合，分数区间内的元素个数
zcount key min max 

# 返回该值在集合中的排名，从0开始
zrank key value 
```



#### 3.6 Bitmaps

1.合理使用操作位可以有效地提高内存使用率和开发使用率
2.本身是一个字符串，不是数据类型，数组的每个单元只能存放0和1，数组的下标在Bitmaps叫做偏移量
3.节省空间，一般存储活跃用户比较多

命令参数：

**1.设置值**

```shell
setbit key offset value
```

> 第一次初始化bitmaps，如果偏移量比较大，那么整个初始化过程执行会比较慢，还可能会造成redis的堵塞

**2.getbit取值**

```shell
getbit key offset 
```

获取bitmaps中某个偏移量的值
获取键的第offset位的值

**3.bitcount 统计数值**

```shell
bitcount key （start end）
```

统计字符串从start 到end字节比特值为1的数量

redis的setbit设置或清除的是bit位置，而bitcount计算的是byte的位置

**4.bitop**

复合操作，交并非异或，结果保存在destkey

    bitop and(or/not/xor）destkey key

这个key放两个key1 key2



#### 3.7 HyperLogLog

1.统计网页中页面访问量
2.只会根据输入元素来计算基数，而不会储存输入元素本身，不能像集合那样，返回输入的各个元素
3.基数估计是在误差可接受的范围内，快速计算（不重复元素的结算）

命令参数：
1.添加指定的元素到hyperloglog中

```shell
pfdd key element
```

列如 pfdd progame “java”
成功则返回1，不成功返回0

2.计算key的近似基数

```shell
pfcount key 
```

即这个key的键位添加了多少个不重复元素

3.一个或多个key合并后的结果存在另一个key

```shell
pfmerge destkey sourcekey sourcekey
```



#### 3.8 Geographic

提供经纬度设置，查询范围，距离查询等

命令参数：
1.添加地理位置（经度纬度名称）
当坐标超出指定的范围，命令会返回一个错误
已经添加的数据，无法再添加

```shell
geoadd key longitude latitude member
```

例如 geoadd china:city 121.47 31.23 shanghai

2.获取指定地区的坐标值

```shell
geopos key member 
```

例如 geopos china:city shanghai

3.获取两个位置之间的直线距离

```shell
geodist key member1 member2 (m km ft mi)
```

4.以给定的经纬度为中心，找出某一半径的内元素

```shell
georadius key longitude latitude radius (m km ft mi)
```



### 4.发布和订阅

发布和订阅消息

客户端可以订阅任意频道的消息

打开一个客户端进行订阅

```shell
subscribe channel1
```

打开另一个客户端进行发布

```shell
publish channel1 hello
```





### 5. Jedis操作Redis

在xml中配置

```xml
<!-- https:# mvnrepository.com/artifact/redis.clients/jedis -->
    <dependencies>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.6.1</version>
        </dependency>
    </dependencies>
```

在类中定义

```java
import redis.clients.jedis.Jedis;

public class jedis {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);

    	jedis.auth("123456");
    	System.out.println(jedis.getClient().getPort());
    	System.out.println("连接本地的Redis服务器成功");
    	# 查看服务是否运行
    	System.out.println("服务正在运行：" + jedis.ping());
	}
}
```



测试代码：

```java
public class TestJedisConnection {

    Jedis jedis = null;

    /**
     * 以下测试之前先执行与redis的连接
     */
    @Before
    public void before(){
        jedis = new Jedis("ip",port);
        	jedis.auth("password")
        String value = jedis.ping();
        System.out.println("测试是否已连接到redis数据-->"+value);
    }

    @Test
    public void testGeneric(){
        /**
         * 1、获取redis的所有key
         * keys *
         */
        Set<String> keys = jedis.keys("*");
        for (String key : keys) {
            System.out.println("key-->"+key);
        }

        /**
         * 2、查看是否存在key...
         * exists k1 k2
         */
        Long exists = jedis.exists("k1", "k2");
        System.out.println("存在的key数量"+exists);


        /**
         * 3、设置过期时间
         * expire key time
         */
        Long k11 = jedis.expire("k1", 60);
        System.out.println(k11);

        /**
         * 4、查看过期时间
         * ttl key
         **/
        Long k12 = jedis.ttl("k1");
        System.out.println("k1的过期时间-->"+k12);
    }


    @Test
    public void test01(){

        /**
         * 1、设置key value
         * set key value
         */
        String set = jedis.set("k1", "v1");
        System.out.println(set);

        /**
         * 2、获取key对应的value
         * get key
         */
        String k1 = jedis.get("k1");
        System.out.println(k1);

        /**
         * 3、批量插入多个key value
         * mset k1 v1 k2 v2 k3 v3
         */
        String mset = jedis.mset("k1", "001", "k2", "v002", "k3", "v003");
        System.out.println("批量插入key value -->"+mset);

        /**
         * 4、批量获取多个value
         * mget k1 k2 k3
         */
        List<String> mget = jedis.mget("k1", "k2", "k3");
        for (String s : mget) {
            System.out.println("批量获取value-->"+s);
        }
    }
```



