## Network Layout

### Proxmox

| Name      | IP         | Role            |
| --------- | ---------- | --------------- |
| pve-srv-1 | 10.10.10.1 | storage, docker |
| pve-srv-2 | 10.10.10.2 | k3s             |
| pve-srv-3 | 10.10.10.3 | k3s             |
| pve-srv-4 | 10.10.10.4 | k3s             |


###### VLANs

| VLAN     | ID  | Network         | Notes                          |
| -------- | --- | --------------- | ------------------------------ |
| Home     | 1   | 10.10.10.0/24   | Proxmox, Unifi, Bind9, Truenas |
| Swarm    | 2   | 10.10.20.0/24   | Swarm                          |
| k3s      | 3   | 10.10.30.0/24   | Masters + Workers, Longhorn    |
| VPN      | 8   | 10.10.80.0/24   | VPN                            |
| Testing  | 200 | 10.200.200.0/24 |                                |
| Ad-Block | 111 | 10.10.111.0/24  | IoT and guests for Adblock DNS |
| torrent  | 49  | 172.16.20.0/24  | torrent + mullvad              |
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
