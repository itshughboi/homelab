### Importing from TrueNAS
1. Create mount point on host 
2. '''sudo mount -t nfs truenas:/mnt/The\040Archive/Liyah /mnt/truenas/eros'''
3. Once container is spun up and we can verify it was mounted into the container, on the GUI go to External Libraries -> Add
Import Path: /mnt/media/truenas OR /usr/src/app/upload