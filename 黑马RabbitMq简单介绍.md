# RabbitMq

### 1 同步通讯优缺点

![](C:\Users\1270212176\Desktop\大三下实训\同步通讯.png)

**优点：**时效性高

**缺点：**

1 耦合度高，每次加入新需求都需要改动代码

2 性能低，吞吐量低

3 一次只能调用一个服务，占着cpu资源，资源浪费率高,

4 级联实现，某个服务挂掉后容易引起阻塞（级联失败）

### 2 异步通讯优缺点

常见实现方式：事件驱动模式（由事件通知其它服务）

![](C:\Users\1270212176\Desktop\大三下实训\异步通讯.png)

***优点**

1 服务解耦

2 性能提升，吞吐量提高

3 服务没有强依赖，没有级联失败的问题

4 流量消峰（由事件来进行控制）

![](C:\Users\1270212176\Desktop\大三下实训\流量消峰.png)



**缺点**

1 依赖于Broker的可靠应、安全性、吞吐能力

2 架构复杂了、业务没有明显的流程性不好追踪管理 



### 3 什么是MQ

MQ(MessageQueue)就是消息队列，即存放消息的队列，也就是事件驱动架构中的Broker



### 4 常用的MQ及区别

![](C:\Users\1270212176\Desktop\大三下实训\常见的MQ技术.png)



### 5 RabbitMq几个概念



![](C:\Users\1270212176\Desktop\大三下实训\RabbitMq组件介绍.png)

### 6 消息队列模型

![](C:\Users\1270212176\Desktop\大三下实训\消息队列模型.png)

**1 简单队列模型**

**生产者**

 1 创建连接，设置参数，建立连接

![](C:\Users\1270212176\Desktop\大三下实训\简单模式建立连接.png)

2 创建通道，创建队列

![](C:\Users\1270212176\Desktop\大三下实训\简单模式创建通道.png)

3 发送消息并关闭连接

![](C:\Users\1270212176\Desktop\大三下实训\简单队列发送消息关闭连接.png)



**消费者**

1 创建连接，设置参数，建立连接

2 创建通道，创建队列

3 订阅消息

![](C:\Users\1270212176\Desktop\大三下实训\简单模式消费者订阅消息.png)

### 7 什么是Spring AMQP

AMQP是消息队列的一种规范协议。

Spring AMQP是基于AMQP协议定义的一套Api规范，提供了模板来发送和接受消息。(类似于Redis的RedisTemplate)

包含两部分：spring-amqp是基础抽象  spring-rabbit是底层实现

### 8 Spring AMQP实现简单队列

**基本流程**

![](C:\Users\1270212176\Desktop\大三下实训\SpringAMQP实现简单对列基本流程.png)

**依赖**

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



**生产者yml文件及测试类**

![](C:\Users\1270212176\Desktop\大三下实训\RabbitMq学习截图\SpringAmqp简单模式生产者配置文件.png)

**注意：此方法不会自动创建队列，队列必须已经存在**



**SpringAMQP接收消息**

1 依赖，yml文件。与生产者类似

**2 创建消费逻辑，监听队列**

![](C:\Users\1270212176\Desktop\大三下实训\RabbitMq学习截图\SpringAmqp简单模式消费者监听队列.png)

注意：消息一旦消费就会从队列中删除，RabbitMq没有消息回溯功能

### 9 Spring AMQP实现工作队列

![](C:\Users\1270212176\Desktop\大三下实训\RabbitMq学习截图\工作队列.png)

**一个队列后跟着两个消费者**

**优点：提高消息处理速度，避免队列消息堆积**



**注意：同一条消息只会被一个消费者消费**



**模拟工作队列**

![](C:\Users\1270212176\Desktop\大三下实训\RabbitMq学习截图\模拟工作队列.png)

**生产者**

