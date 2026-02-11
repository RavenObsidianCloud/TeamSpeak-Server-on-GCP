# Step-by-Step Setup Guide: TeamSpeak 3 on GCP

This guide walks through the complete deployment of a secure TeamSpeak 3 server on Google Cloud Platform.

## Prerequisites

- GCP account with billing enabled
- Basic familiarity with Linux command line
- Domain name (optional, but recommended)
- gcloud CLI installed (for local setup) OR access to Cloud Shell

## Table of Contents

1. [Reserve Static IP Address](#1-reserve-static-ip-address)
2. [Create the VM Instance](#2-create-the-vm-instance)
3. [Configure Firewall Rules](#3-configure-firewall-rules)
4. [Install TeamSpeak Server](#4-install-teamspeak-server)
5. [Configure systemd Service](#5-configure-systemd-service)
6. [Set Up OS-Level Firewall](#6-set-up-os-level-firewall)
7. [Configure DNS](#7-configure-dns)
8. [Test and Verify](#8-test-and-verify)

---

## 1. Reserve Static IP Address

Reserve a static external IP so your server address doesn't change on reboots.
```bash
# Reserve a static IP in your chosen region
gcloud compute addresses create teamspeak-static-ip \
  --region=us-west4

# View the reserved IP address
gcloud compute addresses describe teamspeak-static-ip \
  --region=us-west4 \
  --format="get(address)"
```

**Note:** Save this IP address - you'll need it for DNS configuration.

---

## 2. Create the VM Instance

Create an e2-micro instance (free tier eligible) with the static IP attached.
```bash
# Create the VM
gcloud compute instances create teamspeakvm \
  --zone=us-west4-b \
  --machine-type=e2-micro \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=10GB \
  --boot-disk-type=pd-standard \
  --tags=teamspeak-server \
  --address=teamspeak-static-ip

# Verify the VM is running
gcloud compute instances list
```

**Key Parameters:**
- `--machine-type=e2-micro`: Free tier eligible, sufficient for 15 users
- `--zone=us-west4-b`: Las Vegas region (low latency for US West Coast)
- `--tags=teamspeak-server`: Used for targeted firewall rules
- `--address=teamspeak-static-ip`: Attaches the reserved static IP

---

## 3. Configure Firewall Rules

### A. Create TeamSpeak Firewall Rule
```bash
# Allow UDP port 9987 for TeamSpeak voice traffic
gcloud compute firewall-rules create allow-teamspeak \
  --allow=udp:9987 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=teamspeak-server \
  --description="Allow TeamSpeak voice traffic"
```

### B. Restrict SSH Access (Security Best Practice)
```bash
# Get your public IP address
curl ifconfig.me

# Update SSH rule to only allow your IP
# Replace YOUR_IP with the IP from above
gcloud compute firewall-rules update default-allow-ssh \
  --source-ranges=YOUR_IP/32
```

**Important:** If you also use Cloud Shell, add its IP:
```bash
# Get Cloud Shell IP
curl ifconfig.me  # (run this IN Cloud Shell)

# Update to allow both your home IP and Cloud Shell
gcloud compute firewall-rules update default-allow-ssh \
  --source-ranges=YOUR_HOME_IP/32,CLOUD_SHELL_IP/32
```

---

## 4. Install TeamSpeak Server

SSH into your VM and install TeamSpeak.
```bash
# SSH into the VM
gcloud compute ssh teamspeakvm --zone=us-west4-b
```

### Once connected to the VM:
```bash
# Update system packages
sudo apt-get update && sudo apt-get upgrade -y

# Install required dependencies
sudo apt-get install bzip2 -y

# Create dedicated user for TeamSpeak
sudo adduser --disabled-login teamspeak
# Press Enter through all prompts

# Switch to teamspeak user
sudo -u teamspeak -s

# Navigate to home directory
cd ~

# Download TeamSpeak 3 Server
wget https://files.teamspeak-services.com/releases/server/3.13.7/teamspeak3-server_linux_amd64-3.13.7.tar.bz2

# Extract the archive
tar xvf teamspeak3-server_linux_amd64-3.13.7.tar.bz2

# Navigate to extracted directory
cd teamspeak3-server_linux_amd64

# Accept the license
touch .ts3server_license_accepted

# Start TeamSpeak for the first time
./ts3server_startscript.sh start
```

**🚨 CRITICAL:** When you start TeamSpeak for the first time, it will display a **server admin token**. Copy and save this immediately! It looks like:
```
token=aBcDeFgHiJkLmNoPqRsTuVwXyZ1234567890
```

You need this to claim server admin rights when you first connect.
```bash
# Stop the manual instance (we'll use systemd next)
./ts3server_startscript.sh stop

# Exit back to your regular user
exit
```

---

## 5. Configure systemd Service

Set up TeamSpeak as a systemd service for automatic startup and management.
```bash
# Create the service file
sudo nano /etc/systemd/system/teamspeak.service
```

**Paste this configuration:**
```ini
[Unit]
Description=TeamSpeak 3 Server
After=network.target

[Service]
User=teamspeak
Group=teamspeak
Type=forking
WorkingDirectory=/home/teamspeak/teamspeak3-server_linux_amd64
ExecStart=/home/teamspeak/teamspeak3-server_linux_amd64/ts3server_startscript.sh start
ExecStop=/home/teamspeak/teamspeak3-server_linux_amd64/ts3server_startscript.sh stop
Restart=on-failure
RestartSec=15

[Install]
WantedBy=multi-user.target
```

**Save the file:** Ctrl+X, then Y, then Enter
```bash
# Reload systemd to recognize the new service
sudo systemctl daemon-reload

# Enable the service to start on boot
sudo systemctl enable teamspeak.service

# Start the service
sudo systemctl start teamspeak.service

# Verify it's running
sudo systemctl status teamspeak.service
```

You should see **"active (running)"** in green.

**Test auto-start:**
```bash
# Reboot the VM
sudo reboot

# Wait 30 seconds, then SSH back in
gcloud compute ssh teamspeakvm --zone=us-west4-b

# Check if TeamSpeak started automatically
sudo systemctl status teamspeak.service
```

---

## 6. Set Up OS-Level Firewall

Add an additional security layer with ufw (Uncomplicated Firewall).
```bash
# Install ufw
sudo apt-get install ufw -y

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (CRITICAL: Don't lock yourself out!)
sudo ufw allow 22/tcp

# Allow TeamSpeak voice traffic
sudo ufw allow 9987/udp

# Enable the firewall
sudo ufw enable
# Type 'y' when prompted

# Verify the rules
sudo ufw status
```

**Expected output:**
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
9987/udp                   ALLOW       Anywhere
```

---

## 7. Configure DNS

Point your domain to the static IP address.

### Using Squarespace (or any DNS provider):

1. Log in to your DNS provider
2. Go to DNS settings for your domain
3. Add an **A record:**
   - **Host:** `teamspeak` (creates `teamspeak.yourdomain.com`)
   - **Type:** A
   - **Data/Points to:** Your static IP address
   - **TTL:** Auto or 3600

4. Save the record

**Verify DNS propagation:**
```bash
# Wait 5-30 minutes, then test
ping teamspeak.yourdomain.com

# Or use dig
dig teamspeak.yourdomain.com
```

---

## 8. Test and Verify

### A. Test TeamSpeak Connection

1. Download TeamSpeak 3 Client: https://teamspeak.com/en/downloads/
2. Open the client
3. Click **Connections** → **Connect**
4. **Server Address:** `teamspeak.yourdomain.com` (or your static IP)
5. **Nickname:** Your name
6. Click **Connect**

### B. Claim Server Admin Rights

When you first connect, the server will prompt for a privilege key:
1. Click **Permissions** → **Use Privilege Key**
2. Paste the token you saved during installation
3. Click **OK**

You now have full server admin rights!

### C. Verify Security
```bash
# Check firewall rules
gcloud compute firewall-rules list

# Check ufw status on VM
sudo ufw status verbose

# Check systemd service status
sudo systemctl status teamspeak.service

# View recent SSH login attempts (look for suspicious activity)
sudo cat /var/log/auth.log | grep "Failed password"
```

---

## 9. Post-Deployment

### Regular Maintenance:
```bash
# Update system packages monthly
sudo apt-get update && sudo apt-get upgrade -y

# Check TeamSpeak logs if issues arise
sudo -u teamspeak cat /home/teamspeak/teamspeak3-server_linux_amd64/logs/ts3server_*.log

# Restart TeamSpeak if needed
sudo systemctl restart teamspeak.service
```

### Monitor Costs:
```bash
# Check VM uptime
gcloud compute instances describe teamspeakvm \
  --zone=us-west4-b \
  --format="get(status)"

# Verify you're in the free tier
# e2-micro in us-west4 = free
# Monitor in GCP Console → Billing
```

---

## Troubleshooting

See [troubleshooting.md](Docs/Troubleshooting.md) for common issues and solutions.

---

## Next Steps

- Configure TeamSpeak server settings (permissions, channels)
- Set up automated backups
- Add monitoring and alerting
- Explore adding a music bot

---

**Questions or issues?** Open an issue in this repo!
