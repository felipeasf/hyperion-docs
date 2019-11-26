# Action Data Structure (work in progress)

 - `@timestamp` - block time
 - `global_sequence` - unique action global_sequence, used as index id
 - `parent` - points to the parent action (in the case of an inline action) or equal to 0 if root level
 - `block_num` - block number where the action was processed
 - `trx_id` - transaction id
 - `producer` - block producer
 - `act`
    - `account` - contract account
    - `name` - contract method name
    - `authorization` - array of signers
        - `actor` - signing actor
        - `permission` - signing permission
    - `data` - action data input object
 - `account_ram_deltas` - array of ram deltas and payers
    - `account`
    - `delta`
 - `notified` - array of accounts that were notified (via inline action events)