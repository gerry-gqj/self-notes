# RabbitMQ

> 声明：本文的命令是在Win10系统，内容基本按照官网+谷歌浏览器进行学习RabbitMQ(兔子消息队列？).
>
> 官网地址：[www.rabbitmq.com/](https://link.juejin.cn?target=https%3A%2F%2Fwww.rabbitmq.com%2F)
>
> 参考3y的什么是消息队列：[github.com/ZhongFuChen…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FZhongFuCheng3y%2F3y)
>
> 注意：本文是学习教程，是对基础的使用和参数说明，真正线上使用不止这些配置，后续会进行线上使用更新。

## 安装

> 按照官网瞎摸索，瞎子过程就不说了。。。需要安装客户端软件(下载地址所示)，以及开启服务软件在下载文档说明（如图哪个）
>
> 速度慢可以考虑使用迅雷下载，杠杠的。

下载地址：[erlang.org/download/ot…](https://link.juejin.cn?target=https%3A%2F%2Ferlang.org%2Fdownload%2Fotp_versions_tree.html)

下载文档说明：[www.rabbitmq.com/install-win…](https://link.juejin.cn?target=https%3A%2F%2Fwww.rabbitmq.com%2Finstall-windows.html%23installer)

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/15/172b842aeb09abc1~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

结果：(要下载两个安装)

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/15/172b841cd1ccaa63~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

## 介绍

MQ即是(Message Queue消息队列)，对于队列不陌生吧，就是具有先进先出的特性容器，那消息呢？是指要发送的内容，要提醒其他人的一条信息。

官网说明：（图片翻译：来源谷歌浏览器，注意下面翻译斑点应为文件）

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/15/172b843bc02855cc~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

消息传递通常使用的一些术语：

- 生产者：就是发送消息的程序

- 消费者：就是接收消息的程序

- 队列：就是给要发送的消息进行缓存（给消息进行排排队，咱们消息一个一个来）

  简单形容过程就是生产者把消息给到队列，消费者从队列取出消息。但是这个框架还提供了许多强大的功能，比如保证消息处理成功、通知特定的消费者进行接收等等。

  

## Hello World

需要事先安装好客户端软件和服务软件。建立一个Maven项目，添加RabbitMQ依赖。

maven搜索网站，直接搜索就有依赖：[mvnrepository.com/](https://link.juejin.cn?target=https%3A%2F%2Fmvnrepository.com%2F)

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/15/172b843219e46d25~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

启动安装好的RabbitMQ Service start， 提示》》》请求的服务已经启动。（闪退的话尝试管理员启动）

终于准备好了，可以上代码了！！！！！官网第一个简单的hello world







```java
package top.codexu.rabbitmq.hello;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.nio.charset.StandardCharsets;

/**
 * 发送类
 *
 * @author codeXu
 */
public class Send {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        // 采用工厂模式
        ConnectionFactory factory = new ConnectionFactory();
        // 设置主机，因是本地，故不用配置其他属性。
        factory.setHost("localhost");
        // try-with-resources语句：括号里进行嵌套字连接，使用工厂得到连接类，再建立一个通道。
        // try-with-resources特性是在括号里的资源不用手动finally进行关闭，前提是资源实现了java.lang.AutoCloseable或java.io.Closeable的类
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            // 声明一个队列：指定名字、是否持久、是否仅此连接、是否自动删除、队列的其他属性
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World!";
            // 进行发布消息，指定交易所、队列名、消息的其他属性、消息体
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
            // 打印看看是否发送了
            System.out.println(" [x] 发送 '" + message + "'");
        }
    }
}
```





```java
package top.codexu.rabbitmq.hello;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

/**
 * 接收类
 *
 * @author codeXu
 */
public class Recv {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] 等待消息.");

        // 声明一个回调，进行处理得到的队列消息。
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] 接收 '" + message + "'");
        };

        // 队列名、 如果服务器应考虑消息传递后已确认，则为true、传递回调、取消回调
        // 消费开启：一直保持着，得到了队列内容，就执行回调函数
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });

    }

}
```



先运行消费者或生产者都无所谓，因消费者无明确关闭资源，会一直保持运行，故没使用try-with-resources。

结果能够发送和接收说明运行成功。

### 注意

若要查看本机上的队列，可以移动到对应的工具路径下：安装rabbitmq-service的目录下的\rabbitmq_server-3.8.4\sbin。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/15/172b851928d323e1~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

在对应的路径下运行rabbitmqctl.bat list_queues

显示结果（按照表格形式）有个name（列头）说明是哪个队列，message(列头)说明这个队列目前存在多少个消息。



## 工作队列模式

通过上面的hello world已经大致知道了这个调用的过程，咱们要深入的学习，了解清楚它的特性，继续学习！！！

> 工作队列是啥？就是当遇到了运行耗时久的任务，并且还得等待它完成，这个时候就可以使用工作队列，把这个耗时任务发送给别的工人(消费者)进行处理，生产者可以直接得到处理完的情况。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/15/172b852541812c4f~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

将要说到的特性：消息确认、消息持久、公平派遣

- 消息确认：即是消费者处理完了，发送个通知告诉MQ我处理完成了
- 消息持久：就是遇到了宕机后，会把消息给缓存或保存下来，使得下次启动能够不丢失，但不是百分百。
  - 若要强保证使用发布者确认：[www.rabbitmq.com/confirms.ht…](https://link.juejin.cn?target=https%3A%2F%2Fwww.rabbitmq.com%2Fconfirms.html)
- 循环调度：若开着两个或三个消费者的时候，当多个消息要接收，MQ是会自动循环找下一个，避免一直重复同一个或几个。
- 公平派遣：当有两个消费者的时候，若一个消费者一直再累死累活，另外一个逍遥自在，这是不利于效率提升的，故可以通过设置，限制若A忙就找B去。

咱们还是直接代码说话，注释写详细点。（java代码写的好，很多可以见名知意）







```java
package top.codexu.rabbitmq.work;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

import java.nio.charset.StandardCharsets;

/**
 * @author codeXu
 */
public class NewTask {
    private final static String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            // 声明一个队列：指定名字、是否持久、是否仅此连接、是否自动删除、队列的其他属性
            // 消息持久化前提：这个队列第一次声明的时候就是为持久的，并且在生产者端和消费者端都要声明为true。(第二个参数) 注意：这个持久不是百分百保证。
            channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

            // 循环5发送5次消息
            for (int i = 0; i <= 5; i++) {
                String message = "第" + i + "个消息.";
                // 进行发布消息，指定交易所、队列名、消息的其他属性、消息体
                channel.basicPublish("", TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes(StandardCharsets.UTF_8));
                System.out.println(" [x] 发送 '" + message + "'");
            }
        }
    }
}
```









```java
package top.codexu.rabbitmq.work;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

/**
 * @author codeXu
 */
public class Worker {

    private final static String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();
		// 这里也要为true
        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
        System.out.println(" [*] 等待消息.");

        // 公平派遣设置：一次仅接受一条未经确认的消息，为了实现公平的安排(其他人闲着就安排其他人)
        channel.basicQos(1);

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] 接收 '" + message + "'");
            
            // 假设执行一个耗时的任务，使用等待举例。
            try {
                doWork(message);
            } catch (InterruptedException e) {
                e.getMessage();
            } finally {
                System.out.println(" [x] 结束");
                // 执行完成，返回确认消息，若不返回确认消息(下面这句)，MQ就会认为这个消息没执行。
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        // 消息确认：设置false(默认状态)为需要消费者发送回确认。若没返回确认，这个消息就不算被处理过，就把这个消息从新在MQ上排队。
        // 目的就是为了不丢失消息，使得确保每个消息有执行成功。(网络抖动也不怕，会找别人执行)
        boolean autoAck = false;
        channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });

    }

    private static void doWork(String task) throws InterruptedException {
        for (char ch : task.toCharArray()) {
            if (ch == '.') {
                Thread.sleep(10000);
            }
        }
    }
}
```



若通过系统要查看当前队列有那些消息还未确定的，可以通过以下命令：

\> rabbitmqctl.bat list_queues name messages_ready messages_unacknowledged

运行结果如下：

```shell
Timeout: 60.0 seconds ...
Listing queues for vhost / ...
name    messages_ready  messages_unacknowledged
work    0       3
hello   0       0
```



name：队列名，messages_ready等待接收的消息， messages_unacknowledged未返回成功的消息(已被消费者处理过或正在处理)

## 发布/订阅

构造各种队列的参数说明：[www.rabbitmq.com/queues.html](https://link.juejin.cn?target=https%3A%2F%2Fwww.rabbitmq.com%2Fqueues.html)

> 发布/订阅：即是一个生产者要发布消息，然后有订阅它的都能得到这个消息。实现和上面的差别不大，加了一个交换所再中间。咱们还是直接上代码。



```java
package top.codexu.rabbitmq.publicsubscribe;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * 发射日记
 *
 * @author codeXu
 */
public class EmitLog {

    /** 交互所的名字 */
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            // 采用交易所的模式，无须声明队列，直接声明交互所。
            channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

            String message = "info: Hello World!";

            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            System.out.println(" [x] 发送 '" + message + "'");
        }
    }
}
```





```java
package top.codexu.rabbitmq.publicsubscribe;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

/**
 * 接收日志类
 *
 * @author codeXu
 */
public class ReceiveLogs {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        // 交换所声明，类型是fanout（扇出）
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        String queueName = channel.queueDeclare().getQueue();
        // 队列绑定到对应的交换所
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println(" [*] 等待消息.");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] 接收 '" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }
}
```



测试结果：启动多个接收日志，然后再启动发射类，即可看见结果是多个接收日志都有响应。



查看队列绑定的交换所：

> rabbitmqctl list_bindings

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/15/172b856e57e191ca~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

存在一个logs交换所，类型exchange，目的名：XXX ，目的的类是的队列。



## 路由

> 对比发布订阅模式，路由是可以指定匹配的才接收。
>
> 举例(举例是很好的学习方式)：现在情况如下，一个发布者对照多个接收者，作为发布者：我只想把内容发布出去，并给个等级它，交换所你帮我处理。作为接收者：我只能得到发布者的最高等级的消息，其他的我要来也没用。
>
> 上面的情况如果使用订阅发布模式是不能区分开等级的。故咱们使用接下要讲的路由模式。

等级是为了举例而使用的，现在咱们官方说法是路由键，就是一串字符，好比咱们下面要讲的例子，可以是"info"  "warning" "error" "red" "blue"等等字符串，就是为了匹配不同的队列。路由键 == 事先商量好的匹配字符串

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/15/172b8572ab01c24c~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

咱们用官网的图说话：发送者p把消息交给交易所X，然后发送的级别是info的话，就只有C2接收到了，发送error等级的话就两者可以收到。

上面已经讲清楚了路由的模式，现在咱们来看看实现，使用路由键匹配模式需要声明交换所的类型为直接(direct)类型。上代码，走起！！！







```java
package top.codexu.rabbitmq.routing;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class EmitLogDirect {

    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            // 注意：声明类型为直接交互类型(direct)
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
            
            // 发送的路由键为warning，内容是Hello World!  路由键可以自己换
            String severity = "warning";
            String message = "Hello World!";
            
            channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + severity + "':'" + message + "'");
        }
    }

}
```









```java
package top.codexu.rabbitmq.routing;

import com.rabbitmq.client.*;

public class ReceiveLogsDirect {

    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        // 咱们得到两个队列
        String queueName1 = channel.queueDeclare().getQueue();
        String queueName2 = channel.queueDeclare().getQueue();

        //队列1绑定多个路由键
        String[] severities1 = {"info", "warning", "error"};
        for (String severity : severities1) {
            channel.queueBind(queueName1, EXCHANGE_NAME, severity);
        }

        // 队列2只绑定一个info
        String[] severities2 = {"info"};
        for (String severity : severities2) {
            channel.queueBind(queueName2, EXCHANGE_NAME, severity);
        }
        System.out.println(" [*] 等待消息.");

        // 队列1进行接收响应
        DeliverCallback deliverCallback1 = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] 队列1接收到 '" + delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };
        channel.basicConsume(queueName1, true, deliverCallback1, consumerTag -> { });

        // 队列2进行接收响应
        DeliverCallback deliverCallback2 = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] 队列2接收到 '" + delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };
        channel.basicConsume(queueName2, true, deliverCallback2, consumerTag -> { });
    }
}
```



测试结果：发送info等级的话，两个消费者都可以得到，发送warning、error等级的只有消费者1可以得到



## 主题

主题类型就是在路由的升级一下，就是高级匹配。

现在咱们来仔细说明一下！！！

首先，咱们发送端的路由键改为：info.black.ok             为什么要改呢？是为了说明主题的作用！！！

要想匹配上面的路由键，接收端咱们可以这样写：info.*.*  或 .*.*ok 这样等等。是不是"*"就是全匹配字符！！！

接收端还能这样写：info.#   

咱们说明一下"*"和"#"的作用：

- ("*") 可以代替一个单词
- ("#") 可以代替零个或多个单词

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/15/172b8580fcaf9eaf~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

Q1和Q2队列的匹配路由键如上图所示

- 队列1表明对所有为橙色的感兴趣
- 队列2表明对兔子以及懒惰的感兴趣

咱们来举例发送端的发送情况，看看匹配那些队列。

| 发送端的路由键           | 匹配的队列 | 备注                 |
| ------------------------ | ---------- | -------------------- |
| quick.orange.rabbit      | Q1和Q2     | Q1：orange，Q2：lazy |
| lazy.orange.elephant     | Q1和Q2     | Q1：orange，Q2：lazy |
| quick.orange.fox         | Q1         |                      |
| lazy.brown.fox           | Q2         |                      |
| lazy.pink.rabbit         | Q2         |                      |
| quick.brown.fox          | 无         |                      |
| quick.orange.male.rabbit | 无         | 队列1匹配的是3个.    |
| lazy.orange.male.rabbit  | Q2         | #的作用是多个或零个  |



## 结尾

本文涉及到的交换所内置的交互类型：

- DIRECT("direct") 扇出：特性是广播，在这个交换所的所有接收队列都会收到发送者的消息。
- FANOUT("fanout") 直接：特性是 路由匹配。
- TOPIC("topic") 主题：特性是路由匹配的升级，直接是一个字符串的，主题可以为指定前缀类型。
- HEADERS("headers"); headers 匹配 AMQP 消息的 header 而不是路由键，听说几乎不用了 故博主未了解。

都看到这里了，还不点赞收藏一下吗？谢谢支持



唠叨唠叨，第一次正式的写博文，感觉掘金体验不是非常好。







# spring-boot-rabbitmq使用说明

## 基本概念

queue：队列，每个队列可以有多个消费者，但是一条消息只会被一个消费者消费

exchange:交换机，队列可以绑定交换机，交换机根据路由或者其他匹配信息将消息发送至queue

## 模式介绍

simple模式：不需要交换机，直连模式。一个队列只有一个消费者

work模式：一个队列多个消费者

direct模式：需要交换机，通过交换机的路由key，精确匹配queue，并发送至对应的queue

topic模式：通过路由与路由key，模糊匹配的模式。可用通配符。比如key.1会被绑定路由key.*的queue获取到

fanout: 广播模式，不需要路由key，给所有绑定到交换机的queue

## spring-boot-rabbit

## maven依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

## yml配置

### 简单使用

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1 #ip
    port: 5672      #端口
    username: guest #账号
    password: guest #密码
```

### 全量配置说明

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1 #ip
    port: 5672      #端口
    username: guest #账号
    password: guest #密码
    virtualHost:    #链接的虚拟主机
    addresses: 127.0.0.1:5672     #多个以逗号分隔，与host功能一样。
    requestedHeartbeat: 60 #指定心跳超时，单位秒，0为不指定；默认60s
    publisherConfirms: true  #发布确认机制是否启用
    publisherReturns: #发布返回是否启用
    connectionTimeout: #链接超时。单位ms。0表示无穷大不超时
    ### ssl相关
    ssl:
      enabled: #是否支持ssl
      keyStore: #指定持有SSL certificate的key store的路径
      keyStoreType: #key store类型 默认PKCS12
      keyStorePassword: #指定访问key store的密码
      trustStore: #指定持有SSL certificates的Trust store
      trustStoreType: #默认JKS
      trustStorePassword: #访问密码
      algorithm: #ssl使用的算法，例如，TLSv1.1
      verifyHostname: #是否开启hostname验证
    ### cache相关
    cache:
      channel: 
        size: #缓存中保持的channel数量
        checkoutTimeout: #当缓存数量被设置时，从缓存中获取一个channel的超时时间，单位毫秒；如果为0，则总是创建一个新channel
      connection:
        mode: #连接工厂缓存模式：CHANNEL 和 CONNECTION
        size: #缓存的连接数，只有是CONNECTION模式时生效
    ### listener
    listener:
       type: #两种类型，SIMPLE，DIRECT
       ## simple类型
       simple:
         concurrency: #最小消费者数量
         maxConcurrency: #最大的消费者数量
         transactionSize: #指定一个事务处理的消息数量，最好是小于等于prefetch的数量
         missingQueuesFatal: #是否停止容器当容器中的队列不可用
         ## 与direct相同配置部分
         autoStartup: #是否自动启动容器
         acknowledgeMode: #表示消息确认方式，其有三种配置方式，分别是none、manual和auto；默认auto
         prefetch: #指定一个请求能处理多少个消息，如果有事务的话，必须大于等于transaction数量
         defaultRequeueRejected: #决定被拒绝的消息是否重新入队；默认是true（与参数acknowledge-mode有关系）
         idleEventInterval: #container events发布频率，单位ms
         ##重试机制
         retry: 
           stateless: #有无状态
           enabled:  #是否开启
           maxAttempts: #最大重试次数,默认3
           initialInterval: #重试间隔
           multiplier: #对于上一次重试的乘数
           maxInterval: #最大重试时间间隔
       direct:
         consumersPerQueue: #每个队列消费者数量
         missingQueuesFatal:
         #...其余配置看上方公共配置
     ## template相关
     template:
       mandatory: #是否启用强制信息；默认false
       receiveTimeout: #`receive()`接收方法超时时间
       replyTimeout: #`sendAndReceive()`超时时间
       exchange: #默认的交换机
       routingKey: #默认的路由
       defaultReceiveQueue: #默认的接收队列
       ## retry重试相关
       retry: 
         enabled: #是否开启
         maxAttempts: #最大重试次数
         initialInterval: #重试间隔
         multiplier: #失败间隔乘数
         maxInterval: #最大间隔
```

## 配置queue，exchange以及绑定关系

*****该配置发送端不需配置。接收端可选配置，非必须。

```java
// 队列名
public static final String FANOUT_QUEUE_NAME = "fanout_queue";
//交换机名
public static final String TEST_FANOUT_EXCHANGE = "fanout_exchange";

public static final String DIRECT_QUEUE_NAME = "direct_queue";
public static final String TEST_DIRECT_EXCHANGE = "direct_exchange";
public static final String DIRECT_ROUTINGKEY = "test";

// 创建队列
@Bean
public Queue createFanoutQueue() {
    return new Queue(FANOUT_QUEUE_NAME);
@Bean
public Queue createDirectQueue() {
    return new Queue(DIRECT_QUEUE_NAME);
}
// 创建交换机
@Bean
public FanoutExchange defFanoutExchange() {
     return new FanoutExchange(TEST_FANOUT_EXCHANGE);
}
@Bean
DirectExchange directExchange() {
    return new DirectExchange(TEST_DIRECT_EXCHANGE);
}
// 队列与交换机进行绑定
@Bean
Binding bindingFanout() {
   return BindingBuilder.bind(createFanoutQueue()).to(defFanoutExchange());
}
//队列与交换机绑定并添加路由key（direct和topic模式）
@Bean
Binding bindingDirect() {
    return BindingBuilder.bind(createDirectQueue()).to(directExchange()).with(DIRECT_ROUTINGKEY);
}
```

## 生产者

### simple模式/work模式

不需要交换机，直接发送至queue.

```java
//AqmpTemplate功能一样，也可以用
    @Autowired
    private RabbitTemplate template;

    @Test
     public void sendSimple() {
     String context = "simple---> " + new Date();
     //如果没有配置默认交换机,直接传入queue的name
     template.convertAndSend("ly_simple", context);
     //如果配置了默认的交换机，（交换机，queue_name，内容）
     template.convertAndSend("","""ly_simple", context);
     }
```

### direct模式/topic模式等需要交换机

```java
@Test
    public void sendDirect() {
        String context = "direct---> " + new Date();
        //（交换机名称,路由的key,内容）
        template.convertAndSend("ly_direct", "ly", context);
    }
```

## 消费者

### 注解使用

```java
//基础注解，指定queue的名称，可以多个。如果是simple不需要交换机的直接写队列名称就好。
    //如果是其他的也想只指定一个queues——name的话，需要上面的配置类配置queue或者其他绑定关系
    @RabbitListener(queues = "ly_simple")
    @RabbitHandler
    public void processSimpleMsg(String message) {
        System.out.println("########################received simple" + message);
    }

   //如果不想使用配置类，可以直接注解通过bindings，绑定，spring会根据注解生成绑定
   //ps：如果已有同名称的类。不会覆盖。会影响功能
    @RabbitListener(bindings = { @QueueBinding(value = @Queue(value = "ly_direct", durable = "true"),
            exchange = @Exchange(value = "ly_direct", type = "direct"), key = "ly") })
    @RabbitHandler
    public void processDirectMsg(String message) {
        System.out.println("########################received" + message);
    }

    @RabbitListener(bindings = { @QueueBinding(value = @Queue(value = "ly_fanout", durable = "true"),
            exchange = @Exchange(value = "ly_fanout", type = "fanout")) })
    @RabbitHandler
    public void processFanoutMsg(String message) {
        System.out.println("########################received" + message);
    }
```





# RabbitMQ：@RabbitListener 与 @RabbitHandler 及 消息序列化



- 添加 @RabbitListener 注解来指定某方法作为消息消费的方法，例如监听某 Queue 里面的消息

## MessageConvert

- 涉及网络传输的应用序列化不可避免，发送端以某种规则将消息转成 byte 数组进行发送，接收端则以约定的规则进行 byte[] 数组的解析
- RabbitMQ 的序列化是指 Message 的 body 属性，即我们真正需要传输的内容，**RabbitMQ 抽象出一个 MessageConvert 接口处理消息的序列化**，其实现有 SimpleMessageConverter（默认）、Jackson2JsonMessageConverter 等
- 当调用了 convertAndSend 方法时会使用 MessageConvert 进行消息的序列化
- **SimpleMessageConverter 对于要发送的消息体 body 为 byte[] 时不进行处理，如果是 String 则转成字节数组,如果是 Java 对象，则使用 jdk 序列化将消息转成字节数组，转出来的结果较大，含class类名，类相应方法等信息。因此性能较差**
- 当使用 RabbitMQ 作为中间件时，数据量比较大，此时就要考虑使用类似 Jackson2JsonMessageConverter 等序列化形式以此提高性能

## @RabbitListener 用法

- 使用 @RabbitListener 注解标记方法，当监听到队列 debug 中有消息时则会进行接收并处理



```csharp
@RabbitListener(queues = "debug")
public void processMessage1(Message bytes) {
    System.out.println(new String(bytes));
}
```

### 注意

- 消息处理方法参数是由 MessageConverter 转化，若使用自定义 MessageConverter 则需要在 RabbitListenerContainerFactory 实例中去设置（默认 Spring 使用的实现是 SimpleRabbitListenerContainerFactory）
- 消息的 content_type 属性表示消息 body 数据以什么数据格式存储，接收消息除了使用 Message 对象接收消息（包含消息属性等信息）之外，还可直接使用对应类型接收消息 body 内容，但若方法参数类型不正确会抛异常：
  - **application/octet-stream**：二进制字节数组存储，使用 byte[]
  - **application/x-java-serialized-object**：java 对象序列化格式存储，使用 Object、相应类型（反序列化时类型应该同包同名，否者会抛出找不到类异常）
  - **text/plain**：文本数据类型存储，使用 String
  - **application/json**：JSON 格式，使用 Object、相应类型



### 通过 @RabbitListener 注解声明 Binding

- 通过 @RabbitListener 的 bindings 属性声明 Binding（若 RabbitMQ 中不存在该绑定所需要的 Queue、Exchange、RouteKey 则自动创建，若存在则抛出异常）

```csharp
@RabbitListener(bindings = @QueueBinding(
        exchange = @Exchange(value = "topic.exchange",durable = "true",type = "topic"),
        value = @Queue(value = "consumer_queue",durable = "true"),
        key = "key.#"
))
public void processMessage1(Message message) {
    System.out.println(message);
}
```



## @RabbitListener 和 @RabbitHandler 搭配使用

- @RabbitListener 可以标注在类上面，需配合 @RabbitHandler 注解一起使用
- @RabbitListener 标注在类上面表示当有收到消息的时候，就交给 @RabbitHandler 的方法处理，具体使用哪个方法处理，根据 MessageConverter 转换后的参数类型

```java
@Component
@RabbitListener(queues = "consumer_queue")
public class Receiver {

    @RabbitHandler
    public void processMessage1(String message) {
        System.out.println(message);
    }

    @RabbitHandler
    public void processMessage2(byte[] message) {
        System.out.println(new String(message));
    }
    
}
```