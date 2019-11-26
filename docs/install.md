# Getting Started

## Installation

### Dependencies

This setup has only been tested with Ubuntu 18.04, but should work with other OS versions too

 - [Elasticsearch 7.4.X](https://www.elastic.co/downloads/elasticsearch)
 - [RabbitMQ](https://www.rabbitmq.com/install-debian.html)
 - [Redis](https://redis.io/topics/quickstart)
 - [Node.js v12](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions)
 - [PM2](http://pm2.keymetrics.io/docs/usage/quick-start/)
 - Nodeos 1.8.4 w/ state_history_plugin and chain_api_plugin

!!! note  
    The indexer requires redis, pm2 and node.js to be on the same machine. Other dependencies might be installed on other machines, preferably over a very high speed and low latency network. Indexing speed will vary greatly depending on this configuration.

#### Elasticsearch Installation

!!! info
    Follow the detailed installation instructions on the official [elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)

Edit `/etc/elasticsearch/elasticsearch.yml`

```
cluster.name: myCluster
bootstrap.memory_lock: true
```

Edit `/etc/elasticsearch/jvm.options`
```
# Set your heap size, avoid allocating more than 31GB, even if you have enought RAM.
# Test on your specific machine by changing -Xmx32g in the following command:
# java -Xmx32g -XX:+UseCompressedOops -XX:+PrintFlagsFinal Oops | grep Oops
-Xms16g
-Xmx16g
```

Disable swap on /etc/fstab

Allow memlock:
run `sudo systemctl edit elasticsearch` and add the following lines


```
[Service]
LimitMEMLOCK=infinity
```

Start elasticsearch and check the logs (verify if the memory lock was successful)

```bash
sudo service elasticsearch start
sudo less /var/log/elasticsearch/myCluster.log
sudo systemctl enable elasticsearch
```

Test the REST API `curl http://localhost:9200`

```
{
  "name" : "ip-172-31-5-121",
  "cluster_name" : "hyperion",
  "cluster_uuid" : "....",
  "version" : {
    "number" : "7.1.0",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "606a173",
    "build_date" : "2019-05-16T00:43:15.323135Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

#### RabbitMQ Installation

`https://www.rabbitmq.com/install-debian.html#installation-methods`

```bash
sudo apt install rabbitmq-server
sudo rabbitmq-plugins enable rabbitmq_management
sudo rabbitmqctl add_vhost /hyperion
sudo rabbitmqctl add_user my_user my_password
sudo rabbitmqctl set_user_tags my_user administrator
sudo rabbitmqctl set_permissions -p /hyperion my_user ".*" ".*" ".*"
```

Check access to the WebUI `http://localhost:15672`

#### Redis Installation

```bash
sudo apt install redis-server
```

Edit `/etc/redis/redis.conf` and change supervised to `systemd`

```bash
sudo systemctl restart redis.service
```

#### NodeJS

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

#### PM2

```bash
sudo npm install pm2@latest -g
sudo pm2 startup
```

#### nodeos config.ini
```
state-history-dir = "state-history"
trace-history = true
chain-state-history = true
state-history-endpoint = 127.0.0.1:8080
plugin = eosio::state_history_plugin
```

#### Kibana Installation

```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.4.0-amd64.deb
sudo apt install ./kibana-7.4.0-amd64.deb
sudo systemctl enable kibana
```

Open and test Kibana on `http://localhost:5601`

!!! note
    Check for the latest version on the [official website](https://www.elastic.co/downloads/kibana)

#### Hyperion Indexer

##### 1. Clone & Install packages
```bash
git clone https://github.com/eosrio/Hyperion-History-API.git
cd Hyperion-History-API
npm install
```

##### 2. Edit configs
```
cp example-ecosystem.config.js ecosystem.config.js
nano ecosystem.config.js

# Enter connection details here (chain name must match on the ecosystem file)
cp example-connections.json connections.json
nano connections.json
```

connections.json Reference
```
{
  "amqp": {
    "host": "127.0.0.1:5672", // RabbitMQ Server
    "api": "127.0.0.1:15672", // RabbitMQ API Endpoint
    "user": "username",
    "pass": "password",
    "vhost": "hyperion" // RabbitMQ vhost
  },
  "elasticsearch": {
    "host": "127.0.0.1:9200", // Elasticsearch HTTP API Endpoint
    "user": "elastic",
    "pass": "password"
  },
  "redis": {
    "host": "127.0.0.1",
    "port": "6379"
  },
  "chains": {
    "eos": { // Chain name (must match on the ecosystem file)
      "http": "http://127.0.0.1:8888", // Nodeos Chain API Endpoint
      "ship": "ws://127.0.0.1:8080" // Nodeos State History Endpoint
    },
    "other_chain": {...}
  }
}
```

ecosystem.config.js Reference
```
CHAIN: 'eos',                          // chain prefix for indexing
ABI_CACHE_MODE: 'false',               // only cache historical ABIs to redis
DEBUG: 'false',                        // debug mode - display extra logs for debugging
LIVE_READER: 'true',                   // enable continuous reading after reaching the head block
FETCH_DELTAS: 'false',                 // read table deltas
CREATE_INDICES: 'v1',                  // index suffix to be created, set to false to use existing aliases
START_ON: 0,                           // start indexing on block (0=disable)
STOP_ON: 0,                            // stop indexing on block  (0=disable)
AUTO_STOP: 0,                          // automatically stop Indexer after X seconds if no more blocks are being processed (0=disable)
REWRITE: 'false',                      // force rewrite the target replay range
PURGE_QUEUES: 'false',                 // clear rabbitmq queues before starting the indexer
BATCH_SIZE: 2000,                      // parallel reader batch size in blocks
QUEUE_THRESH: 8000,                    // queue size limit on rabbitmq
LIVE_ONLY: 'false',                    // only reads realtime data serially
FETCH_BLOCK: 'true',                   // Request full blocks from the state history plugin
FETCH_TRACES: 'true',                  // Request traces from the state history plugin
PREVIEW: 'false',                      // preview mode - prints worker map and exit
DISABLE_READING: 'false',              // completely disable block reading, for lagged queue processing
READERS: 3,                            // parallel state history readers
DESERIALIZERS: 4,                      // deserialization queues
DS_MULT: 4,                            // deserialization threads per queue
ES_IDX_QUEUES: 4,              // elastic indexers per queue
ES_AD_IDX_QUEUES: 2,                      // multiplier for action indexing queues
READ_PREFETCH: 50,                     // Stage 1 prefecth size
BLOCK_PREFETCH: 5,                     // Stage 2 prefecth size
INDEX_PREFETCH: 500,                   // Stage 3 prefetch size
ENABLE_INDEXING: 'true',               // enable elasticsearch indexing
INDEX_DELTAS: 'true',                  // index common table deltas (see delta on definitions/mappings)
INDEX_ALL_DELTAS: 'false'              // index all table deltas (WARNING)
```
 
##### 3. Starting
 
 ```
 pm2 start --only Indexer --update-env
 pm2 logs Indexer
 ```
 
##### 4. Stopping
 
 Stop reading and wait for queues to flush
 ```
 pm2 trigger Indexer stop
 ```
 
 Force stop
 ```
 pm2 stop Indexer
 ```
 
##### 5. Starting the API node
 
 ```
 pm2 start --only API --update-env
 pm2 logs API
 ```

### Setup Indices and Aliases

Load templates first by starting the Hyperion Indexer in preview mode `PREVIEW: 'true'`

Indices and aliases are created automatically using the `CREATE_INDICES` option (set it to your version suffix e.g, v1, v2, v3)
If you want to create them manually, use the commands bellow on the kibana dev console
```
PUT mainnet-action-v1-000001
PUT mainnet-abi-v1-000001
PUT mainnet-block-v1-000001

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "mainnet-abi-v1-000001",
        "alias": "mainnet-abi"
      }
    },
    {
      "add": {
        "index": "mainnet-action-v1-000001",
        "alias": "mainnet-action"
      }
    },
    {
      "add": {
        "index": "mainnet-block-v1-000001",
        "alias": "mainnet-block"
      }
    }
  ]
}
```

Before indexing actions into elasticsearch its required to do a ABI scan pass

Start with
```
ABI_CACHE_MODE: 'true',
FETCH_BLOCK: 'false',
FETCH_TRACES: 'false',
INDEX_DELTAS: 'false',
INDEX_ALL_DELTAS: 'false',
```

When indexing is finished, change the settings back and restart the indexer. In case you do not have much contract updates, you do not need to run a full pass.

Tune your configs to your specific hardware using the following settings:
```
BATCH_SIZE
READERS
DESERIALIZERS
DS_MULT
ES_IDX_QUEUES
ES_AD_IDX_QUEUES
READ_PREFETCH
BLOCK_PREFETCH
INDEX_PREFETCH
```

### API Reference

Documentation is automatically generated by Swagger/OpenAPI.

Example: [OpenAPI Docs](https://eos.hyperion.eosrio.io/v2/docs)
