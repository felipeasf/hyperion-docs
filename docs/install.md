# Installation

## nodeos config.ini
```
state-history-dir = "state-history"
trace-history = true
chain-state-history = true
state-history-endpoint = 127.0.0.1:8080
plugin = eosio::state_history_plugin
```

## NodeJS

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## PM2

```bash
sudo npm install pm2@latest -g
sudo pm2 startup
```

## Elasticsearch Installation

* Follow instructions on https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html (Ubuntu/Debian)

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

## Kibana Installation

```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.4.0-amd64.deb
sudo apt install ./kibana-7.4.0-amd64.deb
sudo systemctl enable kibana
```

Open and test Kibana on `http://localhost:5601`

## RabbitMQ Installation

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

## Redis Installation

```bash
sudo apt install redis-server
```

Edit `/etc/redis/redis.conf` and change supervised to `systemd`

```bash
sudo systemctl restart redis.service
```

## Hyperion Indexer

```bash
git clone https://github.com/eosrio/Hyperion-History-API.git
cd Hyperion-History-API
npm install
cp example-ecosystem.config.js ecosystem.config.js
```

Edit `ecosystem.config.js`


## Setup Indices and Aliases

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

