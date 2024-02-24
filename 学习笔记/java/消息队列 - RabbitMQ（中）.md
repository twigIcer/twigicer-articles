# 消息队列 - RabbitMQ（中）

> 本文分为两篇，上篇介绍重点消息队列的概念和RabbitMQ的安装及组件介绍，中篇为RabbitMQ的入门实践，下篇主要讲在Springboot中使用RabbitMQ及RabbitMQ的核心特性。

本来想一篇文章写完，但是发现RabbitMQ的模式比较多，写一起，文章太长了不太好，所以就分两篇了。

官方教程：[RabbitMQ Tutorials — RabbitMQ](https://www.rabbitmq.com/getstarted.html)

参考文章：[爆肝3万字，为你吃透RabbitMQ，最详细的RabbitMQ讲解（VIP典藏版） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/611247550)

参考鱼皮老哥视频：[主页 - 编程导航 (code-nav.cn)](https://www.code-nav.cn/)

## RabbitMQ快速入门：

第一步还是老样子，引依赖，注意是amqp的依赖，不是rabbitmq的依赖：

```xml
<!-- https://mvnrepository.com/artifact/com.rabbitmq/amqp-client -->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.20.0</version>
</dependency>
```

RabbitMQ 提供了多种模式，接下来依次演示每种模式。

### 1. Hello World (单向发送)

一个生产者给一个队列发消息，一个消费者从这个队列取消息。1 对 1。

官方教程：[RabbitMQ tutorial - "Hello world!" — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-one-python.html)

![image-20240219154719775](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240219154719775.png)

#### **1）生产者代码：**

直接复制官方代码：[rabbitmq-tutorials/java/Send.java at main · rabbitmq/rabbitmq-tutorials (github.com)](https://github.com/rabbitmq/rabbitmq-tutorials/blob/main/java/Send.java)

如果自定义了主机，密码，账户需要自己设置，这里使用初始账号，密码，端口。

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.nio.charset.StandardCharsets;

public class SingleProducer {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        // 创建连接工厂实例
        ConnectionFactory factory = new ConnectionFactory();
        // 设置 RabbitMQ 服务器的主机
        factory.setHost("localhost");
//        factory.setPort();
//        factory.setPassword();
//        factory.setUsername();
        // 使用 try-with-resources 语句创建连接和通道
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            // 声明队列，确保队列存在，如果不存在则创建
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            // 要发送的消息内容
            String message = "Hello World!";
            // 发布消息到指定的交换机（空字符串表示默认交换机）和路由键（使用 QUEUE_NAME 作为路由键）
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
            // 打印发送消息的确认信息
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

1. **连接工厂：**创建一个新实例，该实例用于配置与 RabbitMQ 服务器的连接。`ConnectionFactory`
2. **设置主机：**RabbitMQ 服务器的主机设置为“localhost”。如果 RabbitMQ 服务器在其他主机上运行，则应更新此值。
3. **连接和通道：**在 try-with-resources 块中，将创建一个新的连接和通道。try-with-resources 可确保 and 在使用后正确关闭。`Connection``Channel`
4. **队列声明：**该方法用于声明名为 的队列。这些参数指定队列的各种属性，例如队列是否持久、独占、自动删除和其他参数（在本例中为 ）。`queueDeclare``QUEUE_NAME``null`
5. **发布消息：**该方法用于将消息发布到指定的交换（默认交换的空字符串）和路由密钥 （）。使用 UTF-8 编码将消息转换为字节。`basicPublish``QUEUE_NAME`
6. **确认消息：**最后，将确认消息打印到控制台。

**Channel 频道**：理解为操作消息队列的 client（比如 jdbcClient、redisClient），提供了和消息队列 server 建立通信的传输方法（为了复用连接，提高传输效率）。程序通过 channel 操作 rabbitmq（收发消息）

**创建消息队列，channel.queueDeclare()方法参数：**

**queueName：**消息队列名称（注意，同名称的消息队列，只能用同样的参数创建一次）

**durabale：**消息队列重启后，消息是否丢失

**exclusive：**是否只允许当前这个创建消息队列的连接操作消息队列

**autoDelete：**没有人用队列后，是否要删除队列

执行代码：

![image-20240219155740197](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240219155740197.png)

可以看到有一条消息"hello"

![image-20240219155857836](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240219155857836.png)

#### 2）消费者代码：

直接复制官方代码：[rabbitmq-tutorials/java/Recv.java at main · rabbitmq/rabbitmq-tutorials (github.com)](https://github.com/rabbitmq/rabbitmq-tutorials/blob/main/java/Recv.java)

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

public class SingleConsumer {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        // 创建连接工厂实例
        ConnectionFactory factory = new ConnectionFactory();
        // 设置 RabbitMQ 服务器的主机
        factory.setHost("localhost");
        // 创建连接
        Connection connection = factory.newConnection();
        // 创建通道
        Channel channel = connection.createChannel();

        // 声明队列，确保队列存在，如果不存在则创建
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

       // 定义消息接收的回调函数
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            // 从消息中获取内容并以UTF-8编码转换为字符串
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
        };
        // 启动消费者并指定消息的接收回调，持续阻塞
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
    }
}
```

创建一个连接到RabbitMQ服务器的消费者，监听名为“hello”的队列。当队列中有新消息时，通过回调函数处理消息内容，并在控制台输出收到的消息。请注意，方法用于启动消息的消费过程，参数表示消息在消费后会被自动确认。

注意：消费者创建队列的名称还有参数需要和生产者一致！！

#### 3）测试：

启动消费者：

![image-20240219161307617](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240219161307617.png)

可以看到消息被消费了，且消费者个数为1：

![image-20240219161421584](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240219161421584.png)

消费者开启后，会自动消费队列里的消息。

## 2. Work Queues(工作队列)

一个生产者给一个队列发消息，多个消费者 从这个队列取消息。1 对多。

场景：多个机器同时去接受并处理任务（尤其是每个机器的处理能力有限）

官方教程：[RabbitMQ tutorial - Work Queues — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-two-java.html)

![image-20240219162147982](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240219162147982.png)

#### 1）生产者代码：

复制官方代码，通过Scanner改造，可以通过控制台输入发送消息：

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

import java.util.Scanner;

public class WorkProducer {
    private static final String TASK_QUEUE_NAME = "work_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        // 使用连接工厂创建一个新的 RabbitMQ 连接，并使用 try-with-resources 语句确保连接在使用完毕后正确关闭。
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            // 声明了一个名为 "work_queue" 的队列。
            channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNext()) {
                String message = scanner.nextLine();
                // 将用户输入的消息发布到名为 "work_queue" 的队列中。
                channel.basicPublish("", TASK_QUEUE_NAME,
                        MessageProperties.PERSISTENT_TEXT_PLAIN,
                        message.getBytes("UTF-8"));
                System.out.println(" [x] Sent '" + message + "'");
            }
        }
    }
}
```

其中，`channel.queueDeclare()` 方法是用于声明队列的方法。参数如下：

1. `String queue`：指定队列的名称。
2. `boolean durable`：指定队列是否是持久化的。如果设置为 `true`，则 RabbitMQ 会在服务器重启后保留该队列和其中的消息。如果设置为 `false`，则队列将在服务器重启后被删除。注意，持久化队列仅保证队列本身的持久化，不保证队列中的消息持久化，发送消息时需要设置相应的消息属性才能实现消息的持久化。
3. `boolean exclusive`：指定队列是否是独占的。如果设置为 `true`，则只有当前连接可以使用该队列。当连接关闭时，队列将被删除。
4. `boolean autoDelete`：指定队列是否是自动删除的。如果设置为 `true`，则当队列不再被使用时，即没有消费者订阅该队列，且没有未处理的消息，队列将被自动删除。
5. `Map<String, Object> arguments`：指定额外的参数。这个参数可以用来设置各种队列属性和行为。例如，可以设置队列的最大长度、最大优先级等等。

#### 2）消费者代码：

通过for循环创建两个消费者：

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class MultiConsumer {

    private static final String TASK_QUEUE_NAME = "multi_queue";

    public static void main(String[] argv) throws Exception {
        // 建立连接
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        final Connection connection = factory.newConnection();
        for (int i = 0; i < 2; i++) {
            final Channel channel = connection.createChannel();

            channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
            System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

            channel.basicQos(1);

            // 定义了如何处理消息
            int finalI = i;
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                String message = new String(delivery.getBody(), "UTF-8");

                try {
                    // 处理工作
                    System.out.println(" [x] Received '" + "编号:" + finalI + ":" + message + "'");
                    channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
                    // 停 20 秒，模拟机器处理能力有限
                    Thread.sleep(20000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, false);
                } finally {
                    System.out.println(" [x] Done");
                    //channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
                }
            };
            // 开启消费监听
            channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, consumerTag -> {
            });
        }
    }
}
```

注意：channel.basicQos(1)：控制单个消费者的处理任务积压数，每个消费者最多同时处理 1 个任务

调用 `basicQos(1)` 方法，设置每次从队列中获取的消息数量为 1，这样可以确保在处理完当前消息之前不会从队列中获取更多的消息。

#### 3）测试：

启动生产者，并连续发4条消息：

![image-20240220162431176](C:\Users\周斌\AppData\Roaming\Typora\typora-user-images\image-20240220162431176.png)

此时RabbitMQ创建了work_queue,，并有4条消息:

![image-20240220162525979](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220162525979.png)

此时，启动消费者，可以看到消费者1读取了消息1，消费者2读取了消息3：

![image-20240220162842479](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220162842479.png)

> 注意：这里并不说明两个消费者没有按顺序读取消息，而是因为开启了channel.basicQos(1)，消费者1挤压了1条消息。

先启动消费者，再启动生产者，连续发送4条消息，可以看到消费者按顺序消费：

![image-20240220171832997](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220171832997.png)

去掉channel.basicQos(1)，先启动生产者，发送4条消息，再启动消费者，可以看到只有消费者1消费：

![image-20240220171529182](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220171529182.png)

先启动消费者，再启动生产者，连续发送4条消息，消费者按顺序消费：

![image-20240220172116371](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220172116371.png)

> 说明两个消费者是按顺序轮询消费。
>
> 上面消费者代码写的有点问题，应该单独定义两个消费者，而不是用循环，本文不演示。
>
> 这里发现了一个问题，在finally中注释的那句加上，会出现重复确认导致消息丢失的问题，后面有时间会单独写一篇关于这个问题的文章。

#### 接下来的四种模式为 Exchange 交换机模型：分别为扇出(fanout)、直接(direct)、主题(topic)、标题(headers)，第四种模型（标题(headers)）因为效率低，不常使用，所以本文不做演示。

### 3. Publish/Subscribe（fanout扇出交换机/发布订阅）：

从名字就能看出，这种模式很适合做发布订阅，该模型特点为：消息会被转发到所有绑定到该交换机的队列。

官方教程：[RabbitMQ tutorial - Publish/Subscribe — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-three-java.html)

![image-20240220172456367](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220172456367.png)

#### 1）生产者代码：

和上面工作队列基本一致，但是该模型需要声明交换机：

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.Scanner;

public class FanoutProducer {

    private static final String EXCHANGE_NAME = "fanout-exchange";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            // 创建交换机
            channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNext()) {
                String message = scanner.nextLine();
                channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
                System.out.println(" [x] Sent '" + message + "'");
            }
        }
    }
}
```

通过 RabbitMQ 发布消息到一个广播型交换机，该交换机会将消息广播给所有与之绑定的队列。

#### 2）消费者代码：

创建两个队列，绑定到交换机上，编写两个回调函数，分别和两个队列绑定。

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class FanoutConsumer {
    private static final String EXCHANGE_NAME = "fanout-exchange";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel1 = connection.createChannel();
        Channel channel2 = connection.createChannel();
        // 声明交换机
        channel1.exchangeDeclare(EXCHANGE_NAME, "fanout");
        // 创建队列，随机分配一个队列名称
        String queueName = "xiaowang_queue";
        channel1.queueDeclare(queueName, true, false, false, null);
        channel1.queueBind(queueName, EXCHANGE_NAME, "");

        String queueName2 = "xiaoli_queue";
        channel2.queueDeclare(queueName2, true, false, false, null);
        channel2.queueBind(queueName2, EXCHANGE_NAME, "");
        channel2.queueBind(queueName2, EXCHANGE_NAME, "");

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback1 = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [小王] Received '" + message + "'");
        };

        DeliverCallback deliverCallback2 = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [小李] Received '" + message + "'");
        };
        channel1.basicConsume(queueName, true, deliverCallback1, consumerTag -> { });
        channel2.basicConsume(queueName2, true, deliverCallback2, consumerTag -> { });
    }
}
```

`channel.basicConsume()` 方法是用于启动消息消费者的方法，它有以下参数：

1. **queue**: 指定要消费的队列的名称。
2. **autoAck**: 一个布尔值，表示是否自动确认收到消息。如果设置为 `true`，表示一旦消息被接收，就立即确认。如果设置为 `false`，则需要手动调用 `channel.basicAck()` 方法来确认消息。
3. **deliverCallback**: 一个 `DeliverCallback` 接口的实现，用于定义接收到消息时的处理逻辑。这是一个函数式接口，可以通过 Lambda 表达式或其他方式实现。
4. **cancelCallback**: 一个 `CancelCallback` 接口的实现，用于定义消费者取消订阅时的处理逻辑。同样是一个函数式接口。
5. **consumerTag**: 一个字符串，用于标识消费者。通常可以设置为一个唯一的标识符。如果不需要关注消费者标识，可以传入 `""` 或者 `null`。

注意：

1消费者和生产者要绑定同一个交换机

2要先有队列，才能绑定

#### 3）测试：

先启动消费者，再启动生产者，并连续发送4条消息：

![image-20240220175230460](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220175230460.png)

创建了1个交换机和两个队列：

![image-20240220175426799](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220175426799.png)

![image-20240220175445858](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220175445858.png)

队列1和队列2都收到了这4条消息，由于开启了自动确认，所以消息很快被消费了：

![image-20240220175336383](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220175336383.png)

关闭自动确认，再次测试，先启动消费者，再启动生产者，并连续发送4条消息，可以看到小王和小李这两个队列里各有4条消息：

![image-20240220180129495](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220180129495.png)

### 4. Routing（direct直连交换机/路由模式）：

在上面fanout交换机模式上增加了绑定关系，消息会根据路由键转发到指定的队列。

应用场景：特定的消息只交给特定的系统（程序）来处理：

![image-20240220181402021](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220181402021.png)

官方教程：[RabbitMQ tutorial - Routing — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-four-java.html)

![image-20240220180610486](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220180610486.png)

可以看到不同的队列可以绑定相同的路由键：

![image-20240220180726182](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220180726182.png)

一个队列也可以绑定多个不同的路由键：

![image-20240220180816928](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220180816928.png)

#### 1）生产者代码：

在fanout扇出交换机模型上，多加了绑定关系，即路由键。

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.Scanner;

public class DirectProducer {
    private static final String EXCHANGE_NAME = "direct-exchange";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            // 声明一个直连交换机（direct exchange）
            channel.exchangeDeclare(EXCHANGE_NAME, "direct");

            // 创建一个从控制台读取用户输入的 Scanner 对象
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNext()) {
                String userInput = scanner.nextLine();
                String[] strings = userInput.split(" ");
                // 如果分割后的数组长度小于1，则继续等待下一次输入
                if (strings.length < 1) {
                    continue;
                }
                // 获取消息和路由键，第一个元素为消息内容，第二个元素为路由键
                String message = strings[0];
                String routingKey = strings[1];

                // 发布消息到指定的交换机和使用指定的路由键
                channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes("UTF-8"));
                System.out.println(" [x] Sent '" + message + " with routing:" + routingKey + "'");
            }
        }
    }
}
```

