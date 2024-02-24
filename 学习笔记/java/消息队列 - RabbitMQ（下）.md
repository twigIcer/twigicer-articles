# 消息队列 - RabbitMQ（下）

> 本文分为上中下三篇，上篇介绍重点消息队列的概念和RabbitMQ的安装及组件介绍，中篇为RabbitMQ的入门实践，下篇主要讲在Springboot中使用RabbitMQ及RabbitMQ的核心特性。

是真没想到RabbitMQ的内容这么多，从一篇分成两篇，再到分为三篇，不过这篇绝对是最后一篇！！

## 在Spring Boot中使用RabbitMQ:

> 在spring boot中使用RabbitMQ比较简单，下面RabbitMQ的direct交换机模式作为演示。

### 1. 引依赖：

还是老样子，第一步引依赖，直接使用springboot starter:

```yml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-amqp -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>2.7.2</version>
</dependency>
```

注意：

1. 使用的是`spring-boot-starter-amqp`而不是`RabbitMQ` 的starter。
2. 版本与自己项目的springboot版本一致。

### 2. 在yml文件中引入配置：

第二步也还是在application.yml文件中引入配置：

```yml
spring:
	rabbitmq:
    	host: localhost
    	port: 5672
   		password: guest
    	username: guest
```

如果上面的配置都是默认的，不写这些配置也可以，不过最好配置一下。

### 3. 编写配置类：

编写一个配置类，声明交换机，队列，及交换机和队列的绑定：

```java
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    // 定义交换机名称
    public static final String EXCHANGE_NAME = "direct_exchange";

    // 定义队列名称
    public static final String QUEUE_NAME = "direct_queue";

    // 定义Routing Key
    public static final String ROUTING_KEY = "direct_routing_key";

    // 声明Direct交换机
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange(EXCHANGE_NAME);
    }

    // 声明队列
    @Bean
    public Queue queue() {
        return new Queue(QUEUE_NAME, true); // 第二个参数表示队列持久化
    }

    // 绑定队列到交换机，并指定绑定键
    @Bean
    public Binding binding(Queue queue, DirectExchange directExchange) {
        return BindingBuilder.bind(queue).to(directExchange).with(ROUTING_KEY);
    }
}
```

使用BindingBuilder.bind()方法对队列和交换机做绑定。

### 4. 创建生产者：

直接使用`RabbitTemplate` ,调用RabbitTemplate.convertAndSend()方法即可向指定交换机或者队列发送消息：

```java
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

@Component
public class MyMessageProducer {
    @Resource
    private RabbitTemplate rabbitTemplate;

    public void sendMessage(String exchange, String routingKey, String message) {
        rabbitTemplate.convertAndSend(exchange, routingKey, message);
    }
}
```

上面演示的为向指定交换机用指定路由键发送消息，RabbitTemplate.convertAndSend()方法有多种重写，可以根据需要选择（可参考IDEA提示），下面为RabbitTemplate.convertAndSend()方法的重写及参数：

1. convertAndSend(String routingKey, Object message)：

* `routingKey`：指定消息发送到的路由键。

- `message`：要发送的消息对象。

  ```java
  rabbitTemplate.convertAndSend("myQueue", "Hello, RabbitMQ!");
  ```

2. convertAndSend(String exchange, String routingKey, Object message)：

- `exchange`：指定消息发送到的交换机。

- `routingKey`：指定消息发送到的路由键。

- `message`：要发送的消息对象。

  ```java
  rabbitTemplate.convertAndSend("myExchange", "myRoutingKey", "Hello, RabbitMQ!");
  ```

3. convertAndSend(Object message, MessagePostProcessor messagePostProcessor)：

- `message`：要发送的消息对象。

- `messagePostProcessor`：用于在发送消息之前对消息进行后处理的消息后处理器。

  ```java
  rabbitTemplate.convertAndSend("myQueue", "Hello, RabbitMQ!", message -> {
      message.getMessageProperties().setHeader("headerKey", "headerValue");
      return message;
  });
  ```

