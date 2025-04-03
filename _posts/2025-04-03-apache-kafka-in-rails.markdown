## Subscribers -- Kafka as Broker -- Consumers

## Gemfile

```
gem 'ruby-kafka', '~> 1.5'  # This is the correct gem to use
gem 'delivery_boy'  # Add this line
gem 'racecar' # for easy consumer setup
```

https://kafka.apache.org/quickstart#quickstart_send
https://deadmanssnitch.com/opensource/kafka/docs/index.html

** docker pull apache/kafka:4.0.0
** docker run -p 9092:9092 apache/kafka:4.0.0

## setup initializers/kafka.rb

## bash - let's create the topic

`kafka-topics --create --topic blog_events --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1`

## create topic manually

```
docker-compose exec kafka \
 kafka-topics --create \
 --topic blog_events \
 --bootstrap-server localhost:9092 \
 --partitions 1 \
 --replication-factor 1
```

`apache/kafka:4.0.0`

`docker exec -it gallant_shockley kafka-topics --list --bootstrap-server localhost:9092`

> OCI runtime exec failed: exec failed: container_linux.go:380: starting container process caused: exec: "kafka-topics": executable file not found in $PATH: unknown

## may be kafka-topics is not there

`docker exec -it gallant_shockley /opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092`

> did not out put any thing

### This means kafka is running successfully.

`docker logs gallant_shockley --tail 50`
`ssl.endpoint.identification.algorithm = https`

## By default, Kafka does not automatically create topics. You need to manually create blog_events:

```
docker exec -it gallant_shockley /opt/kafka/bin/kafka-topics.sh --create \
 --topic blog_events \
 --bootstrap-server localhost:9092 \
 --partitions 1 \
 --replication-factor 1
```

## Error invoking remote method 'docker-start-container': Error: (HTTP code 500) server error - driver failed programming external connectivity on endpoint gallant_shockley (6e5102c8320a31dab372355d2c43cf70ac239e92fc431982756d6933d66b7bec): Bind for 0.0.0.0:9092 failed: port is already allocated

#### rails c -> Blog created and this is output

