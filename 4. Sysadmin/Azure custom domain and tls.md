## Overview

This guide covers mapping a custom DNS hostname to an Azure resource and configuring a valid TLS certificate for it. Two resource types are covered: App Service and Virtual Machine.

The goal is to serve traffic over:

```
https://app.yourdomain.com
```

with a certificate that matches the hostname. Browsing directly to a public IP will almost never produce a valid certificate match and should not be the end state for any production or lab environment.

---

## Prerequisites

|Requirement|Detail|
|---|---|
|Custom domain|A domain you control with access to its DNS records|
|Azure resource|App Service or VM with a public IP|
|Public IP assignment|Must be set to **Static**, not Dynamic|
|DNS provider access|Azure DNS, Cloudflare, GoDaddy, Route 53, Namecheap, etc.|

---

## Step 1 — Identify the Azure Resource Behind the IP

Before touching DNS, confirm what Azure resource owns the public IP.

1. In the Azure Portal, search for **Public IP addresses**
2. Locate your IP and open it
3. Check the **Associated to** field

The IP may be attached to any of the following:

- Virtual machine NIC
- Load balancer
- Application Gateway
- Azure Firewall
- NAT Gateway
- AKS ingress

> **App Service note:** If the resource is an App Service, do not use the raw public IP. Use the App Service hostname (`yourapp.azurewebsites.net`) as the CNAME target instead.

---

## Step 2 — Set the Public IP to Static

Dynamic IPs change on deallocate/start cycles. Set it to static before creating DNS records.

1. Open the **Public IP address** resource
2. Go to **Configuration**
3. Set **Assignment** to **Static**
4. Save

---

## Step 3 — Choose Your Hostname

Pick a subdomain unless you specifically need the root domain.

```
app.yourdomain.com
portal.yourdomain.com
api.yourdomain.com
```

A subdomain is cleaner for lab and application use. Root domain (`yourdomain.com`) requires additional DNS handling and is typically reserved for primary web presence.

---

## Step 4 — Create the DNS Record

### For a VM or Public IP — A Record

Create an A record at your DNS provider:

|Field|Value|
|---|---|
|Type|A|
|Name|app|
|Value|`<your-public-ip>`|
|TTL|300|

This makes `app.yourdomain.com` resolve to your Azure public IP.

### For App Service — CNAME Record

|Field|Value|
|---|---|
|Type|CNAME|
|Name|app|
|Value|`yourapp.azurewebsites.net`|
|TTL|300|

### Verify DNS Resolution

Do not proceed until DNS resolves correctly.

```powershell
# Windows
nslookup app.yourdomain.com
```

```bash
# Linux / macOS
dig app.yourdomain.com
```

Expected output for A record: `app.yourdomain.com -> <your-public-ip>`  
Expected output for CNAME: `app.yourdomain.com -> yourapp.azurewebsites.net`

---

## Path A — App Service

### Add the Custom Domain

1. In the Azure Portal, open your **App Service**
2. Go to **Custom domains**
3. Click **Add custom domain**
4. Enter `app.yourdomain.com`
5. Azure will validate DNS — if a TXT verification record is required, add it at your DNS provider:

|Field|Value|
|---|---|
|Type|TXT|
|Name|`asuid.app`|
|Value|`<azure-verification-value>`|

6. Once validation passes, click **Add**

### Certificate Options

**Option 1 — Azure-managed certificate (simplest)**

1. Go to **Certificates** in your App Service
2. Select **Managed certificates**
3. Click **Add certificate**
4. Select `app.yourdomain.com`
5. Azure issues and manages renewal automatically

Limitations: does not support wildcard certificates; requires public DNS validation.

**Option 2 — Bring your own certificate**

You need a `.pfx` file containing the certificate and private key. The certificate SAN must include `app.yourdomain.com`.

1. Go to **Certificates**
2. Click **Upload certificate**
3. Select your `.pfx` and enter the password
4. Upload

### Bind the Certificate

1. Go to **TLS/SSL settings**
2. Go to **Bindings**
3. Click **Add TLS/SSL Binding**
4. Select hostname: `app.yourdomain.com`
5. Select your certificate
6. Select **SNI SSL**
7. Save

### Force HTTPS

1. In **TLS/SSL settings**, enable **HTTPS Only**
2. Set minimum TLS version to **TLS 1.2** or higher

---

## Path B — Virtual Machine

### Step 1 — Obtain a Certificate

The certificate must be issued for `app.yourdomain.com`. Common sources:

|Source|Use Case|
|---|---|
|Let's Encrypt (Certbot)|Public-facing, free, auto-renewal|
|DigiCert / Sectigo / GlobalSign|Commercial, paid|
|Internal CA|Internal/lab environments only|

For a public-facing VM, Let's Encrypt via Certbot is the standard choice.

### Step 2 — Install the Certificate on the Web Server

**IIS (Windows)**

1. Open **IIS Manager**
2. Select the server node
3. Open **Server Certificates**
4. Click **Import** and select your `.pfx`
5. Navigate to your website
6. Click **Bindings**
7. Add or edit the HTTPS binding:

|Field|Value|
|---|---|
|Type|https|
|Hostname|`app.yourdomain.com`|
|Port|443|
|Certificate|Your imported certificate|

8. Save

**NGINX (Linux)**

```nginx
server {
    listen 80;
    server_name app.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name app.yourdomain.com;

    ssl_certificate     /etc/letsencrypt/live/app.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/app.yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:<app-port>;
    }
}
```

Reload NGINX after editing:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### Step 3 — Open Firewall Ports

**Azure NSG — Inbound rules required:**

|Port|Protocol|Purpose|
|---|---|---|
|80|TCP|HTTP (redirect to HTTPS)|
|443|TCP|HTTPS|

**VM OS firewall:**

Windows:

```powershell
New-NetFirewallRule -DisplayName "Allow HTTP"  -Direction Inbound -Protocol TCP -LocalPort 80  -Action Allow
New-NetFirewallRule -DisplayName "Allow HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
```

Linux (UFW):

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

---

## Verification

Browse to:

```
https://app.yourdomain.com
```

Click the lock icon in the browser and confirm:

- **Issued to:** `app.yourdomain.com`
- **SAN includes:** `DNS Name: app.yourdomain.com`

The browser warning clears only when the hostname in the URL matches the certificate. Browsing to the raw IP will still show a warning — that is expected and is not a problem once DNS and the hostname binding are correct.

---

## Common Issues

|Symptom|Likely Cause|Fix|
|---|---|---|
|DNS not resolving|Record not yet propagated|Wait and re-test with `nslookup` or `dig`|
|Azure domain validation fails|TXT record missing or not propagated|Add the `asuid.<subdomain>` TXT record and wait|
|Certificate warning persists|Browsing by IP instead of hostname|Always use the hostname URL|
|Dynamic IP changed after VM start|Public IP set to Dynamic|Set Public IP assignment to Static|
|NGINX reload fails|Config syntax error|Run `sudo nginx -t` before reloading|
|NSG blocking traffic|Port 443 not open inbound|Add inbound NSG rule for TCP 443|
|App Service cert not issuing|DNS not validated or propagated|Confirm A/CNAME record resolves before requesting cert|