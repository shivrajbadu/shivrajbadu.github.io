---
layout: post
title: "The Complete Guide to Apache Kafka in Rails: From Development to Production"
date: 2025-04-03 03:20:18 +0545
categories: [ApacheKafka]
tags: [ApacheKafka]
---

# What is Apache Kafka?

Apache Kafka is a distributed event streaming platform designed for high-throughput, fault-tolerant, and real-time data processing. It is used for:

- Messaging System:

Acts as a message broker between producers (data sources) and consumers (data processors).

- Event Streaming:

Allows applications to publish, subscribe to, store, and process event streams in real time.

- Decoupling Services:

Helps in building microservices architectures by enabling loosely coupled communication between services.

- Data Pipeline:

Used for collecting and processing large volumes of data before sending it to databases, analytics tools, or machine learning models

Kafka is widely used by large-scale applications for real-time analytics, monitoring, and log processing.

## Introduction

Apache Kafka has become the backbone of modern event-driven Rails applications. This guide provides a complete implementation walkthrough with real-world examples, troubleshooting tips, and production-ready patterns. We'll cover:

- Docker-based Kafka setup
- Rails integration with best practices
- Real-world event publishing
- Robust consumer implementation
- Production deployment on Heroku
- Detailed troubleshooting

## Subscribers --> Kafka as message Broker --> Consumers

## Gemfile

```
gem 'ruby-kafka', '~> 1.5'  # This is the correct gem to use
gem 'delivery_boy'  # Add this line
gem 'racecar' # for easy consumer setup
```

https://kafka.apache.org/quickstart#quickstart_send
https://deadmanssnitch.com/opensource/kafka/docs/index.html

## Pull Kafka Docker Image

```
docker pull apache/kafka:4.0.0
```

## Run Kafka Container

```
docker run -p 9092:9092 apache/kafka:4.0.0
```

## Setup Initializers

config/initializers/kafka.rb

```
require 'kafka'  # This now loads ruby-kafka

KAFKA = Kafka.new(
  seed_brokers: ['localhost:9092'],  # Note the parameter name
  client_id: 'blog_app',
  logger: Rails.logger
)
```

## Creating Kafka Topics
### Bash Command to Create Topic

```
kafka-topics --create --topic blog_events --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

## Create Topic Manually via Docker

```
docker-compose exec kafka \
 kafka-topics --create \
 --topic blog_events \
 --bootstrap-server localhost:9092 \
 --partitions 1 \
 --replication-factor 1
```

## Check Topics

```
docker exec -it gallant_shockley kafka-topics --list --bootstrap-server localhost:9092
```

## If kafka-topics is Missing

```
docker exec -it gallant_shockley /opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

## If the command does not output any errors, Kafka is running successfully

## Check Kafka Logs

```
docker logs gallant_shockley --tail 50
```

## Create blog_events Topic if Missing

```
docker exec -it gallant_shockley /opt/kafka/bin/kafka-topics.sh --create \
 --topic blog_events \
 --bootstrap-server localhost:9092 \
 --partitions 1 \
 --replication-factor 1
```

# Debugging Kafka Issues

## Port Already in Use Error

#### If you encounter:

```
Error: (HTTP code 500) server error - driver failed programming external connectivity on endpoint gallant_shockley
```

##### Check which process is using port 9092:

```
lsof -i :9092  # Shows which process is using 9092
kill -9 <PID>  # Replace <PID> with the actual process ID
```

##### Restart Kafka:

```
docker restart gallant_shockley
```

##### Verify Topic Creation

```
docker exec -it gallant_shockley /opt/kafka/bin/kafka-topics.sh --describe --topic blog_events --bootstrap-server localhost:9092
```

##### Check Kafka Listeners

```
docker exec -it gallant_shockley sh -c "cat /opt/kafka/config/server.properties | grep listeners"
```

##### If the output is:

```
advertised.listeners=PLAINTEXT://localhost:9092
```

##### Change it to:

```
advertised.listeners=PLAINTEXT://<your_host_ip>:9092
```

##### Restart Kafka:

```
docker restart gallant_shockley
```

### Rails Model Integration

```
class Blog < ApplicationRecord
  after_create :publish_creation_event

  private

  def publish_creation_event
    event = {
      event_id: SecureRandom.uuid,
      event_type: 'blog_created',
      event_time: Time.now.utc.iso8601,
      data: {
        id: id,
        title: title,
        description: description,
        created_at: created_at
      }
    }

    # Asynchronously publish the event
    KafkaDeliveryBoy.deliver_async(
      event.to_json,
      topic: 'blog_events'
    )
  end
end
```

### Final Steps

1. Kill any process using port 9092 (lsof -i :9092).
2. Ensure the topic exists (kafka-topics.sh --list).
3. Check Kafka listener settings (cat server.properties).
4. Restart Kafka (docker restart gallant_shockley).
