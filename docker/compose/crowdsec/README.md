## Installation
1. Configure Traefik. <br>
    a. Add crowdsec middleware
    ```
      http:
        middlewares:    
            crowdsec-bouncer:
            forwardauth:
                address: http://bouncer-traefik:8080/api/v1/forwardAuth
                trustForwardHeader: true     
    ```
    b. Add this to traefik.yml
    ```
    log:
      level: "INFO"
      filePath: "/var/log/traefik/traefik.log"
    accessLog:
      filePath: "/var/log/traefik/access.log"
    ```
    c. Restart traefik
2. Start docker compose
3. Go into crowdsec container to create API key
```sudo docker exec crowdsec cscli bouncers add bouncer-traefik```
4. Put API Key into .env
5. Restart container
