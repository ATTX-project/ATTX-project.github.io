# ActiveMQ - ATTX MessageBroker


## Image

Custom image derived from: https://github.com/krizsan/activemq-docker

## Using ActiveMQ

MessageBroker can be accessed using multiple different protocols.
* OpenWire
* Stomp 1.0+ - in order to use with Python make use of the libraries (https://pypi.org/project/stompest/ and https://pypi.org/project/stompest.async/ or https://pypi.org/project/stomp.py/)
* JMS - recommended implementation is using Spring https://spring.io/guides/gs/messaging-jms/
* AMQP v1.0 (http://qpid.apache.org/proton/)
* MQTT v3.1
[List of code example](http://activemq.apache.org/cross-language-clients.html)

## Administration

When something goes wrong.

* Where is the admin interface?
    * available at: `http://<container-ip>:8161/hawtio/`
* What are the admin credentials?
    * default one available can be further configured using http://activemq.apache.org/security.html information

## References

* [Common Messaging Patterns Using Stomp](https://www.devco.net/archives/2011/12/11/common-messaging-patterns-using-stomp.php)