4. convertAndSend(String exchange, String routingKey, Object message, MessagePostProcessor messagePostProcessor)：

- `exchange`：指定消息发送到的交换机。

- `routingKey`：指定消息发送到的路由键。

- `message`：要发送的消息对象。

- `messagePostProcessor`：用于在发送消息之前对消息进行后处理的消息后处理器。

  ```java
  rabbitTemplate.convertAndSend("myExchange", "myRoutingKey", "Hello, RabbitMQ!", message -> {
      message.getMessageProperties().setHeader("headerKey", "headerValue");
      return message;
  });
  ```

### 5. 创建消费者：

使用 `@RabbitListener` 注解来监听队列并处理接收到的消息：

```java
import com.rabbitmq.client.Channel;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class MyMessageConsumer {
    // 指定程序监听的消息队列和确认机制
    @SneakyThrows
    @RabbitListener(queues = {QUEUE_NAME})
    public void receiveMessage(String message)  {
        log.info("receiveMessage message = {}", message);
    }
}
```

1）`@SneakyThrows` 注解

这是 Lombok 提供的注解，它能够在方法中自动处理异常，使代码更加简洁。如果方法中出现了检查异常，它会自动捕获并抛出非检查异常，但是需要注意，使用这个注解时需要确保代码中没有未被捕获的检查异常。

2）@RabbitListener注解：

这个注解用于将方法标记为 RabbitMQ 消息的监听器。它告诉 Spring Boot 当有消息到达指定的队列时，要调用该方法来处理消息。

1. **queues**：指定监听的队列名，可以是一个字符串数组，表示同时监听多个队列，或者是一个字符串，表示只监听一个队列。
2. **id**：为监听器指定一个唯一的标识符。如果不指定，默认为 bean 名称。
3. **containerFactory**：指定使用的消息监听容器工厂。用于自定义监听器容器的配置，例如线程数、消息转换器等。
4. **ackMode**：指定消息确认模式。可选值包括 `AUTO`、`MANUAL` 和 `NONE`。默认是 `AUTO`，表示自动确认消息；`MANUAL` 表示手动确认消息；`NONE` 表示不进行消息确认。
5. **concurrency**：指定消费者的并发数量。默认为 1。可以通过 `concurrency` 参数设置监听器容器的消费者数量，从而实现多个消费者并发处理消息。

### 6. 测试：

编写一个接口，调用接口来测试：

```java
@RestController
@RequestMapping("/MQ")
@Slf4j
public class MQController {
    @Resource
    private MyMessageProducer myMessageProducer;

    @PostMapping("/tsetMQ")
    public void testMQ(String message){
        myMessageProducer.sendMessage(EXCHANGE_NAME,ROUTING_KEY,message);
    }
}
```

调用接口，发送消息"你好"：

![image-20240222134201981](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240222134201981.png)

可以看到控制台输出了：receiveMessage message = 你好，说明消费成功：

