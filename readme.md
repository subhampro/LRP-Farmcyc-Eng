# **Complete Setup Guide: SkyPort Panel with Cloudflare Tunnel**

## **Introduction**
This guide will walk you through setting up SkyPort Panel along with Cloudflare Tunnel to expose your panel using a custom domain securely. We will replace Ngrok with Cloudflare Tunnel to ensure a free and reliable setup.

---

## **Step 1: Install SkyPort Panel**
Run the following command to install the SkyPort Panel:
```sh
sudo apt-get install screen

screen -R -D skyport

bash <(curl -fsSL https://raw.githubusercontent.com/SoloPlayzDev/skyport-installer/refs/heads/main/install.sh)

CTRL+A+D --[Exit From Screen]
```
This script will install all necessary dependencies and configure the panel automatically.

---

## **Step 2: Install Cloudflare Tunnel**
### **1. Install Cloudflare Tunnel (Cloudflared)**
Run the following command to install `cloudflared`:
```sh
curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o cloudflared
sudo chmod +x cloudflared
sudo mv cloudflared /usr/local/bin/
```

### **2. Authenticate Cloudflare Tunnel**
Run:
```sh
cloudflared tunnel login
```
- This will open a browser window.
- Log in to your Cloudflare account and select your domain.
- Once authenticated, the credentials file will be saved in `/root/.cloudflared/cert.pem`.

### **3. Create a New Cloudflare Tunnel**
```sh
cloudflared tunnel create my-tunnel
```
- This command will generate a credentials file in `/root/.cloudflared/`.
- Take note of the tunnel UUID from the output (something like `b2bf354d-0059-4ffd-af7f-06b45a53b698`).

### **4. Configure the Tunnel**
Create or edit the Cloudflare Tunnel configuration file:
```sh
sudo mkdir -p /etc/cloudflared/

sudo nano /etc/cloudflared/config.yaml
```
Add the following content:
```yaml
tunnel: my-tunnel-2
credentials-file: /root/.cloudflared/my-tunnel.json
ingress:
  - hostname: panel2.prohostvps.com
    service: http://localhost:3001
  - service: http_status:404
```
Replace `panel.prohostvps.com` with your actual domain and ensure that the port (`3001`) matches your SkyPort Panel setup.

### **5. Add DNS Record in Cloudflare**
1. Go to **Cloudflare Dashboard** → **DNS Settings**.
2. Add a new **CNAME Record**:
   - **Name**: `panel` (This makes it `panel.yourdomain.com`)
   - **Target**: `your-tunnel-id.cfargotunnel.com`
   - **Proxy Status**: ✅ **Proxied**

### **6. Start Cloudflare Tunnel**
Run:
```sh
screen -R -D cloudflare
ls -l /root/.cloudflared --[get fine name here]
sudo nano /etc/cloudflared/config.yaml --[set file name]
cloudflared tunnel run my-tunnel
```
To run it in the background:
```sh
nohup cloudflared tunnel run my-tunnel > cloudflared.log 2>&1 &
```
Goto Port-Forward > Open The Link > Login Admin > Nodes > Add Node
---

## **Step 3: Install Wings for SkyPort**
Run the following command to install Wings (SkyPort Daemon):
```sh
bash <(curl -fsSL https://raw.githubusercontent.com/SoloPlayzDev/skyport-installer/refs/heads/main/wings.sh)

cd skyportd
npm i

Enter config code --[Generate From Node (npm run configure -- --panel http://localhost:3001 --key 681a5ab2-dfe0-4fe5-bfe0-dcef15f633d9 --secret)]
Processor : AMD EPYC 7763 64-Core Processor
Memory : 64GB
Disk : 100GB

screen -R -D wings
node .
CTRL+A+D --[Exit From Screen]
```
This script will set up the Wings service required for running game servers.



---

## **Final Verification**
1. **Ensure Cloudflare Tunnel is running**
   ```sh
   ps aux | grep cloudflared
   ```
   If not running, restart it using:
   ```sh
   cloudflared tunnel run my-tunnel
   ```

2. **Test Your Panel**
   - Open `https://panel.yourdomain.com` in your browser.
   - You should see the SkyPort Panel login page.

---

## **Conclusion**
✅ Now, your SkyPort Panel is set up with **Cloudflare Tunnel**, ensuring a secure and **free** way to expose your panel with a custom domain.

Let me know if you need further assistance! 🚀