#### 2）消费者代码：

和fanout扇出交换机代码相似，多了路由键绑定交换机和每个队列：

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class DirectConsumer {
    private static final String EXCHANGE_NAME = "direct-exchange";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "direct");

        // 创建队列，随机分配一个队列名称
        String queueName = "xiaoA_queue";
        channel.queueDeclare(queueName, true, false, false, null);
        channel.queueBind(queueName, EXCHANGE_NAME, "xiaoA");

        // 创建队列，随机分配一个队列名称
        String queueName2 = "xiaoB_queue";
        channel.queueDeclare(queueName2, true, false, false, null);
        channel.queueBind(queueName2, EXCHANGE_NAME, "xiaoB");

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback xiaoADeliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [xiaoA] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };

        DeliverCallback xiaoBDeliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [xiaoB] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };

        channel.basicConsume(queueName, true, xiaoADeliverCallback, consumerTag -> {
        });
        channel.basicConsume(queueName2, true, xiaoBDeliverCallback, consumerTag -> {
        });
    }
}
```

#### 3）测试：

先启动消费者，再启动生产者，向xiaoA、xiaoB分别发送一条消息，再向一个未绑定的"小树枝"发送一条消息：

![image-20240220185453804](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220185453804.png)

可以看到，只有xiaoA和xiaoB收到了消息，小树枝没有收到消息：

![image-20240220185548823](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220185548823.png)

创建了direct交换机和xiaoA、xiaoB队列，同样由于开启了自动确认，所以消息很快被消费了：

![image-20240220185659562](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220185659562.png)

![image-20240220185726443](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220185726443.png)

关闭自动确认，再次测试，可以看到xiaoA、xiaoB队列里各有一条消息：

![image-20240220190033897](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220190033897.png)

### 5. Topics（topic主题交换机/主题模式）

和上面direct直连交换机模式很相似，不过topics模式中的路由键需要模糊定义，消息会根据一个 **模糊的** 路由键转发到指定的队列。

官方教程：[RabbitMQ tutorial - Topics — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-five-java.html)

![image-20240220191123373](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220191123373.png)

- Q1–>绑定的是：中间带 orange 带 3 个单词的字符串(*.orange.*)
- Q2–>绑定的是：最后一个单词是 rabbit 的 3 个单词(*.*.rabbit)，第一个单词是 lazy 的多个单词(lazy.#)

上图是一个队列绑定关系图，我们来看看他们之间数据接收情况是怎么样的：

```js
quick.orange.rabbit             被队列 Q1Q2 接收到
lazy.orange.elephant            被队列 Q1Q2 接收到
quick.orange.fox                    被队列 Q1 接收到
lazy.brown.fox                      被队列 Q2 接收到
lazy.pink.rabbit                    虽然满足两个绑定但只被队列 Q2 接收一次
quick.brown.fox                     不匹配任何绑定不会被任何队列接收到会被丢弃
quick.orange.male.rabbit    是四个单词不匹配任何绑定会被丢弃
lazy.orange.male.rabbit     是四个单词但匹配 Q2
```

●*：匹配一个单词，比如 *.orange，那么 a.orange、b.orange 都能匹配

●#：匹配 0 个或多个单词，比如 a.#，那么 a.a、a.b、a.a.a 都能匹配

应用场景：特定的一类消息可以交给特定的一类系统（程序）来处理

![image-20240220190918642](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220190918642.png)

#### 1）生产者代码：

和上面direct直连交换机生产者代码一样，只不过交换机名称改变：

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.Scanner;

public class TopicProducer {
    private static final String EXCHANGE_NAME = "topic-exchange";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {

            channel.exchangeDeclare(EXCHANGE_NAME, "topic");

            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNext()) {
                String userInput = scanner.nextLine();
                String[] strings = userInput.split(" ");
                if (strings.length < 1) {
                    continue;
                }
                String message = strings[0];
                String routingKey = strings[1];

                channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes("UTF-8"));
                System.out.println(" [x] Sent '" + message + " with routing:" + routingKey + "'");
            }
        }
    }
}
```

