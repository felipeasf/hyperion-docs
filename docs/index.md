# Hyperion History API
Scalable Full History API Solution for EOSIO based blockchains

Made with ♥ by [EOS Rio](https://eosrio.io/)

## Introducing an storage-optimized action format for EOSIO

The original *history_plugin* bundled with eosio, that provided the v1 api, stored inline action traces nested inside their root actions. This led to an excessive amount of data being stored and also transferred whenever a user requested the action history for a given account. Also inline actions are used as a "event" mechanism to notify parties on a transaction. Based on those Hyperion implements some changes

1. actions are stored in a flattened format
2. a parent field is added to the inline actions to point to the parent global sequence
3. if the inline action data is identical to the parent it is considered a notification and thus removed from the database
4. no blocks or transaction data is stored, all information can be reconstructed from actions

With those changes the API format focus on delivering faster search times, lower bandwidth overhead and easier usability for UI/UX developers. 