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
1. Create Proxmox Application in Pocket from instructions above
2. On Proxmox go to **Datacenter** -> **Permissions** -> **Realms** -> Add -> **OpenID Connect Server**. Put in the details generated from Pocket-ID. Name it something like *pocket* or *oidc*. Don't need to do anything else.
3. Still in Proxmox go to **Datacenter** -> **Permissions** -> **Users** -> **Add**. Assign it to the Pocket Realm we created in step 2 (no password)
4. Assign permissions by going to **Datacenter** -> **Permissions** -> **Add** -> **User Permission** 
5. Now when you login you will be able to select the *pocket* realm and sign in with passkey
