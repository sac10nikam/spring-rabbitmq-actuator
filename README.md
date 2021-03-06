spring-rabbitmq-actuator
========================

[![Maven Central](https://img.shields.io/maven-metadata/v/http/central.maven.org/maven2/com/itelg/spring/spring-rabbitmq-actuator/maven-metadata.xml.svg)](https://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.itelg.spring%22%20AND%20a%3A%22spring-rabbitmq-actuator%22)
[![Codacy Badge](https://api.codacy.com/project/badge/grade/ab6ef73712914dabac91965fe49eb297)](https://www.codacy.com/app/eggers-julian/spring-rabbitmq-actuator)
[![Codacy Badge](https://api.codacy.com/project/badge/Coverage/ab6ef73712914dabac91965fe49eb297)](https://www.codacy.com/app/eggers-julian/spring-rabbitmq-actuator)
[![Build Status](https://travis-ci.org/julian-eggers/spring-rabbitmq-actuator.svg?branch=master)](https://travis-ci.org/julian-eggers/spring-rabbitmq-actuator)

SpringBoot RabbitMQ Actuator (Queue Metrics & Health-Checks)

#### Maven
```xml
<dependency>
  <groupId>com.itelg.spring</groupId>
  <artifactId>spring-rabbitmq-actuator</artifactId>
  <version>0.6.0-RC2</version>
</dependency>
```

#### Precondition ([Example](https://github.com/julian-eggers/spring-rabbitmq-dead-letter-queue-example/blob/master/src/main/java/com/itelg/spring/rabbitmq/example/configuration/RabbitConfiguration.java#L76))
You have to add the declaring RabbitAdmin to each queue.
The specific RabbitAdmin is required to fetch the queue-information.
```java
@Bean
public Queue exampleQueue()
{
  Queue queue = new Queue("com.itelg.spring.rabbitmq.test");
  queue.setAdminsThatShouldDeclare(rabbitAdmin());
  return queue;
}
```


## Health-Checks

#### Example
```java
@Bean
public HealthIndicator rabbitQueueCheckHealthIndicator()
{
  RabbitQueueCheckHealthIndicator healthIndicator = new RabbitQueueCheckHealthIndicator();
  healthIndicator.addQueueCheck(exampleQueue1, 10000, 1);
  healthIndicator.addQueueCheck(exampleQueue2, 50000, 3);
  return healthIndicator;
}
```

#### Response ([/actuator/health](http://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-health))
```json
{
	"status" : "DOWN",
	"rabbitQueueCheck" : 
	{
		"status" : "DOWN",
		"com.examle.exampleQueue1" : 
		{
			"status" : "UP",
			"currentMessageCount" : 214,
			"maxMessageCount" : 10000,
			"currentConsumerCount" : 5,
			"minConsumerCount" : 1
		},
		"com.example.exampleQueue2" : 
		{
			"status" : "DOWN",
			"currentMessageCount" : 67377,
			"maxMessageCount" : 50000,
			"currentConsumerCount" : 0,
			"minConsumerCount" : 3
		}
	}
}
```


## Metrics

#### Example (Autowires all queue-beans)
```java
@EnableRabbitMetrics
@Configuration
public class RabbitMetricsConfiguration
{
}
```

#### Response ([/actuator/metrics](http://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-metrics.html))
```json
{
  "names" : 
  [
    "jvm.memory.used",
    "process.cpu.usage",
    "...",  
    "rabbitmq.queue.messages.current",
    "rabbitmq.queue.consumers.current",
    "rabbitmq.queue.messages.max",
    "rabbitmq.queue.consumers.min",
    "..."    
  ]
}
```

Detailed:
```json
{
  "name": "rabbitmq.queue.messages.current",
  "description": null,
  "baseUnit": null,
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 215
    }
  ],
  "availableTags" : [ {
    "tag" : "queue",
    "values" : [ "dlq-example-simple-queue-dlq", "dlq-example-simple-queue" ]
  } ]
}
```

Prometheus-Example:
```
# HELP rabbitmq_queue_messages_current  
# TYPE rabbitmq_queue_messages_current gauge
rabbitmq_queue_messages_current{queue="dlq-example-simple-queue",} 0.0
rabbitmq_queue_messages_current{queue="dlq-example-simple-queue-dlq",} 371.0
# HELP rabbitmq_queue_consumers_current  
# TYPE rabbitmq_queue_consumers_current gauge
rabbitmq_queue_consumers_current{queue="dlq-example-simple-queue",} 1.0
rabbitmq_queue_consumers_current{queue="dlq-example-simple-queue-dlq",} 0.0
```
