## Initial Setup
https://vaultwarden.hughboi.cc/setup

## 2FA
1. In the web app, go to Settings -> Security.
2. Select Two-step login.
3. Under Passkey, hit manage.
    - Put in name (e.g. Yubikey - 1))
    - Make another one! Ideally the other Yubikey at minimum


## Device Sign In
- After signing in to the app or in a web browser, when you have a second device to sign in, put in email, then instead of master password hit "login with device" and then in your logged in session go to Settings -> Account Security -> pending login request, and approve the request there
- Note: You will still need the 2FA token or passkey if 2FA is enabled. Device sign in just bypasses 1st Sign On Factor


## Passkey Sign On (Currently being implemented)
You can add up to five passkeys to your Bitwarden account. To create a passkey for logging in to Bitwarden:

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



## Admin Panel
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






## Recovery Keys
- Obtain by signing in -> Security -> Two-step login -> View Recovery Code
Keep this in 3+ places:
1. Keep physical copy in safe
2. Encrypted file on TrueNAS
3. iCloud Keychain
4. Paperless-ngx
5. (Optional): Create a emergency contact with a second email address (Set to View or Takeover Vault) << Create user first
    - Settings -> Emergency Access -> Add emergency contact (Access Level View) 





## Backups
Official documentation: https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault
1. Proxmox Backup Server VM Image (1x daily)
2. Restic -> TrueNAS (2x daily )
3. tar backup via n8n (1x daily)

### **Manual Backup (Simple)**
```sh
tar -czf /home/hughboi/data/vaultwarden/backups/vaultwarden-$(date +%F).tar.gz --exclude='./backups' -C /home/hughboi/data/vaultwarden .
```


### **n8n Backups (Advanced)**
1. Schedule Trigger (8:00 am daily)
2. Execute command (put a compose down command before backup command)
    - Credentials to connect with (SSH Password account)
    - Resource: Command
    - Operation: Execute
    - Command: (backups to directory, purges anything over 45 days, calculates backup size)
    ```sh
    export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" && BACKUP_NAME="vaultwarden-$(date +%F).tar.gz" && BACKUP_DIR="/home/hughboi/data/vaultwarden/backups" && tar -czf "$BACKUP_DIR/$BACKUP_NAME" --exclude="$BACKUP_DIR" -C /home/hughboi/data/vaultwarden . && PURGED=$(find "$BACKUP_DIR" -name "*.tar.gz" -type f -mtime +45 -print -delete | xargs -n1 basename | tr '\n' ' ') && SIZE=$(du -BG "$BACKUP_DIR/$BACKUP_NAME" | cut -f1) && echo "Size: $SIZE | Purged: ${PURGED:-None}"
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

- n8n should then run 'docker compose up -d' whether succes or fialure

## Restore
1. Stop vaultwarden
2. Move current data to new folder
```sh
mv /home/hughboi/data/vaultwarden/data /home/hughboi/data/vaultwarden/data.broken.$(date +%F)
mkdir -p /home/hughboi/data/vaultwarden/data
```
3. Restore .tar.gz backup (Replace with file name)
```sh
tar -xzf /home/hughboi/data/vaultwarden/backups/vaultwarden-2026-04-13.tar.gz \
-C /home/hughboi/data/vaultwarden
```
4. Set permissions if needed
```sh
chown -R 1000:1000 /home/hughboi/data/vaultwarden/data
```
5. Start vaultwarden again

