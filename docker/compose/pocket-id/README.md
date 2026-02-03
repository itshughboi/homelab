### Initial Setup
1. Go to https://pocket.hughboi.cc/setup for initial setup

### App Connection
1. Go to **Settings** -> **Administration** -> **OIDC Clients** -> **Add OIDC Client**
2. Give it a name and put the **Client Launch URL** to be the URL of the App. Hit save
3. After saving you will be given the Client ID, Client Secret.
4. Click 'Show more details' below the Client Secret to get all of the URL's for issuer, authorization, token, logout, etc. Everything you need for OIDC will be in the more details

### Passkeys
- Upon initial setup you will be asked to setup a Passkey. You can setup either a Passkey within Bitwarden, Touch ID on Macbook, or a Yubikey.
- To add additional passkeys go to **Settings** -> **My Account** -> **Passkeys** -> A**dd Passkey**

### Proxmox OIDC
1. Create Proxmox Application in Pocket from instructions above. Make sure the **Client Launch URL** specifies the port *8006*
    e.g. - https://10.10.10.1:8006 or https://pve-srv-1.hughboi.cc:8006
2. If I don't already have a User Group created in Pocket, do that first. Then on the OIDC Client under **Allowed User Groups**, speicfy that group. << Currently not working. To get this to work I have to remove the Allowed User Groups else I get a "you're not authorized to sign in" message from Pocket
3. On Proxmox go to **Datacenter** -> **Permissions** -> **Realms** -> Add -> **OpenID Connect Server**.
    Issuer URL: https://pocket.hughboi.cc
    Realm: Name it something like **pocket**
    Client ID: Generated from Pocket
    Client Key: Generated from Pocket
    Autocreate Users: Checked << This is important
    Username Claim: Username << otherwise it generates a user with a UID
4. Sign out of Proxmox and sign into the realm once. This will create the user.
5. Sign out and back in with admin account. You will see under Proxmox Users the newly created user for that Realm. Click on the **Permissions** parent tab and grant permissions to this user
7. Now when you login you will be able to select the *pocket* realm and sign in with passkey
