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
