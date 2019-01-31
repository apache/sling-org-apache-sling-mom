[<img src="https://sling.apache.org/res/logos/sling.png"/>](https://sling.apache.org)

 [![Build Status](https://builds.apache.org/buildStatus/icon?job=Sling/sling-org-apache-sling-mom/master)](https://builds.apache.org/job/Sling/job/sling-org-apache-sling-mom/job/master) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.apache.sling/org.apache.sling.mom/badge.svg)](https://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.apache.sling%22%20a%3A%22org.apache.sling.mom%22) [![JavaDocs](https://www.javadoc.io/badge/org.apache.sling/org.apache.sling.mom.svg)](https://www.javadoc.io/doc/org.apache.sling/org.apache.sling.mom) [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

# Apache Sling Message oriented Middleware API

This module is part of the [Apache Sling](https://sling.apache.org) project.

## Message Oriented Middleware API ( or message passing API).

This bundle contains an API to support 2 types of message passing. Pub/Sub and Queue.
Pub/Sub message passing supports the publication of messages to multiple subscribers. Queue 
support guaranteed delivery of a message to one and only one receiver. Pub/Sub messages are
organised by a Topic name. Queues are named. There are no implementation details in this 
bundle and no bindings to any one MoM implementation, although it is expected that this 
API could be implemented over either JMS or a AMQP client.

## Publish Subscribe

Messages that are sent to a topic by publishing are received by all subscribers that were active at the time the message is published.

To publish a message use the [TopicManager API](src/main/java/org/apache/sling/mom/TopicManager.java)

    topicManager.publish(String topic, Map<String, Object> message);
    
    where:
        topic is the name of the topic to which the message is published 
        message is the message as a Map or Maps.
        
To subscribe to a topic the caller must also use the [TopicManager API](src/main/java/org/apache/sling/mom/TopicManager.java)
 
    Subscription subscription = subscribe(Subscriber subscriber, String[] topicNames,MessageFilter filter);
    
    where:
          subscription is a Subsctipion objects which must be disposed (call dispose()) when the subscription ends.
          topicNames is an array of topic names.
          messageFilter is a MessageFilter implementation that accepts only those messages the subscriber is interested in.
          
The API does not impose any stcuture on topic names, but the underlying implementation might.

## Queue

A Queue implementation guarantees that messages will be delivered to one and only once QueueReader
in the order in which the messages were added to the Queue. The QueueReader implementation may
request to re-queue messages. The implementation should retry requeued messages after a suitable delay.

To add a message to a named queue use the [QueueManager API](src/main/java/org/apache/sling/mom/QueueManager.java)


        queueManager.add(String name, Map<String, Object> message);
        
        where: 
           name is the name of the queue.
           message is the message in map of maps form.
           
To receive messages from the queue use the [QueueManager API](src/main/java/org/apache/sling/mom/QueueManager.java)

        QueueSession queueSession =  queueManager.open( QueueReader queueReader, String name, MessageFilter messageFilter);
        
        where:
            queueSession is a QueueSession instance that must be closed (call close()) when the the queue reader requires no 
                         more messages,
            queueReader is a QueueReader implemetnation that will get delivered messages from the queue.
            name is the name of the queueu.
            messageFilter is a message filter intended to accept messages.
            
The QueueManager implementation will deliver messages to the queueReader.onMessage(...) method. The implementation of the QueueReader.onMessage method
may process the message, returning normally, or request the message is requeued by throwing a RequeueMessageExption.