```
 //模拟工作队列
    @Test
    public void workQueueAMQP() throws InterruptedException {
        String queueName="simple.queue";
        String messqge = "WorkQueue SpringAMQP __";
        for (int i=1;i<=50;i++){
            rabbitTemplate.convertAndSend(queueName,messqge+i);
            Thread.sleep(20);
        }

    }
```

**消费者1**

```
//模拟工作队列消费者1
    @RabbitListener(queues = "simple.queue")
    public void workQueueAMQP(String msg) throws Exception{
        System.out.println("消费者1接收到消息：[" + msg +"]" + LocalTime.now());
        Thread.sleep(20);
    }
```

**消费者2**

```
//模拟工作队列消费者2
    @RabbitListener(queues = "simple.queue")
    public void workQueueAMQP2(String msg) throws Exception{
        System.err.println("消费者2接收到消息：[" + msg +"]" + LocalTime.now());
        Thread.sleep(200);
    }
```

**注意：按照上面的实现方式，50条消息会平均分配给两个消费者，这是因为Rabbitmq的消息预取装置，两个队列在消费消息前，会轮流从队列取出消息，不会考虑自身性能**



**通过在yml文件中修改spring.rabbitmq.listener.direct.prefetch的值来设置预取**

```
spring:
  rabbitmq:
    host: 192.168.61.131
    port: 5672
    virtual-host: /
    username: guest
    password: guest
    listener:
      direct:
        prefetch: 1 #每次只能获取一条消息，处理完再进行获取
```



### 10 SpringAMQP 实现发布订阅模式



**发布订阅模式允许将同一条信息发给多个消费者。实现方式是加入exchange(交换机)**

![](C:\Users\1270212176\Desktop\大三下实训\RabbitMq学习截图\发布订阅模式.png)

**发给几个队列由交换机的类型决定**

**常见的交换机有：Fanout(广播)、Direct(路由)、Topic(话题)**



### 11 广播交换机（FanoutExchange）

会将接收到的消息路由每个跟其绑定的queue

![](C:\Users\1270212176\Desktop\大三下实训\RabbitMq学习截图\SpringAMQP实现Fanout.png)

![](C:\Users\1270212176\Desktop\大三下实训\RabbitMq学习截图\fanout交换机.png)

 **步骤1：再consumer服务声明ExChange、Queue、Binding**



声明交换机

```
//声明交换机
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("itcast.fanout");
    }
```



声明队列1，并绑定到交换机

```
 //声明队列1
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }

    //绑定队列1与交换机
    @Bean
    public Binding fanoutBinding(Queue fanoutQueue1,FanoutExchange fanoutExchange){
        return BindingBuilder
                .bind(fanoutQueue1)
                .to(fanoutExchange);
    }
```



声明队列2，并绑定到交换机

```
//声明队列2
@Bean
public Queue fanoutQueue2(){
    return new Queue("fanout.queue2");
}

//绑定队列2与交换机
@Bean
public Binding fanoutBinding2(Queue fanoutQueue2,FanoutExchange fanoutExchange){
    return BindingBuilder
            .bind(fanoutQueue2)
            .to(fanoutExchange);
}
```

**上面三个步骤均在消费者的配置类下配置**



消费者

```
//订阅模式队列1
@RabbitListener(queues = "fanout.queue1")
public void fanoutExchange(String msg) throws Exception{
    System.err.println("fanout.queue1接收到消息：[" + msg +"]" + LocalTime.now());

}

//订阅模式队列2
@RabbitListener(queues = "fanout.queue2")
public void fanoutExchange2(String msg) throws Exception{
    System.err.println("fanout.queue2接收到消息：[" + msg +"]" + LocalTime.now());

}
```



生产者

```
@Test
public void sendFanoutExchange(){
    //交换机名称
    String exchangeName = "itcast.fanout";

    //消息
    String message = "hello,everyone";

    rabbitTemplate.convertAndSend(exchangeName,"",message);
}
```



交换机的作用：接受生产者发送的消息

将消息按照规则路由到与之绑定的队列



**注意：交换机只能做消息的转发不能做消息的存储，如果路由没有成功，消息会丢失**