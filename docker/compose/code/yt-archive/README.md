- Taken from official documentation: https://github.com/tubearchivist/tubearchivist

## Permissions for elasticsearch

If you see a message similar to Unable to access 'path.repo' (/usr/share/elasticsearch/data/snapshot) or failed to obtain node locks, tried [/usr/share/elasticsearch/data] and maybe these locations are not writable when initially starting elasticsearch, that probably means the container is not allowed to write files to the volume. To fix that issue, shutdown the container and on your host machine run:
```
chown 1000:0 -R /path/to/mount/point
```

This will match the permissions with the UID and GID of elasticsearch process within the container and should fix the issue.

*** 
<br>

## Truenas Connection

1. Mounting locally via NFS
```sh
sudo nano /etc/fstab
```

Add the following: (Reaplce YT-Audios with whatever TrueNAS dataset is called, and what you want local folder to be called under /mnt). \040 is there to ignore the space in the TrueNAS pool name. Using double quotes to make it a string doesn't always work either. Copy exactly as below.
```sh
10.10.10.5:/mnt/The\040Archive/YT-Audios /mnt/truenas/yt-audios nfs defaults 0 0
```

Then run this to mount it:
```sh
sudo mount -a
```

#### TrueNAS Permissions
- Tube Archivist runs as root user. On TrueNAS, on the NFS share, set **mapall user** to root. Then under Dataset -> Permissions, give root full access to the dataset