## Installation
1. docker compose up
2. Create user // probably need to do it from the container
```
semaphore user add --admin --login hughboi --name hughboi --email info@hughboi.cc --passwo
rd CHANGEME
```

3. Add templates
    - Update
        - apt upgrade & restart
    - Maintenance
        - Check diskspace and send notification if running low
        - Provision VM's
    - Installation
        - Install docker
        - apt update
    - Configuration
        - ssh-copy
    - Deployment
4. Do the following
- Define an inventory (list of IP -> Hosts)
- Secure Authentication
- Create Playbooks
- Git

5. Add SSH Key in keystore
6. Create Inventory and use DNS or IP
7. Create repository
- Point this to my Git Repository
- Access Key is only used if I ned to do ssh rather than HTTPS or if it's private repo
8. Add environment
- For now just put in {}


