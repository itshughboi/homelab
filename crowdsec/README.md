### Troubleshooting
Ensure that traefik logs can be accessed by the container. 
```sudo exec -it crowdsec /bin/sh``` and then ```cd /var/log/traefik/```. You should see **access.log** and **traefik.log**.


***

### API Key
``` sudo docker exec crowdsec cscli bouncers add bouncer-traefik``` and put it into the env
