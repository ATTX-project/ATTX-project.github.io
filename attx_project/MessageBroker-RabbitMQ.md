# RabbitMQ - ATTX MessageBroker


## Image

rabbitmq:3.6.11-management

## Debuging messages using Firehose

http://www.rabbitmq.com/firehose.html

Enable trace
```
rabbitmqctl trace_on
```

Enable tracing plugin
```
rabbitmq-plugins enable rabbitmq_tracing
```

Tracing plugin assumes that there is guest/guest user available (add if missing).
Tracing UI can be found under the Admin tab.

## Working with JMS messages

JMS support is based on JMS topic selector plugin and JMS client library.
More detailed information can be found in https://www.rabbitmq.com/jms-client.html.

**Setup**

Enable jms plugin for a running RabbitMQ instance.

```
docker exec -it {containerID} rabbitmq-plugins enable rabbitmq_jms_topic_exchange
```

Another way to enable the plugin is to modify file /etc/rabbitmq/enabled_plugins and
add "rabbitmq_jms_topic_exchange" to the array of enabled plugins before starting the container.

**Usage**

Add JMS client dependency to the project.

Maven:
```
<dependency>
    <groupId>com.rabbitmq.jms</groupId>
    <artifactId>rabbitmq-jms</artifactId>
    <version>1.7.0</version>
</dependency>
```

Gradle:
```
compile group: 'com.rabbitmq.jms', name: 'rabbitmq-jms', version: '1.7.0'
```

**Sending messages**

On the client side (ie. the one sending service requests), the only required change concerns the ConnectionFactory, which needs to setup in the following way:

```java
RMQConnectionFactory connectionFactory = new RMQConnectionFactory();
connectionFactory.setUsername("");
connectionFactory.setPassword("");
connectionFactory.setHost("");
connectionFactory.setPort(12345);
```

**Receiving messages**

Simple example service, which uses a temporary queue setup by the client to send its response.

```java
import com.rabbitmq.jms.admin.RMQConnectionFactory;
import com.rabbitmq.jms.admin.RMQDestination;
import javax.jms.Connection;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.MessageListener;
import javax.jms.MessageProducer;
import javax.jms.Session;
import javax.jms.TextMessage;

public class Service implements MessageListener {

    private final String messageQueueName = "serviceQueue";
    private final int ackMode = Session.AUTO_ACKNOWLEDGE;

    private Session session = null;
    private MessageConsumer consumer = null;
    private MessageProducer producer = null;

    public RMLService() throws Exception {
        RMQConnectionFactory connectionFactory = new RMQConnectionFactory();
        connectionFactory.setUsername("");
        connectionFactory.setPassword("");

        Connection c = connectionFactory.createConnection();
        c.setClientID("Service");
        c.start();

        this.session = c.createSession(false, ackMode);
        this.consumer = session.createConsumer(session.createQueue(messageQueueName));
        this.consumer.setMessageListener(this);

        this.producer = session.createProducer(null);        
        log.info("Service created");

    }    

    @Override
    public void onMessage(Message msg) {
        try {
            TextMessage m = (TextMessage)msg;

            TextMessage response =  this.session.createTextMessage();            
            response.setText("response");          
            response.setStringProperty("correlation-id", m.getStringProperty("correlation-id"));
            String replyTo = msg.getStringProperty("reply-to");
            RMQDestination d = new RMQDestination(replyTo, true, true);
            d.setDeclared(true); // needed, because the sender has already created the queue
            this.producer.send(d, response);

        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }         

```

*TODO: SpringBoot example*
