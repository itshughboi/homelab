## NIC's
#### **pve-srv-1** <br>
Intel 82575EB Gigabit Network Connection - 1C:86:0B:20:06:B8 <br>
Intel 82575EB Gigabit Network Connection - 1C:86:0B:20:06:B9 <br>

Intel Ethernet I350-T4 - 98:B7:85:1E:A0:22 - rightmost <br>
Intel Ethernet I350-T4 - 98:B7:85:1E:A0:23 - second to the right <br>
Intel Ethernet I350-T4 - 98:B7:85:1E:A0:24 - second to the left <br>
Intel Ethernet I350-T4 - 98:B7:85:1E:A0:25 - leftmost of the four <br>




## Cluster Setup
1. Manually boot first proxmox node from Ventoy. Enable SSH, and get to where the Web UI is accessible at https://IP:8006
2. Bootstrap a template Ubuntu machine via **cloud-init**.
   - Add SSH key
   - Set Network
4. Full clone the cloud-init template and install Ansible
5. Manually boot the rest of the proxmox nodes. Enable SSH, and get to where the Web UI is accessible at https://IP:8006
6. SSH into ansible machine + git pull ansible playbooks
7. Run the ansible proxmox-node-setup script

There should now be a working proxmox cluster with best security practices.
