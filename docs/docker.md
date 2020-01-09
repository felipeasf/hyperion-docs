# Hyperion Docker

## Dependencies
- `docker` and `docker-compose`

## INSTALL
1. Edit connections.json file and change http://host.docker.internal:8888 and ws://host.docker.internal:8080
for the nodeos address
2. Change passwords in docker-compose.yml file
3. Run `docker-compose up`

## Troubleshooting
If you're having problems accesing Kibana or using elasticsearch api, you could disable the xpack security
on the docker-compose.yml setting it to false:

```
xpack.security.enabled=false
```