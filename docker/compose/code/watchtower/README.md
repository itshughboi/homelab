## Overview
Watchtower is used to automatically pull the newest image (if available) every 24 hours. It works on containers that use the 'latest' tag.
I specify critical containers that should not be managed by watchtower but still might be under latest tag.

## Installation
1. Create App Token from Gotify and put it in 'access_token' file in same directory as docker-compose.
2. Docker compose up and enjoy


## Notifications
1. Gotify
   1. Register Gotify App Token in Gotify UI and put it into **gotify_token**
2. Ntfy << Uses shoutrr
   1. Add HTTP_API_TOKEN environment variable
   2. Add a ntfy_token and put in a random string.
3. Discord < Uses shoutrr
```sh
environment:
  - WATCHTOWER_HTTP_API_TOKEN=/run/secrets/access_token
```
   1. Add credentials to .env variable

## Metrics
- New as of Watchtower v1.0.4
**Pre-Requisisites**
1. Setup API endpoint. Add this to environment
```sh
- WATCHTOWER_HTTP_API_METRICS=true
- WATCHTOWER_HTTP_API_TOKEN=/run/secrets/watchtower_api_token
```
2. Add API Token
   1. Create new file called **watchtower_api_token** and put in a random complex string. Put this string into prometheus.yml
3. Add a port mapping for port **8080** e.g. 8765:8080
