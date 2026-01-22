# Deploy Wazuh Docker in single node configuration

This deployment is defined in the `compose.yaml` file with one Wazuh manager containers, one Wazuh indexer containers, and one Wazuh dashboard container. It can be deployed by following these steps: 

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
- it took me a long time to find how to put this behind traefik. Everything was configured, but I could only access it via ip, not the dns name. Fixed by commenting the following in this file:
<br>
 */home/hughboi/data/wazuh/config/wazuh_dashboard/opensearch_dashboards.yml*
 ```
    #server.ssl.enabled: true
    #server.ssl.key: "/usr/share/wazuh-dashboard/certs/wazuh-dashboard-key.pem"
    #server.ssl.certificate: "/usr/share/wazuh-dashboard/certs/wazuh-dashboard.pem"
```


### Discord Notifications
Hamburger Menu -> Server Management -> Settings -> Edit Configuration (Top right corner)
1. Add the following in the </global> field
```
 <integration>
     <name>custom-discord</name>
     <hook_url>https://discord.com/api/webhooks/1385XXXXXXXXXXXX/XXXXXXXXXXXXX</hook_url>
     <alert_format>json</alert_format>
 </integration>
```

 2. Save + Restart Manager
 3. On the ubuntu host, go to the integrations mapped folder in the cli and do the following while under **sudo su**
      1. Grab the integration files <br>
        ```
        wget https://raw.githubusercontent.com/maikroservice/wazuh-integrations/main/discord/custom-discord
        wget https://raw.githubusercontent.com/maikroservice/wazuh-integrations/main/discord/custom-discord.py
        ```

      2. Then run the following:
        ```
        sudo chmod 750 ./wazuh_integrations/custom-*
        sudo chown root:wazuh ./wazuh_integrations/custom-*
        ```
      3. Then get the python extension <br>
        ```
        sudo apt-get install python3-pip
        pip3 install requests
        ```
  4.  Restart Wazuh's controls
  ```
  sudo docker compose up -d --force-recreate
  ```
