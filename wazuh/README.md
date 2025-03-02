# Deploy Wazuh Docker in single node configuration

This deployment is defined in the `docker-compose.yml` file with one Wazuh manager containers, one Wazuh indexer containers, and one Wazuh dashboard container. It can be deployed by following these steps: 

1) Increase max_map_count on your host (Linux). This command must be run with root permissions:
```
$ sysctl -w vm.max_map_count=262144
```
2) Run the certificate creation script:
```
$ docker-compose -f generate-indexer-certs.yml run --rm generator
```
3) Start the environment with docker-compose:

- In the foregroud:
```
$ docker-compose up
```
- In the background:
```
$ docker-compose up -d
```

The environment takes about 1 minute to get up (depending on your Docker host) for the first time since Wazuh Indexer must be started for the first time and the indexes and index patterns must be generated.

### Traefik issues
- it took me a long time to find how to put this behind traefik. Everything was configured, but I could only access it via ip, not the dns name. Fixed by commenting the following in this file: ./config/wazuh_dashboard/opensearch_dashboards.yml
    #server.ssl.enabled: true
    #server.ssl.key: "/usr/share/wazuh-dashboard/certs/wazuh-dashboard-key.pem"
    #server.ssl.certificate: "/usr/share/wazuh-dashboard/certs/wazuh-dashboard.pem"
