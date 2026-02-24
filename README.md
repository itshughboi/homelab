## Network Layout

### Proxmox

| Name      | IP         | Role            |
| --------- | ---------- | --------------- |
| pve-srv-1 | 10.10.10.1 | storage, docker |
| pve-srv-2 | 10.10.10.2 | k3s             |
| pve-srv-3 | 10.10.10.3 | k3s             |
| pve-srv-4 | 10.10.10.4 | k3s             |


###### VLANs

| Name         | VLAN ID | CIDR           | Notes                  |
| ------------ | ------- | -------------- | ---------------------- |
| Management   | 1       | 10.10.10.0/24  | SSH, Web UI, Unifi, Bind9
| Cluster      | 2       | 10.10.20.0/24  | Corosync               | 
| k3s          | 3       | 10.10.30.0/24  |                        |   
| Storage      | 4       | 10.10.40.0/24  | TrueNAS, PBS, Longhorn |
| VPN          | 8       | 10.10.80.0/24  | Tailscale              |
| Torrent      | 49      | 172.16.49.0/24 |                        |
| Provisioning | 99      | 10.10.99.0/24  | Netboot                |

***
**Notes:** Ports that connect to Proxmox nodes, trunked devices, or managed switches (Unifi) should all be set to **trunk** or **allow all VLANs**. Configure VLANs on both Proxmox hosts under the virtual NIC (assign the VLAN to a bridge interface which binds to a specific port) and then that bridge will carry the VLAN for that VM.
***


### Essentials
| Name        | VLAN | IP           | Gateway      |
| ----------- | ---- | ------------ | ------------ |
| truenas     | 1    | 10.10.10.5   | 10.10.10.254 |
| pbs         | 1    | 10.10.10.6   | 10.10.10.254 |
| postgres-ha | 1    | 10.10.10.7   | 10.10.10.254 |
| ansible     | 1    | 10.10.10.8   | 10.10.10.254 |
| bind9       | 1    | 10.10.10.9   | 10.10.10.254 |
| ubnt-prod   | 1    | 10.10.10.10  | 10.10.10.254 |
| vpn-gateway | 8    | 10.10.80.254 | 10.10.80.254 |


### k3s
| Name             | IP          |
| ---------------- | ----------- |
| k3s-master-1     | 10.10.30.1  |
| k3s-master-2     | 10.10.30.2  |
| k3s-master-3     | 10.10.30.3  |
| k3s-worker-1     | 10.10.30.11 |
| k3s-worker-2     | 10.10.30.12 |
| k3s-worker-3     | 10.10.30.13 |
| k3s-api-vip      | 10.10.30.30 |
| k3s-longhorn-vip | 10.10.30.50 |
| k3s-longhorn-1   | 10.10.30.51 |
| k3s-longhorn-2   | 10.10.30.52 |
| k3s-longhorn-3   | 10.10.30.53 |
| MetalLB Services | 10.10.30.60 |
| .                |             |
| traefik-vip      | 10.10.30.65 |
| pihole-vip       | 10.10.30.69 |
| .                |             |
| End MetalLB      | 10.10.30.99 |



***

<br>
<br>
<br>
<br>

## To-Do
1. Setup LACP for a single high bandwidth connection between two switches, and then setup MLAG between the two switches with a high speed link
2. QoS
	1. Prioritize Storage backups + RKE2 control plane
3. DNS
	1. Ensure my split-horizon DNS is setup correctly within Bind9
	2. Make sure DNS-over-TLS is configured
	3. Integrate with Ansible
	4. Make sure my DNS is setup with whatever Load Balancer I use on my cluster
4. Use network monitoring to monitor each vlan. Watch for latency, jitter, packet loss
5. Setup offsite backup from PBS
	1. Backup:
		1. Network configs (routers + switches)
		2. Proxmox cluster config
		3. Swarm + RKE2 service manifests
		4. App volumes