apache-kafka-demo(dev)> Blog.create(title: 'blog 1', description: 'blog 1 description')
TRANSACTION (0.2ms) BEGIN immediate TRANSACTION /_application='ApacheKafkaDemo'_/
Blog Create (2.2ms) INSERT INTO "blogs" ("title", "description", "created*at", "updated_at") VALUES ('blog 1', 'blog 1 description', '2025-04-02 01:42:33.049807', '2025-04-02 01:42:33.049807') RETURNING "id" /\_application='ApacheKafkaDemo'*/
I, [2025-04-02T07:27:33.055743 #85432] INFO -- : [Producer ] Starting async producer in the background...
TRANSACTION (0.3ms) COMMIT TRANSACTION /_application='ApacheKafkaDemo'_/
=>
#<Blog:0x000000011fc20b58
id: 1,
title: "blog 1",
description: "blog 1 description",
created_at: "2025-04-02 01:42:33.049807000 +0000",
updated_at: "2025-04-02 01:42:33.049807000 +0000">
apache-kafka-demo(dev)> I, [2025-04-02T07:27:43.058547 #85432] INFO -- : [Producer ] New topics added to target list: blog_events
I, [2025-04-02T07:27:43.058967 #85432] INFO -- : [Producer ] Fetching cluster metadata from kafka://localhost:9092
D, [2025-04-02T07:27:43.059640 #85432] DEBUG -- : [Producer ] [topic_metadata] Opening connection to localhost:9092 with client id delivery_boy...
D, [2025-04-02T07:27:43.065307 #85432] DEBUG -- : [Producer ] [topic_metadata] Sending topic_metadata API request 1 to localhost:9092
D, [2025-04-02T07:27:43.065638 #85432] DEBUG -- : [Producer ] [topic_metadata] Waiting for response 1 from localhost:9092
D, [2025-04-02T07:27:43.127048 #85432] DEBUG -- : [Producer ] [topic_metadata] Received response 1 from localhost:9092
I, [2025-04-02T07:27:43.127517 #85432] INFO -- : [Producer ] Discovered cluster metadata; nodes: localhost:9092 (node_id=1)
D, [2025-04-02T07:27:43.127560 #85432] DEBUG -- : [Producer ] Closing socket to localhost:9092
E, [2025-04-02T07:27:43.128119 #85432] ERROR -- : [Producer ] Failed to assign partitions to 1 messages in blog_events
W, [2025-04-02T07:27:43.128314 #85432] WARN -- : [Producer ] Failed to send all messages to ; attempting retry 1 of 2 after 1s
I, [2025-04-02T07:27:44.132206 #85432] INFO -- : [Producer ] Fetching cluster metadata from kafka://localhost:9092
D, [2025-04-02T07:27:44.132823 #85432] DEBUG -- : [Producer ] [topic_metadata] Opening connection to localhost:9092 with client id delivery_boy...
D, [2025-04-02T07:27:44.135533 #85432] DEBUG -- : [Producer ] [topic_metadata] Sending topic_metadata API request 1 to localhost:9092
D, [2025-04-02T07:27:44.135948 #85432] DEBUG -- : [Producer ] [topic_metadata] Waiting for response 1 from localhost:9092
D, [2025-04-02T07:27:44.154177 #85432] DEBUG -- : [Producer ] [topic_metadata] Received response 1 from localhost:9092
I, [2025-04-02T07:27:44.154276 #85432] INFO -- : [Producer ] Discovered cluster metadata; nodes: localhost:9092 (node_id=1)
D, [2025-04-02T07:27:44.154299 #85432] DEBUG -- : [Producer ] Closing socket to localhost:9092
D, [2025-04-02T07:27:44.154541 #85432] DEBUG -- : [Producer ] Current leader for blog_events/0 is node localhost:9092 (node_id=1)
I, [2025-04-02T07:27:44.154591 #85432] INFO -- : [Producer ] Sending 1 messages to localhost:9092 (node_id=1)
D, [2025-04-02T07:27:44.154690 #85432] DEBUG -- : [Producer ] [produce] Opening connection to localhost:9092 with client id delivery_boy...
D, [2025-04-02T07:27:44.155944 #85432] DEBUG -- : [Producer ] [produce] Sending produce API request 1 to localhost:9092
D, [2025-04-02T07:27:44.790992 #85432] DEBUG -- : [Producer ] [produce] Waiting for response 1 from localhost:9092
D, [2025-04-02T07:27:44.836204 #85432] DEBUG -- : [Producer ] [produce] Received response 1 from localhost:9092
D, [2025-04-02T07:27:44.836494 #85432] DEBUG -- : [Producer ] Successfully appended 1 messages to blog_events/0 on localhost:9092 (node_id=1)
apache-kafka-demo(dev)>

- Above output indicates
- The blog record was successfully created in your database (INSERT INTO blogs)
- Kafka Connection:
  Your producer connected to Kafka at localhost:9092
  Discovered the cluster metadata (single node)
  Confirmed the blog_events topic exists
- Message Delivery:
  After 2 retries (due to initial partition assignment failure), the event was successfully published to blog_events/0 (partition 0)

- The Error Sequence
  First Attempt Failed:
  Failed to assign partitions to 1 messages in blog_events
  This is normal when Kafka is still initializing or when the producer first discovers the cluster

- Automatic Recovery:
  The producer retried after 1 second (as configured)
  On the second attempt:

  - Re-fetched cluster metadata
  - Identified the leader for blog_events/0
  - Successfully delivered the message

- Why This Happened
  Normal Kafka Behavior:
  Partition assignment can fail temporarily during:
  Broker startup
  Topic creation
  Network latency

- Your Configuration is Correct:
  The system self-healed because:
  Retries were enabled (default in ruby-kafka)
  The topic existed (created earlier via kafka-topics.sh)

- How to Verify the Message  
  ➜ apache_kafka_demo git:(main) ✗ docker logs my_kafka | grep "blog_events"
  [2025-04-02 01:42:43,114] INFO Sent auto-creation request for Set(blog_events) to the active controller. (kafka.server.DefaultAutoTopicCreationManager)
  [2025-04-02 01:42:43,173] INFO [ReplicaFetcherManager on broker 1] Removed fetcher for partitions Set(blog_events-0) (kafka.server.ReplicaFetcherManager)
  [2025-04-02 01:42:43,189] INFO [LogLoader partition=blog_events-0, dir=/tmp/kraft-combined-logs] Loading producer state till offset 0 (org.apache.kafka.storage.internals.log.UnifiedLog)
  [2025-04-02 01:42:43,192] INFO Created log for partition blog_events-0 in /tmp/kraft-combined-logs/blog_events-0 with properties {} (kafka.log.LogManager)
  [2025-04-02 01:42:43,193] INFO [Partition blog_events-0 broker=1] No checkpointed highwatermark is found for partition blog_events-0 (kafka.cluster.Partition)
  [2025-04-02 01:42:43,194] INFO [Partition blog_events-0 broker=1] Log loaded for partition blog_events-0 with initial high watermark 0 (kafka.cluster.Partition)

## ^^ This tells:

Kafka Logs Confirm Success:
The blog_events topic was auto-created when first used
Partition blog_events-0 was initialized with offset 0
All metadata was properly registered

➜ apache_kafka_demo git:(main) ✗ docker exec -it my_kafka \
 /opt/kafka/bin/kafka-console-consumer.sh \
 --topic blog_events \
 --from-beginning \
 --bootstrap-server localhost:9092

{"event_id":"4ebd836d-fab7-4958-90e1-f116661c7059","event_type":"blog_created","event_time":"2025-04-02T01:42:33Z","data":{"id":1,"title":"blog 1","description":"blog 1 description","created_at":"2025-04-02T01:42:33.049Z"}}

## This tells ^^

Message Verified:

    Your blog creation event was successfully published to Kafka

    The consumer shows the complete JSON message with:

    {
      "event_id": "4ebd836d-fab7-4958-90e1-f116661c7059",
      "event_type": "blog_created",
      "data": {
        "id": 1,
        "title": "blog 1",
        "description": "blog 1 description"
      }
    }

# config/initializers/kafka.rb

```
DeliveryBoy.configure do |config|
  config.delivery_interval = 1  # Batch messages for 1 second
  config.delivery_threshold = 5 # Or batch 5 messages
  config.max_retries = 3        # Increase retries
end
```

## Create Blog model title:string, description:text

## Subscribers

See logs

```
docker logs my_kafka | grep "blog_events"
```

output like this

```
[2025-04-02 01:42:43,114] INFO Sent auto-creation request for Set(blog_events) to the active controller. (kafka.server.DefaultAutoTopicCreationManager)
[2025-04-02 01:42:43,173] INFO [ReplicaFetcherManager on broker 1] Removed fetcher for partitions Set(blog_events-0) (kafka.server.ReplicaFetcherManager)
[2025-04-02 01:42:43,189] INFO [LogLoader partition=blog_events-0, dir=/tmp/kraft-combined-logs] Loading producer state till offset 0 (org.apache.kafka.storage.internals.log.UnifiedLog)
[2025-04-02 01:42:43,192] INFO Created log for partition blog_events-0 in /tmp/kraft-combined-logs/blog_events-0 with properties {} (kafka.log.LogManager)
[2025-04-02 01:42:43,193] INFO [Partition blog_events-0 broker=1] No checkpointed highwatermark is found for partition blog_events-0 (kafka.cluster.Partition)
[2025-04-02 01:42:43,194] INFO [Partition blog_events-0 broker=1] Log loaded for partition blog_events-0 with initial high watermark 0 (kafka.cluster.Partition)
```

```
docker exec -it my_kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --topic blog_events \
  --from-beginning \
  --bootstrap-server localhost:9092
```

```
{"event_id":"4ebd836d-fab7-4958-90e1-f116661c7059","event_type":"blog_created","event_time":"2025-04-02T01:42:33Z","data":{"id":1,"title":"blog 1","description":"blog 1 description","created_at":"2025-04-02T01:42:33.049Z"}}
docker exec -it my_kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --topic blog_events \
  --from-beginning \
  --bootstrap-server localhost:9092{"event_id":"4ebd836d-fab7-4958-90e1-f116661c7059","event_type":"blog_created","event_time":"2025-04-02T01:42:33Z","data":{"id":1,"title":"blog 1","description":"blog 1 description","created_at":"2025-04-02T01:42:33.049Z"}}
```

To see the logs of the kafka, run the command `docker logs container_id`

`docker logs gallant_shockley`

output looks like this

```
===> User
uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)
===> Setting default values of environment variables if not already set.
CLUSTER_ID not set. Setting it to default value: "5L6g3nShT-eMCtK--X86sw"
===> Configuring ...
===> Launching ...
===> Using provided cluster id 5L6g3nShT-eMCtK--X86sw ...
[0.001s][warning][cds] The shared archive file has a bad magic number: 0
[2025-04-02 00:45:09,555] INFO Registered kafka:type=kafka.Log4jController MBean (kafka.utils.Log4jControllerRegistration$)
[2025-04-02 00:45:09,743] INFO Registered signal handlers for TERM, INT, HUP (org.apache.kafka.common.utils.LoggingSignalHandler)
[2025-04-02 00:45:09,744] INFO [ControllerServer id=1] Starting controller (kafka.server.ControllerServer)
[2025-04-02 00:45:09,933] INFO Updated connection-accept-rate max connection creation rate to 2147483647 (kafka.network.ConnectionQuotas)
[2025-04-02 00:45:09,957] INFO [SocketServer listenerType=CONTROLLER, nodeId=1] Created data-plane acceptor and processors for endpoint : ListenerName(CONTROLLER) (kafka.network.SocketServer)
[2025-04-02 00:45:09,969] INFO CONTROLLER: resolved wildcard host to d3cc737bfffc (org.apache.kafka.metadata.ListenerInfo)
[2025-04-02 00:45:09,973] INFO authorizerStart completed for endpoint CONTROLLER. Endpoint is now READY. (org.apache.kafka.server.network.EndpointReadyFutures)
[2025-04-02 00:45:09,974] INFO [SharedServer id=1] Starting SharedServer (kafka.server.SharedServer)
[2025-04-02 00:45:10,009] INFO [LogLoader partition=__cluster_metadata-0, dir=/tmp/kraft-combined-logs] Loading producer state till offset 0 (org.apache.kafka.storage.internals.log.UnifiedLog)
[2025-04-02 00:45:10,009] INFO [LogLoader partition=__cluster_metadata-0, dir=/tmp/kraft-combined-logs] Reloading from producer snapshot and rebuilding producer state from offset 0 (org.apache.kafka.storage.internals.log.UnifiedLog)
[2025-04-02 00:45:10,012] INFO [LogLoader partition=__cluster_metadata-0, dir=/tmp/kraft-combined-logs] Producer state recovery took 0ms for snapshot load and 0ms for segment recovery from offset 0 (org.apache.kafka.storage.internals.log.UnifiedLog)
[2025-04-02 00:45:10,030] INFO Initialized snapshots with IDs SortedSet() from /tmp/kraft-combined-logs/__cluster_metadata-0 (kafka.raft.KafkaMetadataLog$)
[2025-04-02 00:45:10,040] INFO [raft-expiration-reaper]: Starting (kafka.raft.TimingWheelExpirationService$ExpiredOperationReaper)
[2025-04-02 00:45:10,048] INFO [RaftManager id=1] Reading KRaft snapshot and log as part of the initialization (org.apache.kafka.raft.KafkaRaftClient)
[2025-04-02 00:45:10,050] INFO [RaftManager id=1] Starting voters are VoterSet(voters={1=VoterNode(voterKey=ReplicaKey(id=1, directoryId=<undefined>), listeners=Endpoints(endpoints={ListenerName(CONTROLLER)=localhost/127.0.0.1:9093}), supportedKRaftVersion=SupportedVersionRange[min_version:0, max_version:0])}) (org.apache.kafka.raft.KafkaRaftClient)
[2025-04-02 00:45:10,050] INFO [RaftManager id=1] Starting request manager with static voters: [localhost:9093 (id: 1 rack: null isFenced: false)] (org.apache.kafka.raft.KafkaRaftClient)
```

blog.rb model

```
# app/models/blog.rb
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

blog_consumer.rb

```
# app/consumers/blog_consumer.rb
class BlogConsumer < Racecar::Consumer
  subscribes_to "blog_events"

  def process(message)
    event = JSON.parse(message.value)
    # Same processing logic as above
  end
end
```


## Once blog is created, event is pushed to kafka broker.

apache-kafka-demo(dev)> Blog.create(title: 'blog 1', description: 'blog 1 description')
  TRANSACTION (0.2ms)  BEGIN immediate TRANSACTION /*application='ApacheKafkaDemo'*/
  Blog Create (2.2ms)  INSERT INTO "blogs" ("title", "description", "created_at", "updated_at") VALUES ('blog 1', 'blog 1 description', '2025-04-02 01:42:33.049807', '2025-04-02 01:42:33.049807') RETURNING "id" /*application='ApacheKafkaDemo'*/
I, [2025-04-02T07:27:33.055743 #85432]  INFO -- : [Producer ] Starting async producer in the background...
  TRANSACTION (0.3ms)  COMMIT TRANSACTION /*application='ApacheKafkaDemo'*/
=>
#<Blog:0x000000011fc20b58
 id: 1,
 title: "blog 1",
 description: "blog 1 description",
 created_at: "2025-04-02 01:42:33.049807000 +0000",
 updated_at: "2025-04-02 01:42:33.049807000 +0000">
apache-kafka-demo(dev)> I, [2025-04-02T07:27:43.058547 #85432]  INFO -- : [Producer ] New topics added to target list: blog_events
I, [2025-04-02T07:27:43.058967 #85432]  INFO -- : [Producer ] Fetching cluster metadata from kafka://localhost:9092
D, [2025-04-02T07:27:43.059640 #85432] DEBUG -- : [Producer ] [topic_metadata] Opening connection to localhost:9092 with client id delivery_boy...
D, [2025-04-02T07:27:43.065307 #85432] DEBUG -- : [Producer ] [topic_metadata] Sending topic_metadata API request 1 to localhost:9092
D, [2025-04-02T07:27:43.065638 #85432] DEBUG -- : [Producer ] [topic_metadata] Waiting for response 1 from localhost:9092
D, [2025-04-02T07:27:43.127048 #85432] DEBUG -- : [Producer ] [topic_metadata] Received response 1 from localhost:9092
I, [2025-04-02T07:27:43.127517 #85432]  INFO -- : [Producer ] Discovered cluster metadata; nodes: localhost:9092 (node_id=1)
D, [2025-04-02T07:27:43.127560 #85432] DEBUG -- : [Producer ] Closing socket to localhost:9092
E, [2025-04-02T07:27:43.128119 #85432] ERROR -- : [Producer ] Failed to assign partitions to 1 messages in blog_events
W, [2025-04-02T07:27:43.128314 #85432]  WARN -- : [Producer ] Failed to send all messages to ; attempting retry 1 of 2 after 1s
I, [2025-04-02T07:27:44.132206 #85432]  INFO -- : [Producer ] Fetching cluster metadata from kafka://localhost:9092
D, [2025-04-02T07:27:44.132823 #85432] DEBUG -- : [Producer ] [topic_metadata] Opening connection to localhost:9092 with client id delivery_boy...
D, [2025-04-02T07:27:44.135533 #85432] DEBUG -- : [Producer ] [topic_metadata] Sending topic_metadata API request 1 to localhost:9092
D, [2025-04-02T07:27:44.135948 #85432] DEBUG -- : [Producer ] [topic_metadata] Waiting for response 1 from localhost:9092
D, [2025-04-02T07:27:44.154177 #85432] DEBUG -- : [Producer ] [topic_metadata] Received response 1 from localhost:9092
I, [2025-04-02T07:27:44.154276 #85432]  INFO -- : [Producer ] Discovered cluster metadata; nodes: localhost:9092 (node_id=1)
D, [2025-04-02T07:27:44.154299 #85432] DEBUG -- : [Producer ] Closing socket to localhost:9092
D, [2025-04-02T07:27:44.154541 #85432] DEBUG -- : [Producer ] Current leader for blog_events/0 is node localhost:9092 (node_id=1)
I, [2025-04-02T07:27:44.154591 #85432]  INFO -- : [Producer ] Sending 1 messages to localhost:9092 (node_id=1)
D, [2025-04-02T07:27:44.154690 #85432] DEBUG -- : [Producer ] [produce] Opening connection to localhost:9092 with client id delivery_boy...
D, [2025-04-02T07:27:44.155944 #85432] DEBUG -- : [Producer ] [produce] Sending produce API request 1 to localhost:9092
D, [2025-04-02T07:27:44.790992 #85432] DEBUG -- : [Producer ] [produce] Waiting for response 1 from localhost:9092
D, [2025-04-02T07:27:44.836204 #85432] DEBUG -- : [Producer ] [produce] Received response 1 from localhost:9092
D, [2025-04-02T07:27:44.836494 #85432] DEBUG -- : [Producer ] Successfully appended 1 messages to blog_events/0 on localhost:9092 (node_id=1)

## ---------------------- Consumer -------------------- ##
## Run BlogConsumer -- log event in BlogAudit

bundle exec racecar BlogConsumer

###### In production (including Heroku) you must run bundle exec racecar BlogConsumer continuously as a separate process (not a scheduler). This is because:

    Kafka consumers are long-running processes (like a web server)

    They maintain persistent connections to Kafka brokers

    They process messages in real-time as they arrive

## How to Handle This in Production (Heroku)

### Option 1: Heroku Worker Dyno (Recommended)

#### 1. Add to your Procfile:

web: bundle exec rails server
worker: bundle exec racecar BlogConsumer

#### Scale the worker:

heroku ps:scale worker=1

#### 2. Sidekiq/Scheduler (Not Recommended ❌)

```
# Bad approach - don't do this!
# This will miss messages between scheduler runs
class ScheduledConsumerJob
  def perform
    system("bundle exec racecar BlogConsumer --timeout 10")
  end
end
```

#### 3. Kubernetes/ECS

For containerized setups:

```
# docker-compose.yml
services:
  consumer:
    command: bundle exec racecar BlogConsumer
    depends_on: [kafka]
```

### Best Practices for Heroku

1. Auto-restart: Heroku will restart crashed consumers

2. Scaling:

```
heroku ps:scale worker=2  # Add more consumers
```

3. Logging:

```
heroku logs --tail --ps worker
```

4. Pricing: Worker dynos cost like web dynos (~$25/month for Hobby tier)

## What If You Stop the Consumer?

    Messages will accumulate in Kafka (no data loss)

    When restarted, it processes from the last committed offset

    Use this to pause/upgrade consumers safely

## Alternatives If You Prefer Schedulers

If you truly want periodic processing:

1. Use Kafka Consumer Groups with short timeouts

`bundle exec racecar BlogConsumer --timeout 300  # Exit after 5min`

2. Then run via Heroku Scheduler:

`bash -c "while true; do bundle exec racecar BlogConsumer --timeout 300; done"`
But this is not recommended for real-time systems.

### Example: Full Heroku Setup

1. Add Redis/Kafka addons:

```
heroku addons:create heroku-kafka:basic-0
heroku addons:create heroku-redis:mini
```

2. Update config/racecar.yml:

```
production:
  brokers: <%= ENV["KAFKA_URL"] %>
  group_id: "blog_events_#{Rails.env}"
```

![CircleCI]({{ "/assets/img/kafka/blog_create_event_publish.png" | relative_url }})
![CircleCI]({{ "/assets/img/kafka/blog_create_event_publish_1.png" | relative_url }})