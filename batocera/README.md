## Create and Configure a Proxmox VM

This assumes you already have the Latest Proxmox installed and have some basic familiarity with setting up VM's and Configuring some of the settings. 
### General

1. **VM ID:** Any unused number (default is fine).
2. **Name:** Choose a name (e.g., `Batocera`, `Gaming-VM`).
3. **Start at Boot:** Optional (enable if you want auto-start).
4. Click **Next**.

### OS
1. **Guest OS Type:** Linux
2. **Version:** Latest available
3. **Do not use any media**
4. Click **Next**.
   
### C. System
1. **Graphics Card:** VirtIO-GPU (Virtio) (For using sunshine while still being able to see it in the Proxmox Console and configure the device. )
>[!alert]
If this isn't an option, select none, but afterwards, enable that option before first boot or you will run into issues and problems. 

2. **Machine Type:** `q35`
3. **SCSI Controller:** `VirtIO SCSI single`
4. **BIOS:** Default
5. Leave both checkboxes **unchecked**
6. Click **Next**.
   
### D. Disks
1. **Delete** the default virtual disk
2. Click **Next**.

### E. CPU
1. **Cores:**
	- Minimum 4 cores for SNES, Mega Drive, PlayStation 1.
	- Maximum 8+ cores for PS2, PS3, Wii, etc.
2. **Type:** Select based on your CPU specs for best performance.
3. Click **Next**.
   
### F. Memory
1. **RAM Allocation:**
    - **4GB** minimum for older consoles (SNES, Mega Drive, PS1, etc.).
    - **16GB+** for high-demand emulation (PS2, PS3, Wii, etc.).
2. **Ballooning:** Untick
3. Click **Next**

   
### G. Network
1. Leave as default unless you need a dedicated network adapter.
2. **Disconnected:** Leave unticked (network updates & ROM transfers need this).
3. Click **Next**.


### H. Confirm
1. Review settings and click **Finish**.
2. **Do not tick** "Start VM after creation" (further setup needed).

## GPU Passthrough
After Setting up the VM you will need to enable GPU Passthrough. 

### A. VM > Hardware

### B. Add top of the right section and select PCI Device

### C. Scroll down until you find your Device ID (Usually formatted like ####:##:##.#) and find your Manufacturer and model. 

## Configure The VM to use the Batocera Image
The easiest way is to do it in the command line, you can either SSH into your Proxmox Host, or you can use the GUI command line. 

GUI command line with you will need to manually type out as Copy/Pasting doesn't work great.


>[!alert]
>IF YOU ARE USING GUI, Skip to the ALTERNATIVE wget Option.

### Download Batocera to ISO folder 

1. In a new Tab, Navigate here: https://batocera.org/download
2. Find the best image for your architecture, and right click "download" and then "Copy Link Address"
```shell
cd /var/lib/vz/template/iso
```
3. Type wget and paste the link)
>[!alert]
>Below is an Example. This may not be the best image for you

```shell
wget [https://5d8907c9884e7bcab7c209e10cbce3b3-13730.sites.k-hosting.co.uk/batocera-x86_64-41-20250106.img.gz](https://5d8907c9884e7bcab7c209e10cbce3b3-13730.sites.k-hosting.co.uk/batocera-x86_64-41-20250106.img.gz)
```


### Alternative wget option:
1. Click on the local storage in the Proxmox Host
2. select ISO Images
3. Select Download from URL
4. In a new Tab, Navigate here: https://batocera.org/download
5. Find the best image for your architecture, and right click "download" and then "Copy Link Address"
6. Navigate back to Proxmox and paste that into the Download from URL Box. 
7. Wait until that is finished, and then navigate to the location in the Proxmox Host Terminal.
```shell
cd /var/lib/vz/template/iso
```

### Uncompress Batocera and adjust to 32 GB to support Network Play
Make sure you are in the ISO directory and run these commands: 
```shell
gzip -d batocera-*.gz
```
```shell
truncate -s 32G batocera-*.img
```

### Assign .img file to Batocera VM and make it First Boot (Adjust VM ID and storage location (local-lvm))
Figure out what your VM ID is (Listed top the left of your machine). In this example we are using 2120. 
```shell
qm importdisk 2120 batocera-*.img local-lvm
```
```shell
qm set 2120 --sata0 local-lvm:vm-2120-disk-0
```
```shell
qm set 2120 --boot order=sata0
```


### Delete downloaded batocera image to save 32 GB space*
THIS IS OPTIONAL. I KEPT IT TEMPORARILY FOR TROUBLESHOOTING PURPOSES SO I COULD REBUILD IT IF NECESSARY. AFTER YOU ARE FINISHED IT IS NICE TO REMOVE IT TO CLEAN UP SPACE.

```shell
rm /var/lib/vz/template/iso/batocera-*.img
```
## Set up Tailscale and Sunshine
### Tailscale 
1. **Run the Script**: To install and configure both Tailscale and Sunshine, use the following command in your Batocera terminal:
```shell
curl -L https://bit.ly/BatoceraRDP | bash
```

This should output a tailscale login URL. You will need to type it into your browser manually if you are using the GUI, but should be able to copy/paste it if you are using SSH.
2. Login into Tailscale, and make sure you enable subnet routes in your dashboard 

### Sunshine
That script, after you are finished with tailscale, should automatically install Sunshine. 
1. Navigate to: https://[tailscale-ip-of-Batcoera]:47990 
2. Configure your username and password. 
3. Leave the page open temporarily.

## Set up Moonlight for Streaming your Batocera Machine
1. Download Moonlight on the device you would like to stream too. 
2. Once Downloaded, open it and select settings at the top right and change any of the settings you would like
3. Select "Add Server"
	If you are external, you will need to use your tailscale IP and if you are internal, you will need to use your internal IP
4. It should now appear in the list. If you click it it will give you a pin. Go pack to your webpage we left open, and then select "Pin" at the top of the page, and input the pin and name of your device. 
	For usability, if i am connecting with external i name it as: Device-External
	Alternatively internal, it is just: Device
5. it should now be connected, and accessible! Enjoy!


## Notes:
This guide does not explain how to load ROMS onto the Batocera system. In my personal setup i have a NAS that is storing all of my games, while my Machine is storing the local saves and BIOS. I will eventually move the BIOS to a network storage so i won't need to update it for multiple Batocera machines inside my network, but where everyone keeps all of there own saves on there own machine. 

If there is additional help with setup, i can make another Guide on ROM storage and my problems and solutions in setting up something like this. 

https://github.com/DTJW92/batocera-remote-desktop

