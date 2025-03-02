### Troubleshooting
Ensure that traefik logs can be accessed by the container. 
```sudo exec -it crowdsec /bin/sh``` and then ```cd /var/log/traefik/```. You should see **access.log** and **traefik.log**.


***

### API Key
``` sudo docker exec crowdsec cscli bouncers add bouncer-traefik``` and put it into the env


***

### Traefik ForwardAuth
We need to add a middleware to our traefik config. ```nano /home/hughboi/traefik/data/config.yml```

```
http:
  middlewares:    
    crowdsec-bouncer:
      forwardauth:
        address: http://bouncer-traefik:8080/api/v1/forwardAuth
        trustForwardHeader: true
```
