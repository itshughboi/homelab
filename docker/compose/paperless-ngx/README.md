### Troubleshooting
###### Permission Denied
1. Create user on TrueNAS for paperless. Take note of UID & GID. 
2. Create the NFS share and set the **Mapall User** to be **hughboi** and save
3. Put UID & GID in .env of docker project
4. Stand up and should be good to do
