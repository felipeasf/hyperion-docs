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

##### 1. Edit `/etc/elasticsearch/elasticsearch.yml`

```
cluster.name: myCluster
bootstrap.memory_lock: true
```

##### 2. Edit `/etc/elasticsearch/jvm.options`

```
# Set your heap size, avoid allocating more than 31GB, even if you have enought RAM.
# Test on your specific machine by changing -Xmx32g in the following command:
# java -Xmx32g -XX:+UseCompressedOops -XX:+PrintFlagsFinal Oops | grep Oops
-Xms16g
-Xmx16g
```

##### 3. Disable swap

* run `swapoff -a` this will immediately disable swap
* remove any swap entry from /etc/fstab

!!! tip
    Identify configured swap devices and files with cat /proc/swaps

##### 4. Allow memlock
run `sudo systemctl edit elasticsearch` and add the following lines


```
[Service]
LimitMEMLOCK=infinity
```

##### 5. Start elasticsearch and check the logs (verify if the memory lock was successful)

```bash
sudo service elasticsearch start
sudo less /var/log/elasticsearch/myCluster.log
sudo systemctl enable elasticsearch
```

##### 6. Test the REST API

`curl http://localhost:9200`

```json
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

The Default user and password is:
```
user: elastic
password: changeme
```

You can change the password via the API, like this:
```
curl -X POST "localhost:9200/_security/user/elastic/_password?pretty" -H 'Content-Type: application/json' -d'
{
  "password" : "new_password"
}'
```

#### RabbitMQ Installation

!!! info
    Follow the detailed installation instructions on the official [RabbitMQ documentation](https://www.rabbitmq.com/install-debian.html#installation-methods)

##### 1. Enable the WebUI

```bash
sudo rabbitmq-plugins enable rabbitmq_management
```

##### 2. Add vhost
```bash
sudo rabbitmqctl add_vhost /hyperion
```

##### 2. Create a user and password
```bash
sudo rabbitmqctl add_user {my_user} {my_password}
```
##### 3. Set the user as administrator
```bash
sudo rabbitmqctl set_user_tags {my_user} administrator
```

##### 4. Set the user permissions to the vhost
```bash
sudo rabbitmqctl set_permissions -p /hyperion {my_user} ".*" ".*" ".*"
```

##### 5. Check access to the WebUI

[http://localhost:15672](http://localhost:15672)

#### Redis Installation

##### 1. Install
```bash
sudo apt install redis-server
```

##### 2. Edit `/etc/redis/redis.conf`

!!! note
    By default, redis binds to the localhost address. You need to edit
    the config file if you want to listen to other network.

##### 3. Change supervised to `systemd`
```bash
sudo systemctl restart redis.service
```

#### NodeJS

##### 1. Install the nodejs source
```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

##### 2. Install
```bash
sudo apt-get install -y nodejs
```

#### PM2

##### 1. Install
```bash
sudo npm install pm2@latest -g
```

##### 2. Run
```bash
sudo pm2 startup
```

#### Kibana Installation

!!! note
    Check for the latest version on the [official website](https://www.elastic.co/downloads/kibana)

##### 1. Get the binary
```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.4.0-amd64.deb
```
##### 2. install
```bash
sudo apt install ./kibana-7.4.0-amd64.deb
```

##### 3. Change supervised to `systemd`
```bash
sudo systemctl enable kibana
```

##### 4. Open and test Kibana
[http://localhost:5601](http://localhost:5601)

#### nodeos config.ini
```
state-history-dir = "state-history"
trace-history = true
chain-state-history = true
state-history-endpoint = 127.0.0.1:8080
plugin = eosio::state_history_plugin
```

#### Hyperion

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
```json
{
   "amqp":{
      "host":"127.0.0.1:5672",
      "api":"127.0.0.1:15672",
      "user":"my_user",
      "pass":"my_password",
      "vhost":"hyperion"
   },
   "elasticsearch":{
      "host":"127.0.0.1:9200",
      "ingest_nodes":[
         "127.0.0.1:9200"
      ],
      "user":"",
      "pass":""
   },
   "redis":{
      "host":"127.0.0.1",
      "port":"6379"
   },
   "chains":{
      "eos":{
         "name":"EOS Mainnet",
         "chain_id":"aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906",
         "http":"http://127.0.0.1:8888",
         "ship":"ws://127.0.0.1:8080",
         "WS_ROUTER_PORT":7001
      }
   }
}
```
For more details, refer to the [connections section](connections.md)

ecosystem.config.js Reference
```javascript
module.exports = {
    apps: [
        addIndexer('eos'),
        addApiServer('eos', 1)
    ]
};
```
For more details, refer to the [ecosystem section](ecosystem.md)

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

Before starting to index actions into elasticsearch its required to do a ABI scan.
Set the indexer section of the chain/CHAINNAME.config.json as the following: 
 
```json
"indexer":{ 
      "start_on":0,
      "stop_on":0,
      "rewrite":false,
      "purge_queues":true,
      "live_reader":false,
      "live_only_mode":false,
      "abi_scan_mode":true,
      "fetch_block":true,
      "fetch_traces":true,
      "disable_reading":false,
      "disable_indexing":false,
      "process_deltas":true,
      "repair_mode":false
   },
```

Tune your configs to your specific hardware using the following settings:
```json
"scaling":{
      "batch_size":10000,
      "queue_limit":50000,
      "readers":1,
      "ds_queues":1,
      "ds_threads":1,
      "ds_pool_size":1,
      "indexing_queues":1,
      "ad_idx_queues":1,
      "max_autoscale":4,
      "auto_scale_trigger":20000
   },
```
For more details related to the chain configuration please, refer to the [chain section](chain.md)

### Start and Stop

We provide scripts to make simple the process of start and stop you Hyperion Indexer or API instance.
But, you can also do it manually if you prefer. This section will cover both ways.

#### Using run / stop script

You can use `run` script to start the Indexer or the API.
```
./run.sh chain-indexer

./run.sh chain-api
```
Examples:
Start indexing EOS mainnet
```
./run.sh eos-indexer
```
Start EOS API
```
./run.sh eos-api
```
Remember that the chain name was previously defined on the Hyperion section.

The `stop` script follows the same pattern of the `run` script:
```
./stop.sh chain-indexer

./stop.sh chain-api
```

Example:
Stop the EOS mainnet indexer
```
./stop.sh eos-indexer
```

#### Commands

Start indexing
```
pm2 start --only Indexer --update-env
pm2 logs Indexer
```

Stop reading and wait for queues to flush
```
pm2 trigger Indexer stop
```

Force stop
```
pm2 stop Indexer
```

Starting the API node
```
pm2 start --only API --update-env
pm2 logs API
```

### API Reference

API Reference: [API section](api.md)

Example: [OpenAPI Docs](https://eos.hyperion.eosrio.io/v2/docs)
