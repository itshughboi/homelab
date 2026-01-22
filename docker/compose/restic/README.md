## TrueNAS NFS Mount
1. On Ubuntu host, add the mount (make sure /mnt/truenas/restic exists prior)
```sh
sudo mount "truenas:/mnt/The Archive/Restic" /mnt/truenas/restic
```
2. Mount upon reboot ``` nano /etc/fstab ```
```sh
"truenas:/mnt/The Archive/Restic" /mnt/truenas/restic nfs defaults 0 0
```

## TrueNAS Permissions
1. Click on the Dataset -> Edit Permissions <br>
User: hughboi (Apply user CHECKED) <br>
Group: hughboi (Apply Group CHECKED) <br>

***
**Advanced**: <br>
Apply permissions recursively (CHECKED) <br>
Apply permissions to child datasets (CHECKED) <br>

***
**Access**:
- User: RWX
- Group: RX
- Other: RX
