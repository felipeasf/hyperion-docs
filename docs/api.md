# API Reference

***

### /v1/history/get_actions

#### POST
##### Summary:

get actions

##### Description:

legacy get actions query

##### Responses

| Code | Description |
| ---- | ----------- |
| 200  |             |       

##### Example
```
curl -X POST "https://eos.hyperion.eosrio.io/v1/history/get_actions"
```

### /v1/history/get_controlled_accounts

#### POST
##### Summary:

get controlled accounts by controlling accounts

##### Description:

get controlled accounts by controlling accounts

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 |  |

### /v1/history/get_key_accounts

#### POST
##### Summary:

get accounts by public key

##### Description:

get accounts by public key

##### Responses

##### Example
```
curl -X POST "https://eos.hyperion.eosrio.io/v1/history/get_actions"
```

| Code | Description |
| ---- | ----------- |
| 200 |  |

### /v1/history/get_transaction

#### POST
##### Summary:

get transaction by id

##### Description:

get all actions belonging to the same transaction

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

##### Example
```
curl -X POST "https://eos.hyperion.eosrio.io/v1/history/get_transaction"
```

### /v1/chain/get_block

#### POST
##### Summary:

Returns an object containing various details about a specific block on the blockchain.

##### Description:

Returns an object containing various details about a specific block on the blockchain.

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

##### Example
```
curl -X POST "https://api.eossweden.org/v1/chain/get_block" -d '{"block_num_or_id": "1000"}'
```

### /v2/history/get_abi_snapshot

#### GET
##### Summary:

fetch abi at specific block

##### Description:

fetch contract abi at specific block

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| contract | query | contract account | Yes | string |
| block | query | target block | No | integer |
| fetch | query | should fetch the ABI | No | boolean |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

### /v2/history/get_actions

#### GET
##### Summary:

get root actions

##### Description:

get actions based on notified account. this endpoint also accepts generic filters based on indexed fields (e.g. act.authorization.actor=eosio or act.name=delegatebw), if included they will be combined with a AND operator

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| account | query | notified account | No | string |
| track | query | total results to track (count) [number or true] | No | string |
| filter | query | code:name filter | No | string |
| skip | query | skip [n] actions (pagination) | No | integer |
| limit | query | limit of [n] actions per page | No | integer |
| sort | query | sort direction | No | string |
| after | query | filter after specified date (ISO8601) | No | string |
| before | query | filter before specified date (ISO8601) | No | string |
| simple | query | simplified output mode | No | boolean |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 |  |

### /v2/history/get_blocks

#### GET
##### Summary:

get block range

##### Description:

get block range

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| from | query | starting block | No | integer |
| to | query | last block | No | integer |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

### /v2/history/get_created_accounts

#### GET
##### Summary:

get created accounts

##### Description:

get all accounts created by one creator

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| account | query | creator account | No | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 |  |

### /v2/history/get_creator

#### GET
##### Summary:

get account creator

##### Description:

get account creator

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| account | query | created account | No | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 |  |

### /v2/history/get_deltas

#### GET
##### Summary:

get state deltas

##### Description:

get state deltas

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| code | query | contract account | No | string |
| scope | query | table scope | No | string |
| table | query | table name | No | string |
| payer | query | payer account | No | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

### /v2/history/get_transacted_accounts

#### GET
##### Summary:

get interactions based on transfers

##### Description:

get all account that interacted with the source account provided

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| account | query | source account | Yes | string |
| symbol | query | token symbol | No | string |
| contract | query | token contract | No | string |
| direction | query | search direction | Yes | string |
| min | query | minimum value | No | number |
| max | query | maximum value | No | number |
| limit | query | query limit | No | number |
| after | query | filter after specified date (ISO8601) or block number | No | string |
| before | query | filter before specified date (ISO8601) or block number | No | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

### /v2/history/get_transaction

#### GET
##### Summary:

get transaction by id

##### Description:

get all actions belonging to the same transaction

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| id | query | transaction id | Yes | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

### /v2/history/get_transfers

#### GET
##### Summary:

get token transfers

##### Description:

get token transfers utilizing the eosio.token standard

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| from | query | source account | No | string |
| to | query | destination account | No | string |
| symbol | query | token symbol | No | string |
| contract | query | token contract | No | string |
| skip | query | skip [n] actions (pagination) | No | integer |
| limit | query | limit of [n] actions per page | No | integer |
| after | query | filter after specified date (ISO8601) | No | dateTime |
| before | query | filter before specified date (ISO8601) | No | dateTime |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 |  |

### /v2/state/get_account

#### GET
##### Summary:

get account summary

##### Description:

get account data

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| account | query | account name | No | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

### /v2/state/get_key_accounts

#### GET
##### Summary:

get accounts by public key

##### Description:

get accounts by public key

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| public_key | query | public key | Yes | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 |  |

#### POST
##### Summary:

get accounts by public key

##### Description:

get accounts by public key

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 |  |

### /v2/state/get_proposals

#### GET
##### Summary:

get proposals

##### Description:

get proposals

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| proposer | query | filter by proposer | No | string |
| proposal | query | filter by proposal name | No | string |
| account | query | filter by either requested or provided account | No | string |
| requested | query | filter by requested account | No | string |
| provided | query | filter by provided account | No | string |
| executed | query | filter by execution status | No | boolean |
| track | query | total results to track (count) [number or true] | No | string |
| skip | query | skip [n] actions (pagination) | No | integer |
| limit | query | limit of [n] actions per page | No | integer |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

### /v2/state/get_tokens

#### GET
##### Summary:

get tokens from account

##### Description:

get tokens from account

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| account | query | account | Yes | string |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 |  |

### /v2/state/get_voters

#### GET
##### Summary:

get voters

##### Description:

get voters

##### Parameters

| Name | Located in | Description | Required | Schema |
| ---- | ---------- | ----------- | -------- | ---- |
| producer | query | filter by voted producer (comma separated) | No | string |
| skip | query | skip [n] actions (pagination) | No | integer |
| limit | query | limit of [n] actions per page | No | integer |

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

### /v2/health

#### GET
##### Summary:

API Service Health Report

##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | Default Response |

### /stream-client.js

#### GET
##### Responses

| Code | Description |
| ---- | ----------- |
| 200 | default response |
