# 安装配置
rabbitmq暗转完成后，Windows平台运行`rabbitmq command prompt`，然后执行`rabbitmq-plugins.bat enable rabbitmq_management`安装插件。执行`rabbitmq-server.bat`启动rabbitmq服务。成功启动之后可以通过`localhost:15672`访问管理后台，默认账号、密码均为"guest"。

# hello world

Java版本需要添加如下依赖：

```java
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.10.0</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.30</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.30</version>
</dependency>
```

生产者示例：

```java
public class Sender {
    private final static String queueName = "hello";
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(queueName, false, false, false, null);
            String message = "hello, world";
            channel.basicPublish("", queueName, null, message.getBytes());
            System.out.println(" [x] send ' " + message + "'");
        }
    }
}
```

消费者示例：
```java
public class Recv {
    private final static String queueName = "hello";
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(queueName, false, false, false, null);
        System.out.println(" [*] waiting for message. To exit press ctrl+c");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] received '" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {});
    }
}
```

# 任务队列

任务队列背后主要的思想是：避免资源密集型任务立即执行并且必须等待其完成。
