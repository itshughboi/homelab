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

### Proxmox OIDC Setup via Pocket

Setting up OpenID Connect allows for seamless, secure logins (like passkeys) into your Proxmox VE environment.
1. Pocket Configuration

    Create Application: Use the instructions provided in your Pocket instance.

    Client Launch URL: Must include the port 8006.

        Example: https://10.10.10.1:8006 or https://pve-srv-1.hughboi.cc:8006

    Groups: If a dedicated User Group doesn't exist, create one now.

    Allowed User Groups: Add your group here.

        ⚠️ Troubleshooting: If you receive an "Unauthorized" message, ensure the Scopes in your client include groups or profile so the group claim is passed.

2. Proxmox Configuration

Navigate to Datacenter > Permissions > Realms > Add > OpenID Connect Server.
Field	Value/Setting
Issuer URL	https://pocket.hughboi.cc
Realm	pocket (or your preferred name)
Client ID	From Pocket
Client Key	From Pocket
Autocreate Users	Checked (Required for first-time login)
Username Claim	username (Prevents UID-based naming)
Scopes	openid email profile
3. User Provisioning & Permissions

    Initial Sync: Sign out of Proxmox. Log back in using the Pocket realm. This triggers the "Autocreate" function.

    Elevate Permissions: * Sign out and log back in as a Proxmox Admin (root@pam).

        Go to Permissions > Add > User Permission.

        Select your newly created user (e.g., hugh@pocket) and assign the appropriate role (e.g., Administrator or PVEVMAdmin).

    Final Test: Log out and sign back in via the Pocket realm using your passkey.
