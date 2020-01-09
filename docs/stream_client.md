# Hyperion Stream Client

Live Streaming client for Hyperion History API

## Instalation
1. Clone this repository
2. Run npm install
3. Configure the client
4. Run the client

## Configuration
### 1. Connection

Setup the endpoint that you want to fetch data

```
const client = new Hyperion({endpoint}}, {
        async: false
    });
```

{Explain async!!!}

Example:

```
const client = new Hyperion('https://daobet.eosrio.io'}, {
        async: false
    });
```

When you successfully connects to the hyperion, you'll receive a handshake message:

`{ event: 'handshake', chain: '{chain_name}' }`

### 2. Stream Data Structure

Setup the data structure to be streamed

 - `contract` - contract account
 - `action` - action name
 - `account` - account name
 - `start_from` - start reading on block (0=disable)
 - `read_until` - stop reading on block  (0=disable)
 - `filters` - actions filter (more details below)
 
Example:

```
client.streamActions({
            contract: 'eosio.token',
            action: 'transfer',
            account: 'eosriobrazil',
            start_from: 1,
            read_until: 0,
            filters: [],
        });
``` 

#### 2. Filters
You can setup filters to refine your stream. For example, to filter the stream above for
every transfers made to eosriobrazil account:

```
client.streamActions({
            contract: 'eosio.token',
            action: 'transfer',
            account: 'eosriobrazil',
            start_from: 1,
            read_until: 0,
            filters: [
                {field: '@transfer.to', value: 'eosriobrazil'}
            ],
        });
``` 

To refine even more your stream, you could add more filters. Remember that adding more filters
will result in a AND operation, by now it's not possible to make OR operations with filters.

Example:

```
client.streamActions({
            contract: 'eosio.token',
            action: 'transfer',
            account: 'eosriobrazil',
            start_from: 1,
            read_until: 0,
            filters: [
                {field: '@transfer.from', value: 'eosio'}
                {field: '@transfer.to', value: 'eosriobrazil'}
            ],
        });
``` 
{ADICIONAR LISTA DE FILTROS??}

#### 3. Handling Data
```
client.onData = async (data, ack) => {
    {code here}
}
```

Example:

```
client.onData = async (data, ack) => {
        const act = data['act'];

        console.log(`\n---- ACTION - ${data['global_sequence']} | BLOCK: ${data['block_num']} | ${act.account}::${act.name}`);
        for (const key in act.data) {
            if (act.data.hasOwnProperty(key)) {
                console.log(`${key} = ${act.data[key]}`);
            }
        }
```