6. Recovery
	1. Setup automation for redeployment in case of failures (Ansible + Terraform
7. Ensure IDS/IPS on Unifi is working properly. Look into Snort for my test lab
	1. This should be on a mirrored port or inline between VLANs
8. VLANs
	1. Ensure proper rules for inter-vlan routing are setup
	2. Use Ansible/Terraform to automate VLAN management + firewall/routing rules + device configuration backups
	3. Setup a Lab/Test environment. Put OPNSense in this and i can play more with VLANs and stuff
9. VPN
	1. Add backup VPN if tailscale isn't available. Wireguard
	2. Segment VPN into its own VLAN for monitoring and control
10. HA
	1. Setup shared storage and fencing for Proxmox for cluster recovery. GlusterFS or Ceph. Redundancy and scales
11. CI/CD
	1. Create pipelines with Gitea
12. Logging
	1. Centralize logs from Unifi, Proxmox, TrueNAS, Docker, etc. into a SIEM like Wazuh or an ELK Stack, but probably Wazuh if possible
13. Power Backup
	1. Setup critical services on UPS in case of outage. Trigger with Kestra if it goes to battery backup, to start the graceful shutdown period shortly after if there is no uptime 
14. Environment monitor
	1. Put sensors near my equipment to monitor
15. Setup Out of Band Management
16. Use Authentik to authenticate across devices, Proxmox, etc. make sure I can get into things if Authentik is down
17. Encryption
	1. Make sure datasents on TrueNAS and Proxmox are encrypted
	2. make sure backups to offsite are also encrypted
18. Automate responses to detected threats with Wazuh
	1. Quarantine compromised VM's or block IPs in real time << with Crowdsec too
19. Backups
	1. Ensure backups stored on PBS and elsewhere are **write-once-read-many** to protect against ransomware
20. DevOps 
	1. Canary Deployment
		1. Test updates in production environment with minimal risk
	2. Chaos Engineering
		1. Use something like **Gremlin** or **Chaos Mesh** to test failures and validate system's resilience
21. Upgrades
	1. Upgrade hardware eventually to 10 GbE
	2. Enable and setup IPv6 for future proofing


### Adguard
1. Get DNSEC support
2. Enable DoH/DoT or even better DoQ if available. 
	1. See how unbound handles this and what all needs to be configured both on adguard and on unbound



### To-Do
***
##### Adguard
1. Setup DoH/DoT
2. Setup Adguard-sync so I can have another replica
3. Think about switching DNS to be a VIP pointing to both this adguard + another one elsewhere


##### Gitea
1. Change database from HA Postgres to a container
3. Connect with Renovate bot if it's compatible


#### Grafana
- Move promtail over to Grafana Alloy


#### Homepage
- Get crowdsec integration at least tested and put into gitea even if i comment it out


##### Kestra
1. Take a look on how I can use this in my workflow


##### Loki
1. Add Authentic middleware for machine to machine authentication for logs
2. Fix my promtail /var/logs push - TLS error
3. Add env variables for Loki api call
4. Adjust Wazuh labels
5. Add redis to searxng (optionally add Mullvad via Gluetun config to route all traffic through)


##### OpenTofu
1. Setp all mt files and providers so that I can start using this to provision machines and states
2. Connect Terraform to Proxmox to provision VM's and LXC's


#### Prometheus
- Setup Alert Manager & Node Exporter on my hosts so that I can monitor everything
- Setup Exporter with my Bind9 and configure Grafana Dashboard
- Setup Exporter with unbound DNS and configure Grafana Dashboard


##### Semaphore
1. Connect Semaphore with my git repository hosting my ansible files.
2. Integrate with OpenTofu


##### Speedtest-tracker
1. Add to InfluxDB and setup Grafana graph for it


##### TrueNAS
1. Setup backups from Windows to TrueNAS via FileHistory


#### Watchtower
1. Bring watchtower into Grafana via Prometheus

##### Wazuh
1. Setup the ansible script it provides and use inventory file to setup all agents for it to be installed on. Integrate this with OpenTofu for full stack automation
2. Get logs from Unifi working properly
3. Change password of admn user
4. Add all my agents/vm's to Wazuh + utilize Docker plugin for containers
5. Make sure notifications are set up with correct thresholds

