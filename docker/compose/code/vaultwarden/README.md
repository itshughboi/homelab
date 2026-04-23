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




# Backups + Restore
Official documentation: https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault \
My setup:
1. Proxmox Backup Server VM Image (1x daily)
2. Restic -> TrueNAS (2x daily )
3. Encrypted tar backup of /data to local folder via n8n (1x daily)


## Manual Backup
*Unencrypted*
```sh
tar -czf /home/hughboi/data/vaultwarden/backups/vaultwarden-$(date +%F).tar.gz --exclude='./backups' -C /home/hughboi/data/vaultwarden .
```
*Encrypted + updated*
```sh
DATE=$(TZ="America/Denver" date +%F) && TAR_FILE="/home/hughboi/data/vaultwarden/backups/vaultwarden-$DATE.tar.gz" && ENC_FILE="$TAR_FILE.age" && tar -czf "$TAR_FILE" --exclude='./backups' -C /home/hughboi/data/vaultwarden . && age -r YOUR_PUBLIC_KEY_HERE -o "$ENC_FILE" "$TAR_FILE" && rm "$TAR_FILE"
```

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
3. Save private key to Vaultwarden
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

Note! Always test restores. To restore, unzip the archive back into /data, replacing the previous data directory

<br>



## N8N Backups
1. Schedule Trigger (8:00 am daily)
2. Execute command
    - Credentials to connect with (SSH Password account)
    - Resource: Command
    - Operation: Execute
    - Command: (backups sqlite database + data folder, zips it, encrypts it, puts into local backups, purges anything over 45 days, calculates backup size)
    ```sh
    sqlite3 /home/hughboi/data/vaultwarden/data/db.sqlite3 ".backup /home/hughboi/data/vaultwarden/data/db-backup.sqlite3" && sync && BACKUP_DIR="/home/hughboi/data/vaultwarden/backups" && mkdir -p "$BACKUP_DIR" && DATE=$(TZ="America/Denver" date +%F) && TAR_FILE="$BACKUP_DIR/vaultwarden-$DATE.tar.gz" && ENC_FILE="$TAR_FILE.age" && tar --warning=no-file-changed -czf "$TAR_FILE" -C /home/hughboi/data/vaultwarden data && age -r age15zjkhtmytnx5l6fhhpte6gg5ha9ede28tgwcd2ewxtwth9l5090sve0rv6 -o "$ENC_FILE" "$TAR_FILE" && sha256sum "$ENC_FILE" > "$ENC_FILE.sha256" && rm "$TAR_FILE" && rm /home/hughboi/data/vaultwarden/data/db-backup.sqlite3 && LATEST=$(ls -t "$BACKUP_DIR"/vaultwarden-*.tar.gz.age | head -1) && SIZE=$(du -h "$LATEST" | awk '{print $1}') && PURGED=$(find "$BACKUP_DIR" \( -name "*.tar.gz.age" -o -name "*.tar.gz.age.sha256" \) -type f -mtime +45 -print -delete | tr '\n' ' ') && printf "size=%s purged=%s\n" "$SIZE" "${PURGED:-None}"
    ```
    - Working Directory: /home/hughboi/
    - Settings: ALYWAYS OUTPUT DATA / On Error (Continue)
3. IF
    - Condition: {{ $json.code }} is equal to **0** << True Route
4. Success Path:
    - HTTP Post to https://ntfy.hughboi.cc/n8n
    - Body Content Type: Raw
    - Send Body (Checked)

        *Body*
        ```sh
        Vaultwarden backup completed!
        Date/Time: {{ $now.format('yyyy-MM-dd HH:mm:ss') }}
        Content: {{ $json.stdout }}
        ```
        
5. Failure Path:
    - *Body*
        ```sh
        Vaultwarden Backup FAILURE - {{ $now.format('yyyy-MM-dd HH:mm:ss') }}
        Error: {{ $json.stderr }}
        ```

<br>


## Manual Restore
1. Stop vaultwarden
2. Move current data to new folder
```sh
mv /home/hughboi/data/vaultwarden/data /home/hughboi/data/vaultwarden/data.broken.$(date +%F)
mkdir -p /home/hughboi/data/vaultwarden/data
```
3. Decrypt backup
Note! change ~/.config/age/keys.txt to location of key file and cd into backups directory
```sh
age -d -i ~/.config/age/keys.txt vaultwarden-YYYY-MM-DD.tar.gz.age > vaultwarden.tar.gz
```
3. Restore .tar.gz backup (Replace with file name)
```sh
tar -xzf /home/hughboi/data/vaultwarden/backups/vaultwarden-YYYY-MM-DD.tar.gz -C /home/hughboi/data/vaultwarden
```
4. Set permissions if needed
```sh
chown -R 1000:1000 /home/hughboi/data/vaultwarden/data
```
5. Start vaultwarden again

