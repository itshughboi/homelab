## Initial Setup
https://vaultwarden.hughboi.cc/setup 

<br>



# Security

### 2FA
1. In the web app, go to Settings -> Security.
2. Select Two-step login.
3. Under Passkey, hit manage.
    - Put in name (e.g. Yubikey - 1))
    - Make another one! Ideally the other Yubikey at minimum


### Device Sign In
- After signing in to the app or in a web browser, when you have a second device to sign in, put in email, then instead of master password hit "login with device" and then in your logged in session go to Settings -> Account Security -> pending login request, and approve the request there
- Note: You will still need the 2FA token or passkey if 2FA is enabled. Device sign in just bypasses 1st Sign On Factor


### Passkey Sign On (Bitwarden only!!)
        You can add up to five passkeys to your Bitwarden account (Vaultwarden later). To create a passkey for logging in to Bitwarden:

        1. In the web app, go to Settings → Security.
        2. Select Master password.
        3. Within the Log in with passkey section, select Turn on or, if you've already setup a passkey, New passkey. You'll be prompted to enter your master password.
        4. Follow prompts from your browser to create a FIDO2 passkey. You can complete user verification using a factor like a biometric or by creating a PIN.
        5. Enter a Name for your passkey.
        6. (Optional) If your browser and authenticator are PRF-capable, the Use for vault encryption setting will be checked by default. This allows your passkey to decrypt and unlock your vault. Uncheck this option if you do not want the passkey to unlock your vault:


        To log in with a passkey and unlock your vault:

        1. On the Bitwarden login screen, select Log in with passkey where you'd usually enter your email address.
        2. Follow prompts from your browser to read the passkey. This will authenticate you with Bitwarden.
        3. What happens next depends on if your passkey is set up for vault encryption:
            - If your passkey is set up for vault encryption, you're done! The passkey is used to decrypt and unlock your vault.
            - If your passkey is not set up for vault encryption, enter your master password and select Unlock, or use another unlock method you previously configured.

        To unlock your vault when you're already logged in, select Unlock with Passkey on the locked vault screen. Follow prompts from your browser to read the passkey. This will unlock your account, opening your vault.


<br>

### Recovery Keys
- Obtain by signing in -> Security -> Two-step login -> View Recovery Code
Keep this in 3+ places:
1. Keep physical copy in safe
2. Encrypted file on TrueNAS
3. iCloud Keychain
4. Paperless-ngx
5. (Optional): Create a emergency contact with a second email address (Set to View or Takeover Vault) << Create user first
    - Settings -> Emergency Access -> Add emergency contact (Access Level View) 



\
\
<br>

# Admin Panel
**Note**: Everything in here can pretty much be set with env variables, so this isn't necessary unless I actually want to do it from the GUI
#### Enabling Panel
- Set an environment variable with "ADMIN_TOKEN=". Execute into the vaultwarden container and run this:
```sh
/vaultwarden hash
```
- Set a password and then copy out the hashed string it gives. Remove the single quotes around the string. 
    - IF THERE ARE ANY DOLLAR SIGNS YOU MUST ESCAPE IT WITH ANOTHER DOLLAR SIGN. 
    - Check logs to make sure hash is being taken properly
- Navigate to https://vaultwarden.hughboi.cc/admin

#### OpenID SSO
- OpenID Connect SSO Settings
    - Enabled (Check)
    - Setup ID, Key, Callback with PocketID
    - Authoritzation request scopes:
        - openid email profile
- **NOTE: If using Pocket ID, you cannot link it to an existing account. It thinks you're doing a account hijacking attack basically. I went down this route for many hours, but decided I am just going to wait til Vaultwarden officially supports passkeys natively.**

#### SMTP
- SMTP Email Settings (Only needed to change email or for emergency user to request to admins email to take over the account)
    - Enabled (True)
    - Host: smtp.fastmail.com
    - Secure SMTP: starttls
    - Port: 587
    - From: Fastmail email
    - Password: App Password created in Fastmail


\
\
<br>




# Backups
Official documentation: https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault \
My setup:
1. Proxmox Backup Server VM Image (1x daily)
2. Restic -> TrueNAS (2x daily )
3. Encrypted tar backup of /data to local folder via ansible (1x daily)


## Manual Backup
NOTE! SHUT DOWN VAULTWARDEN FIRST \
These manual commands don't utilize the proper  `sqlite3 backup` that my ansible playbooks utilize. As is, these are less safe than running the ansible playbook. If I do in fact want to run it manually, shut down container first!
*Unencrypted*
```sh
tar -czf /home/hughboi/data/vaultwarden/backups/vaultwarden-$(date +%F).tar.gz -C /home/hughboi/data/vaultwarden data
```
*Encrypted + updated*
```sh
DATE=$(TZ="America/Denver" date +%F) && TAR_FILE="/home/hughboi/data/vaultwarden/backups/vaultwarden-$DATE.tar.gz" && ENC_FILE="$TAR_FILE.age" && tar -czf "$TAR_FILE" -C /home/hughboi/data/vaultwarden data && age -R ~/.config/age/keys.txt -o "$ENC_FILE" "$TAR_FILE" && rm "$TAR_FILE"
```

### No downtime manual backup
```sh
docker exec vaultwarden sqlite3 /data/db.sqlite3 ".backup '/data/db_backup.sqlite3'"
```

Then tar up the db_backup.sqlite3 instead of the live db.sqlite3

<br>

