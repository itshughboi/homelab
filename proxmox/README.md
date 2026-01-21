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


***

## Cloud Init
Go to Ubuntu Cloud Images Releases. Find the LTS I want. Navigate into the 'current' directory. Find the amd.img and right click and hit 'copy link address' and then on local storage in Proxmox query that URL.
https://cloud-images.ubuntu.com/

#### Shell
```
qm create 5003 --memory 8192 --core 4 --name ubnt-cloud-noble --net0 virtio,bridge=vmbr0
```

##### Verify iso exists
```
cd /var/lib/vz/template/iso/
ls
```

##### Add iso to that machine
```
qm importdisk 5003 noble-server-cloudimg-amd64.img local-lvm
qm set 5003 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-5003-disk-0
```

##### Expand Hard drive
```
qm disk resize 5003 scsi0 50G
```

##### Attaching dvd drive essentially
```
qm set 5003 --ide2 local-lvm:cloudinit
```

```
qm set 5003 --boot c --bootdisk scsi0
qm set 5003 --serial0 socket --vga serial0
```

##### Proxmox GUI

##### Hardware
1. Click on the Hard Disk. Turn on **SSD Emulation** so that it can do S.M.A.R.T. 
2. On Memory turn off **ballooning memory**

##### Cloud-init
- Change username to **hughboi**
- Add public key from my automation machine AND from proxmox. I want to add it from the proxmox host so that I can **VNC** into it from console. I can create new ssh keys if I want, but I just use the default ssh-rsa
***
!! In production I would want to specify separate specific keys for ^^
***
- Make sure IP Config is set to DHCP. Else it will clone each VM with same IP. Instead, take the hardware device after I do a Full Clone and then set a **DHCP Reservation**


##### VM Clone
- Right click on template and hit **Full Clone**