<br>

## Testing Restores with n8n
- Node 1 Schedule
    - 1x month
- Node 2 SSH
    - Find latest backup
    ```sh
    ls -t /home/hughboi/data/vaultwarden/backups/*.tar.gz.age | head -1
    ```
- Node 3 Code (Javascript)
    - Data modification (Raw -> Structured)
    ```sh
    return [{
    json: {
        ...$json,
        backup: $json.stdout.trim()
    }
    }];
    ```
- Node 4 SSH
    - Decrypt backup (Expression)
    ```sh
    BACKUP="{{ $json.backup }}"
    BACKUP_DIR=$(dirname "$BACKUP")
    BACKUP_FILE=$(basename "$BACKUP")
    TMP=$(mktemp -d)

    # 1. Change to the directory so sha256sum finds the file correctly
    cd "$BACKUP_DIR"

    # 2. Verify Checksum
    if ! sha256sum -c "$BACKUP_FILE.sha256" > /dev/null 2>&1; then
        printf '{"status":"fail","error":"checksum_mismatch","backup":"%s"}' "$BACKUP"
        exit 0
    fi

    # 3. Decrypt
    if age -d -i ~/.config/age/keys.txt "$BACKUP_FILE" > "$TMP/vw.tar.gz" 2>/dev/null; then
        printf '{"status":"ok","backup":"%s","tmp":"%s"}' "$BACKUP" "$TMP"
    else
        printf '{"status":"fail","error":"decryption_failed","backup":"%s"}' "$BACKUP"
    fi
    ```
- Node 5 Code (Javascript)
    - JSON parse -> object
    ```sh
    const data = JSON.parse($json.stdout);

    return [{
    json: {
        backup: data.backup,
        tmp: data.tmp.replace("tmp:", "")
    }
    }];
    ```
- Node 6 SSH
    - Extract backup (Expressions)
    ```sh
    TMP="{{ $json.tmp }}" && tar -xzf "$TMP/vw.tar.gz" -C "$TMP" && printf "$TMP"   
    ```
- Node 7 Code (Javascript)
    ```sh
    for (const item of $input.all()) {
    item.json.myNewField = 1;
    }

    return $input.all();
    ```
- Node 8 SSH
    - Start vaultwarden test container (Expression)
    ```sh
    # 1. Setup path
    TMP=$(echo "{{ $json.stdout }}" | head -n 1 | tr -d '"')

    # 2. Start container and WAIT for it to be ready
    docker run -d --rm --name vw-test -p 8093:80 -v "$TMP/data:/data" vaultwarden/server:latest > /dev/null
    until curl -sf http://localhost:8093/alive > /dev/null; do sleep 1; done

    # 3. RUN THE CHECKS while it is still running
    STATUS="ok"
    ERROR=""
    if ! curl -sf http://localhost:8093/api/config > /dev/null; then 
        ERROR="api_failed"
    elif ! (sqlite3 "$TMP/data/db.sqlite3" "PRAGMA integrity_check;" || sqlite3 "$TMP/db.sqlite3" "PRAGMA integrity_check;") | grep -q "ok"; then 
        ERROR="sqlite_failed"
    fi

    # 4. NOW TEARDOWN (Only after checks are done)
    [ -n "$ERROR" ] && STATUS="fail"
    docker stop vw-test > /dev/null || true

    # 5. Output the result to n8n
    printf '{"status":"%s","error":"%s","backup":"%s"}\n' "$STATUS" "$ERROR" "{{ $('Extract Path').item.json.backup }}"
    ```
- Node 9 IF
    ```sh
    {{ JSON.parse($('Start Test Container').item.json.stdout).status }} == ok
    ``` 
    - Success: 
        - POST
            - https://ntfy.hughboi.cc/n8n
        - Send Body (Raw)
            - Vaultwarden Restore Test PASSED | Backup={{ $json.backup }} | Time={{ $now }}
    - Failure
        - POST
            - https://ntfy.hughboi.cc/n8n
        - Send Body (Raw)
            - Vaultwarden Restore Test FAILED | Error={{ $json.error }} | Backup={{ $json.backup }} | Time={{ $now }}



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

# Wipe the corrupted directory
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
