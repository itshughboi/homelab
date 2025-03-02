### Troubleshooting
Ensure that traefik logs can be accessed by the container. 
```sudo exec -it crowdsec /bin/sh``` and then ```cd /var/log/traefik/```. You should see **access.log** and **traefik.log**.

Adding an IP to the decisions list:
```sudo docker exec crowdsec cscli descisions add --ip 10.10.10.137```
Query decisions list
```sudo docker exec crowdsec cscli decisions list```

***

### API Key
``` sudo docker exec crowdsec cscli bouncers add bouncer-traefik``` and put it into the env


***

### Traefik ForwardAuth
1. We need to add a middleware to our traefik config. ```nano /home/hughboi/traefik/data/config.yml```

```
http:
  middlewares:    
    crowdsec-bouncer:
      forwardauth:
        address: http://bouncer-traefik:8080/api/v1/forwardAuth
        trustForwardHeader: true
```

2. Then we need to add this middleware as an entrypoint. ```nano /home/hughboi/traefik/data/traefik.yml```

***
```
entryPoints:
  http:
    address: ":80"
    http:
      middlewares:
        - crowdsec-bouncer@file
      redirections:
        entryPoint:
          to: https
          scheme: https
  https:
    address: ":443"
    http:
      middlewares:
        - crowdsec-bouncer@file
```