#### 2）消费者代码：

消费者代码也很相似，讲路由键由固定精确表达改为模糊表达，这次创建三个队列方便测试：

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class TopicConsumer {
    private static final String EXCHANGE_NAME = "topic-exchange";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "topic");

        // 创建队列
        String queueName = "frontend_queue";
        channel.queueDeclare(queueName, true, false, false, null);
        channel.queueBind(queueName, EXCHANGE_NAME, "#.前端.#");

        // 创建队列
        String queueName2 = "backend_queue";
        channel.queueDeclare(queueName2, true, false, false, null);
        channel.queueBind(queueName2, EXCHANGE_NAME, "#.后端.#");

        // 创建队列
        String queueName3 = "product_queue";
        channel.queueDeclare(queueName3, true, false, false, null);
        channel.queueBind(queueName3, EXCHANGE_NAME, "#.产品.#");

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback xiaoaDeliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [xiaoa] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };

        DeliverCallback xiaobDeliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [xiaob] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };

        DeliverCallback xiaocDeliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [xiaoc] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };

        channel.basicConsume(queueName, true, xiaoaDeliverCallback, consumerTag -> {
        });
        channel.basicConsume(queueName2, true, xiaobDeliverCallback, consumerTag -> {
        });
        channel.basicConsume(queueName3, true, xiaocDeliverCallback, consumerTag -> {
        });
    }
}
```

#### 3）测试：

先关闭自动确认，开启消费者，再开启生产者，分别发送消息给："前端.后端.产品"、"前端.后端"、"后端.产品"、"后端"、"后端.后端"、"运维.后端"、"运维.UI"：

![image-20240220194448532](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220194448532.png)

预期匹配关系：

![image-20240221110447047](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240221110447047.png)

实际匹配关系，和预期一致：

![image-20240220194433870](C:\Users\周斌\AppData\Roaming\Typora\typora-user-images\image-20240220194433870.png)

创建了topic交换机和frontend、backend、product队列，backend队列里有6条消息，frontend队列有2条消息，product队列里有2条消息：

![image-20240220195220315](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220195220315.png)

![image-20240220195059831](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240220195059831.png)

### 6. Headers 交换机：

> 该模型使用场景不多，本文不做演示。

> 以下内容来自ChatGPT：
>
> 在 RabbitMQ 中，Headers 交换机（Headers Exchange）是一种交换机类型，它是根据消息的 headers 属性来路由消息的。具体来说，Headers 交换机会检查消息的 headers 中的键值对，只有当消息的 headers 中包含了与绑定规则完全匹配的键值对时，消息才会被路由到绑定到该 Headers 交换机上的队列中。
>
> Headers 交换机的使用场景通常是需要根据消息的 headers 属性来进行复杂路由的情况，比如根据不同的 headers 属性值将消息路由到不同的队列中。Headers 交换机的主要优点是可以实现灵活的消息路由，不依赖于消息的 routing key，而是根据 headers 中的键值对来进行路由；同时可以匹配多个条件，实现更灵活的消息过滤与路由。
>
> 然而，Headers 交换机也存在一些缺点。首先是性能相对较低，因为需要在 headers 中进行键值对的匹配，需要消耗更多的计算资源。其次，Headers 交换机在匹配规则方面相对复杂，配置较为繁琐，不如 Direct Exchange 或 Topic Exchange 直观易用。
>
> 综上所述，Headers 交换机适合于需要根据消息的 headers 属性来进行复杂路由的场景，可以提供灵活的消息过滤与路由，但在性能和配置上相对较高。在选择使用 Headers 交换机时，需要根据实际需求权衡其优缺点。

### 7. RPC模型（远程调用）：

因为远程调用有专门的远程调用框架，Dubbo，GRPC，用RabbitMQ来做远程调用的场景也不是很多，所以本文不再演示。

### 8. Publisher Confirms（发布者确认模式）：

> [Publisher confirms](https://www.rabbitmq.com/confirms.html#publisher-confirms) are a RabbitMQ extension to implement reliable publishing. When publisher confirms are enabled on a channel, messages the client publishes are confirmed asynchronously by the broker, meaning they have been taken care of on the server side.

官方教程：[RabbitMQ tutorial - Reliable Publishing with Publisher Confirms — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-seven-java.html)

使用 Publisher Confirms 模型时，当生产者发送消息到 RabbitMQ 之后，会等待来自 RabbitMQ 的确认消息。如果确认消息被返回，意味着消息已经被发布到交换机上。如果在一定时间内没有收到确认消息，生产者将会认为消息发送失败，并可以进行相应的处理，比如重发消息或记录日志。

Publisher Confirms 模型的主要用途是确保消息的可靠性传递。通过等待确认消息，生产者可以了解消息是否成功发送到 RabbitMQ 中。如果消息发送失败或丢失，生产者可以根据具体情况采取相应的措施，保证消息不会丢失。

该模式和下篇RabbitMQ的特性有关，所以本文不做演示。



#### Springboot使用RabbitMQ及RabbitMQ特性请看下篇！！

#### 