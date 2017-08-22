## Messaging Brokers Solutions Overview

Overview based on:
* **Ecosystem** - Documentation, Active Development, Open License, Ease of Use
* **Features** - Topics and Queues, Reliable Messaging, REST Management API, Streams processing

| Broker Tool / Messaging Platform                                                                         | Ok | Ecosystem | Features |
|----------------------------------------------------------------------------------------------------------|----|-----------|----------|
| [Apache Kafka](http://kafka.apache.org/)                                                                 | ✔️️  | ✔️️         | ✔️️        |
| [NATS](http://www.nats.io/)                                                                              |    |           |          |
| [Active MQ](http://activemq.apache.org/) | ✔️️  | ✔️️         | ✔️️  (using [hawt.io](http://hawt.io))      |
| [RabbitMQ](https://www.rabbitmq.com/)                                                                    |  ✔️️  |      ✔️️     |     ✔️️     |
| [Celery](http://www.celeryproject.org/)                                                                  |    |           |          |
| [nsq.io](http://nsq.io/)                                                                                 | ✔️️  | ✔️️         | ~        |
| [ZeroMQ](http://zeromq.org/)                                                                             | ✔️️  | ~         | ~        |
| [Apache Qpid](https://qpid.apache.org/)                                                                  |    |           |          |
| [RestMQ](http://restmq.com/)                                                                             |    |           |          |
| [beanstalkd](http://kr.github.io/beanstalkd/)                                                            |    |           |          |

Worth mentioning:
* [Apache Storm](https://storm.apache.org/) - distributed realtime computation system
* [Apache Samza](http://samza.apache.org/) -  distributed stream processing framework uses [Apache Kafka](http://kafka.apache.org/)

## References

* http://queues.io/
* http://data-artisans.com/kafka-flink-a-practical-how-to/
* From Kafka to ZeroMQ for real-time log aggregation https://tomasz.janczuk.org/2015/09/from-kafka-to-zeromq-for-log-aggregation.html
* Reactive Streams for Kafka https://github.com/akka/reactive-kafka
