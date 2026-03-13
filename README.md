# Localhost to Public (Cloudflare Tunnel)

A concise guide to securely exposing a local development project to the internet on a **real, custom domain** using Cloudflare Tunnels (`cloudflared`).

## Prerequisites
- A domain whose DNS is actively managed by Cloudflare.
- Your project running locally (e.g., on port `3000`).

---

## Step 1: Install `cloudflared` CLI
Download and install the Cloudflare daemon from the [official downloads page](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/downloads/). For Linux (Debian/Ubuntu):
```bash
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb
```
*(macOS users can use: `brew install cloudflare/cloudflare/cloudflared`)*

Verify the installation by checking the version:
```bash
cloudflared --version
```

## Step 2: Login to Cloudflare
Authenticate the CLI with your Cloudflare account.
```bash
cloudflared tunnel login
```
*A browser link will open (or will be printed in the terminal). Open it and select the specific domain you want to use.*

## Step 3: Create the Tunnel
Create a named tunnel (e.g., `my-app-tunnel`) to establish the connection.
```bash
cloudflared tunnel create my-app-tunnel
```
*Take note of the **Tunnel UUID**. A credentials file will be generated automatically at `~/.cloudflared/<UUID>.json`.*

## Step 4: Route Traffic to Your Domain
Attach your domain (or a subdomain) to the tunnel. This creates the necessary CNAME record in your Cloudflare dashboard.
```bash
cloudflared tunnel route dns my-app-tunnel api.yourdomain.com
```

## Step 5: Configure the Ingress Rules
Create a configuration file at `~/.cloudflared/config.yml` to define where incoming web traffic should go.

> **Tip: Locating your `config.yml`**
> - On **Linux/macOS**, this file is usually located in the `~/.cloudflared/` directory. You can open and edit it using `nano ~/.cloudflared/config.yml` or your preferred text editor.
> - On **Windows**, it is typically located in `%USERPROFILE%\.cloudflared\`.

```yaml
tunnel: my-app-tunnel
credentials-file: /home/USERNAME/.cloudflared/<UUID>.json

ingress:
  - hostname: api.yourdomain.com
    service: http://localhost:3000
  - service: http_status:404
```
*(Remember to replace `USERNAME`, `<UUID>`, `api.yourdomain.com`, and `http://localhost:3000` with your actual details).*

## Step 6: Start the Tunnel
Run your tunnel to connect your local app to the public web!
```bash
cloudflared tunnel run my-app-tunnel
```

---

### Create Temporary Public URL
If you ever need a quick, temporary public URL **without** configuring a custom domain, you can skip steps 2-6 and simply run the following command (assuming your app is on port `3000`):
```bash
cloudflared tunnel --url http://localhost:3000
```

---

## Managing Your Tunnel

### Stopping a Running Tunnel
If your tunnel is actively running in the foreground of your terminal, simply press `Ctrl + C` to stop it. 

### Deleting a Tunnel
To permanently delete a tunnel and remove its associated UUID and credentials, run the following:
```bash
cloudflared tunnel delete my-app-tunnel
```
*Note: You may also want to manually remove the `~/.cloudflared/<UUID>.json` file if it was not automatically removed.*