![image-20240222134032553](https://articleimgs.xiaoshuzhi.xyz/imgs/image-20240222134032553.png)

## RabbitMQ的特性（机制）：

### 1. 消息持久化：

RabbitMQ 的消息持久化是指将消息保存在磁盘上，以确保在 RabbitMQ 服务器重启或发生故障时，消息不会丢失。消息的持久化涉及到两个方面：**队列持久化**和**消息持久化**。

#### 1）队列持久化：

在声明队列时，通过设置  参数为  来使队列持久化：`durable``true`。

```java
// 创建连接和通道
Connection connection = factory.newConnection();
Channel channel = connection.createChannel();

// 声明一个持久化队列
channel.queueDeclare("your_queue_name", true, false, false, null);
```

这样设置后，队列将在 RabbitMQ 服务器重启时仍然存在。

#### 2）消息持久化：

当发送消息时，通过设置 `deliveryMode` 参数为 `2` 来使消息持久化：

```java
// 创建连接和通道
Connection connection = factory.newConnection();
Channel channel = connection.createChannel();

// 发送持久化消息
String message = "Hello, RabbitMQ!";
channel.basicPublish("", "your_queue_name", MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
```

这里使用了 `MessageProperties.PERSISTENT_TEXT_PLAIN` 来设置消息的持久化属性，其中 `PERSISTENT_TEXT_PLAIN` 表示消息内容是文本，并且需要持久化。

>  注意：消息持久化并不是百分之百的保证消息不会丢失。在消息刚被接收但尚未被写入磁盘时，RabbitMQ 服务器如果发生崩溃，消息可能会丢失。因此，建议在生产环境中搭配合适的高可用和备份机制。

#### 3）在Spring Boot项目中的持久化设置：

消息持久化：

Spring Boot中可以通过配置属性来设置消息持久化：

```yml
# 消息持久化
spring:
	rabbitmq: 
		template:
			default-receive-queue: your-queue-name
            delivery-mode: 2
                
# 队列持久化
spring:
	rabbitmq: 
		listener:
			direct:
				default-requeue-rejected: false
            simple:
				default-requeue-rejected: false
```

1. `spring.rabbitmq.template.default-receive-queue=your-queue-name`：这个属性指定了默认接收消息的队列名称。你需要将"your-queue-name"替换为你实际使用的队列名称。
2. `spring.rabbitmq.template.delivery-mode=2`：这个属性设置了消息的传递模式。在这里，`2`表示消息持久化（Persistent）。这样设置后，所有通过RabbitMQ模板发送的消息都将被标记为持久化。
3. `spring.rabbitmq.listener.direct.default-requeue-rejected=false`：这个属性设置了是否在拒绝消息时重新排队。当消费者拒绝消息时，如果将此属性设置为`false`，则消息将被丢弃而不会重新排队。这样可以避免消息在处理失败后不断地重新排队。
4. `spring.rabbitmq.listener.simple.default-requeue-rejected=false`：与上面的属性类似，但是这个属性适用于简单的消息监听器容器。同样，设置为`false`可以确保在消息被拒绝时不会重新排队。

队列持久化：

在配置类中，通过`Queue`的构造方法的第二个参数设置队列的持久化，值为`true`表示持久化。通过`RabbitTemplate`的`setQueue`方法和`setDeliveryMode`方法分别设置消息发送到的队列和消息的持久化模式。

```java
@Configuration
public class RabbitMQConfig {

    @Bean
    public Queue myQueue() {
        // 队列持久化
        return new Queue("your-queue-name", true);
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        // 消息持久化
        rabbitTemplate.setQueue("your-queue-name");
        rabbitTemplate.setDeliveryMode(2);
        return rabbitTemplate;
    }
}
```

### 2. 消息确认机制：

RabbitMQ 的消息确认机制是为了确保消息在生产者和消息代理之间的可靠传递。这个机制涉及到**生产者确认**（Publisher Confirms）和**消费者确认**（Consumer Acknowledgments）两个方面。

#### 1）生产者确认（Publisher Confirms）：

* **异步确认：** 在生产者发送消息到 RabbitMQ 时，可以通过开启异步确认模式来确保消息是否成功发送到 RabbitMQ 服务器。

```java
// 开启异步确认模式
rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
    if (ack) {
        // 消息成功发送到 RabbitMQ
        System.out.println("Message successfully sent to RabbitMQ");
    } else {
        // 消息发送失败，处理失败逻辑
        System.err.println("Message failed to be sent to RabbitMQ: " + cause);
    }
});
```

* **事务确认：** 生产者还可以选择使用事务机制，将消息发送到 RabbitMQ 前开启事务，待消息成功发送后提交事务，否则回滚事务。

```java
try {
    // 开启事务
    channel.txSelect();
    
    // 发送消息
    channel.basicPublish(exchange, routingKey, null, message.getBytes());
    
    // 提交事务
    channel.txCommit();
} catch (IOException e) {
    // 处理异常，回滚事务
    channel.txRollback();
}
```

#### 2) 消费者确认（Consumer Acknowledgments）：

* **自动确认：** 默认情况下，RabbitMQ 的消费者是自动确认消息的，即消费者收到消息后立即确认。这样可能导致消息在处理过程中出现异常时丢失，因为消息一旦被确认就从队列中删除了。

```java
@RabbitListener(queues = "your_queue_name")
public void handleMessage(Message message) {
    // 处理消息
    
    // 消息会在方法执行完成后自动确认
}
```

* **手动确认：** 为了更好地控制消息的确认，消费者可以选择手动确认模式。在消息处理完成后，通过调用 `basicAck` 方法手动确认消息。

```java
@RabbitListener(queues = "your_queue_name")
public void handleMessage(Message message, Channel channel) throws IOException {
    try {
        // 处理消息
        
        // 手动确认消息
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    } catch (Exception e) {
        // 处理消息出现异常，拒绝消息并重新入队列
        channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
    }
}
```

`channel.basicAck` 用于手动确认消息，`channel.basicNack` 用于拒绝消息并重新入队列，可以根据业务需要进行合适的处理。

#### 3. 死信队列：

RabbitMQ 中的死信队列（Dead Letter Queue，DLQ）是一种用于处理那些无法被消费者成功处理的消息的机制。当消息满足一定条件时，例如消息被拒绝、消息过期或者队列达到最大长度时，这些消息就会被标记为“死信”，然后被重新路由到死信队列中。

#### 1）死信队列的作用：

1. **错误处理**：将处理失败的消息转移到死信队列中，便于进一步分析和处理错误的消息。
2. **重试机制**：将死信队列配置为另一个消费者消费，以实现消息的重试机制。
3. **延迟队列**：结合消息的 TTL（Time-To-Live）设置和死信队列，可以实现延迟消息队列的功能。

#### 2）设置死信队列的步骤：

1. **定义死信交换机和队列**：首先需要定义一个死信交换机和一个死信队列。当消息成为死信时，会被发送到死信交换机，并根据绑定规则路由到死信队列中。
2. **配置原始队列**：在定义原始队列时，需要指定该队列中的消息成为死信的条件。这可以通过设置队列的参数 `x-dead-letter-exchange`（指定死信交换机）和 `x-dead-letter-routing-key`（指定死信路由键）来实现。
3. **发送消息**：将消息发送到原始队列。当消息满足死信条件时，会被发送到死信队列中。

#### 3）代码演示：

```java
// 声明死信交换机
channel.exchangeDeclare("dlx_exchange", "direct");

// 声明死信队列
channel.queueDeclare("dlx_queue", true, false, false, null);
channel.queueBind("dlx_queue", "dlx_exchange", "dlx_routing_key");

// 声明原始队列，并设置死信交换机和死信路由键
Map<String, Object> arguments = new HashMap<>();
arguments.put("x-dead-letter-exchange", "dlx_exchange");
arguments.put("x-dead-letter-routing-key", "dlx_routing_key");
channel.queueDeclare("original_queue", true, false, false, arguments);

// 发送消息到原始队列
String message = "Hello, RabbitMQ!";
channel.basicPublish("", "original_queue", null, message.getBytes());
```

## 小结：

终于写完了，不整理不知道，一整理吓一跳，RabbitMQ的内容居然这么多！！其实RabbitMQ的机制这部分写的有点水，这些机制要细写的话，估计还要再开两篇，RabbitMQ还有一些其他的机制，我实在是写不动了，就不写了。

很佩服知乎这位作者，写三万字来讲解RabbitMQ，讲解的很到位，很详细，想要更深入了解RabbitMQ的友子们，可以去看看这篇文章：

[爆肝3万字，为你吃透RabbitMQ，最详细的RabbitMQ讲解（VIP典藏版） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/611247550)