##### Encryption Tutorial
Note: Create ~/.config/age if not already created
1. install 'age'
```sh
sudo apt install age
```
2. Generate key
```sh
age-keygen -o ~/.config/age/keys.txt
```
3. Save private key to Vaultwarden, iCloud, TrueNAS, and have printed physical copy.
4. Ensure permission lock
```sh
chmod 600 ~/.config/age/keys.txt
```
5. Run the 'encrypted and updated' command above and replace YOUR_PUBLIC_KEY_HERE with 'age -r age1xxxxxxx'


<br>

## Docker-Based Automated Backups (Official Example)
This stops the container to ensure consistency, zips the data directory, transfers it via scp, and then restarts Vaultwarden. However, some of the other methods don't require downtime and it's not suitable for zero-downtime or automated environments so I wouldn't personally use it

*example*
```bash
#!/bin/bash
docker-compose down
datestamp=$(date +%m-%d-%Y)
backup_dir="/home/hughboi/data/vaultwarden/backups"
zip -9 -r "${backup_dir}/${datestamp}.zip" /home/hughboi/data/vaultwarden/data
scp -i ~/.ssh/id_rsa "${backup_dir}/${datestamp}.zip" hughboi@<10.10.10.5>:~/homelab/vaultwarden/
docker-compose up -d
```

- You can automate this via cron:
```
0 0 * * * /root/transfer_vaultwarden_logs.sh
```
Cleanup Script (optional) to keep only the most recent backup:

```bash
#!/bin/bash
cd ~/backups || exit
find . -type f -name '*.zip' ! -mtime -1 -exec rm {} +
```



<br>



## Ansible Backups
Scheduled every day to run from an ansible playbook via semaphore


<br>

## Weekly Backup Testing
Ansible playbook + Semaphore runs weekly. It finds the latest encrypted backup, verifies its checksum, decrypts it, stands up a test Vaultwarden container, and curls endpoints to confirm the backup is restorable.

<br>

# Restores
Note! Always test restores. To restore, unzip the archive back into /data, replacing the previous data directory


🚨 CRITICAL RESTORE STEPS
    1. Stop Container: docker stop vaultwarden  \
    2. Verify Integrity: Run PRAGMA integrity_check; on any restored DB.    \
    3. Fix Permissions: chown -R 1000:1000 is mandatory for the container to write to the DB.   


## Manual Restore
1. Stop vaultwarden
2. Move current data to new folder

```sh
mv /home/hughboi/data/vaultwarden/data /home/hughboi/data/vaultwarden/data.broken.$(date +%F)
mkdir -p /home/hughboi/data/vaultwarden/data
```
3.Decrypt backup — note: change ~/.config/age/keys.txt to location of key file and cd into backups directory
```sh
age -d -i ~/.config/age/keys.txt vaultwarden-YYYY-MM-DD.tar.gz.age > vaultwarden.tar.gz
```
4. Extract archive
```sh
tar -xzf vaultwarden-YYYY-MM-DD.tar.gz -C /home/hughboi/data/vaultwarden
```
5. Promote the hot backup as the canonical database
- The archive contains both db.sqlite3 (live at time of backup, may have been mid-write) and db-backup.sqlite3 (clean consistent snapshot taken by the playbook). Always use the hot backup as your source of truth:

```sh
mv /home/hughboi/data/vaultwarden/data/db-backup.sqlite3 \
   /home/hughboi/data/vaultwarden/data/db.sqlite3
```
7. Set permissions if needed
```sh
chown -R 1000:1000 /home/hughboi/data/vaultwarden/data
```
8. Start vaultwarden






<br>

## Restic Restore
1. List existing snapshots
```sh
docker exec -it restic restic snapshots
```
2. Targeted restore
```sh
# Replace SNAPSHOT_ID with your actual ID (e.g., 36aa3445)
docker exec -it restic restic restore SNAPSHOT_ID \
  --target /tmp-for-restore \
  --include /mnt/volumes/data/vaultwarden/data/db.sqlite3
```
Note: Restic ignores lchown errors on NFS mounts; as long as it says "Summary: Restored 1 files", it worked
3. Integrity check
```sh
sqlite3 /mnt/truenas/restic/tmp-for-restore/mnt/volumes/data/vaultwarden/data/db.sqlite3 "PRAGMA integrity_check;"
```
CRITICAL: If the output is anything other than ok, STOP. Delete the temp file and go back to Step 1 with an older Snapshot ID.
4. Once I see "ok" wipe corrupted directory and replace it
```sh
# Stop the service
docker stop vaultwarden

# Wipe the corrupted directory. MAKE SURE THAT YOU REMOVE .WAL AND SHM FILES!!!
rm -rf /home/hughboi/data/vaultwarden/data/*

# Move the verified DB into place
cp /mnt/truenas/restic/tmp-for-restore/mnt/volumes/data/vaultwarden/data/db.sqlite3 /home/hughboi/data/vaultwarden/data/

# Restore the config if needed
[ -f /mnt/truenas/restic/tmp-for-restore/mnt/volumes/data/vaultwarden/data/config.json ] && cp /mnt/truenas/restic/tmp-for-restore/mnt/volumes/data/vaultwarden/data/config.json /home/hughboi/data/vaultwarden/data/
```
5. Final touches
```sh
# Set ownership back to hughboi
chown -R 1000:1000 /home/hughboi/data/vaultwarden/data

# Bring it back up
docker start vaultwarden
```

<br>

### Other
- I run a `VACCUM` on sqlite.db once a year for good health maintenance of the database. This is orchestrated through ansible + semaphore

