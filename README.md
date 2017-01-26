# kafka-avro

> Node.js bindings for librdkafka with Avro schema serialization.

The kafka-avro library is a wrapper that combines the [node-rdkafka][node-rdkafka] and [avsc](avsc) libraries to allow for Production and Consumption of messages on kafka validated and serialized by Avro.

## Install

Install the module using NPM:

```
npm install kafka-avro --save
```

## Documentation

The kafka-avro library operates in the following steps:

1. You provide your Kafka Brokers and Schema Registry (SR) Url to a new instance of kafka-avro.
1. You initialize kafka-avro, that will tell the library to query the SR for all registered schemas, evaluate and store them in runtime memory.
1. kafka-avro will then expose the `getConsumer()` and `getProducer()` methods, which both return instsances of the corresponding Constructors from the [node-rdkafka][node-rdkafka] library.

The instances of "node-rdkafka" that are returned by kafka-avro are hacked so as to intercept produced and consumed messages and run them by the Avro de/serializer.

You are highly encouraged to read the ["node-rdkafka" documentation](https://blizzard.github.io/node-rdkafka/current/), as it explains how you can use the Producer and Consumer instances as well as check out the [available configuration options of node-rdkafka](https://github.com/edenhill/librdkafka/blob/2213fb29f98a7a73f22da21ef85e0783f6fd67c4/CONFIGURATION.md)

### Initialize kafka-avro

```js
var KafkaAvro = require('kafka-avro');

var kafkaAvro = new KafkaAvro({
    kafkaBroker: 'localhost:9092',
    schemaRegistry: 'localhost:8081',
});

kafkaAvro.on('log', function(message) {
    console.log(message);
})

// Query the Schema Registry for all topic-schema's
// fetch them and evaluate them.
kafkaAvro.init()
    .then(function() {
        console.log('Ready to use');
    });
```

### Quick Usage Producer

> NOTICE: You need to initialize kafka-avro before you can produce or consume messages.

```js
var producer = kafkaAvro.getProducer({
  // Options listed bellow
});

var topicName = 'test';

//Wait for the ready event before producing
producer.on('ready', function() {
  //Create a Topic object with any options our Producer
  //should use when producing to that topic.
  var topic = producer.Topic(topicName, {
    // Make the Kafka broker acknowledge our message (optional)
    'request.required.acks': 1
  });

  var value = new Buffer('value-' +i);
  var key = 'key';

  // if partition is set to -1, librdkafka will use the default partitioner
  var partition = -1;
  producer.produce(topic, partition, value, key);
});

producer.on('disconnected', function(arg) {
  console.log('producer disconnected. ' + JSON.stringify(arg));
});

//starting the producer
producer.connect();
```

What kafka-avro basically does is wrap around node-rdkafka and intercept the produce method to validate and serialize the message.

* [node-rdkafka Producer Tutorial](https://blizzard.github.io/node-rdkafka/current/tutorial-producer_.html)
* [Full list of Producer's options](https://github.com/edenhill/librdkafka/blob/2213fb29f98a7a73f22da21ef85e0783f6fd67c4/CONFIGURATION.md)

### Quick Usage Consumer

> NOTICE: You need to initialize kafka-avro before you can produce or consume messages.

```js
var Transform = require('stream').Transform;

var consumer = kafkaAvro.getConsumer({
  'group.id': 'librd-test',
  'socket.keepalive.enable': true,
  'enable.auto.commit': true,
});

var topicName = 'test';

var stream = consumer.getReadStream(topicName, {
  waitInterval: 0
});

stream.on('error', function() {
  process.exit(1);
});

consumer.on('error', function(err) {
  console.log(err);
  process.exit(1);
});

stream.on('data', function(message) {
    console.log('Received message:', message);
});
```

Same deal here, thin wrapper around node-rdkafka and deserialize incoming messages before they reach your consuming method.

* [node-rdkafka Consumer Tutorial](https://blizzard.github.io/node-rdkafka/current/tutorial-consumer.html)
* [Full list of Consumer's options](https://github.com/edenhill/librdkafka/blob/2213fb29f98a7a73f22da21ef85e0783f6fd67c4/CONFIGURATION.md)

#### Consumer Data Object

kafka-avro intercepts all incoming messages and augments the object with one more property named `parsed` which contained the avro deserialized object. Here is a breakdown of the properties included in the `message` object you receive when consuming messages:

* `value` **Buffer** The raw message buffer from Kafka.
* `size` **Number** The size of the message.
* `key` **String|Number** Partioning key used.
* `topic` **String** The topic this message comes from.
* `offset` **Number** The Kafka offset.
* `partition` **Number** The kafka partion used.
* `parsed` **Object** The avro deserialized message as a JS Object ltieral.

## Releasing

1. Update the changelog bellow.
1. Ensure you are on master.
1. Type: `grunt release`
    * `grunt release:minor` for minor number jump.
    * `grunt release:major` for major number jump.

## Release History

- **v0.1.0**, *26 Jan 2016*
    - First fully working release.
- **v0.0.1**, *25 Jan 2016*
    - Big Bang

## License

Copyright Waldo, Inc. All rights reserved.

[avsc]: https://github.com/mtth/avsc
[node-rdkafka]: https://github.com/Blizzard/node-rdkafka
