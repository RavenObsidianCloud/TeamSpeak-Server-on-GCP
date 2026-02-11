# Troubleshooting Guide

Common issues encountered during deployment and their solutions.

## SSH Connection Issues

### Problem: "Connection timed out" when SSH'ing

**Possible Causes:**
1. Your IP address changed
2. Firewall rule is too restrictive
3. VM is stopped or in an error state

**Solutions:**
```bash
# 1. Check your current IP
curl ifconfig.me

# 2. Update firewall rule with new IP
gcloud compute firewall-rules update default-allow-ssh \
  --source-ranges=YOUR_NEW_IP/32

# 3. Check VM status
gcloud compute instances list

# 4. Start VM if stopped
gcloud compute instances start teamspeakvm --zone=us-west4-b
```

### Problem: "Could not SSH into the instance. SSH key has not propagated"

**Solution:**
```bash
# Force regenerate SSH keys
gcloud compute ssh teamspeakvm --zone=us-west4-b --force-key-file-overwrite
```

### Problem: Locked out of SSH after updating firewall rules

**Solution:**

Use the **Console SSH button** (bypasses firewall rules):
1. Go to GCP Console
2. Compute Engine → VM Instances
3. Click **SSH** button next to your VM
4. Fix the firewall rule from there

---

## Cloud Shell vs. Local Computer SSH

### Problem: SSH works from Cloud Shell but not from local computer (or vice versa)

**Cause:** Different source IPs

**Solution:** Add both IPs to the firewall rule
```bash
# Get Cloud Shell IP (run IN Cloud Shell)
curl ifconfig.me

# Get your home IP (run on local computer)
curl ifconfig.me

# Update firewall to allow both
gcloud compute firewall-rules update default-allow-ssh \
  --source-ranges=HOME_IP/32,CLOUD_SHELL_IP/32
```

---

## TeamSpeak Connection Issues

### Problem: Can't connect to TeamSpeak server

**Diagnosis steps:**
```bash
# 1. Check if VM is running
gcloud compute instances list

# 2. Check if TeamSpeak service is running
gcloud compute ssh teamspeakvm --zone=us-west4-b
sudo systemctl status teamspeak.service

# 3. Check firewall rules
gcloud compute firewall-rules list | grep teamspeak

# 4. Test port is open
sudo netstat -tulpn | grep 9987
```

**Common fixes:**
```bash
# Restart TeamSpeak service
sudo systemctl restart teamspeak.service

# Check TeamSpeak logs
sudo -u teamspeak cat /home/teamspeak/teamspeak3-server_linux_amd64/logs/ts3server_*.log

# Verify ufw allows port 9987
sudo ufw status | grep 9987
```

---

## DNS Issues

### Problem: Domain doesn't resolve to server IP

**Diagnosis:**
```bash
# Check DNS propagation
dig teamspeak.yourdomain.com

# Test with direct IP (bypasses DNS)
# If this works, it's a DNS issue
ping YOUR_STATIC_IP
```

**Solutions:**
- Wait longer (DNS can take up to 24 hours, usually 5-30 minutes)
- Verify A record is correct in your DNS provider
- Clear DNS cache locally: `ipconfig /flushdns` (Windows) or `sudo dscacheutil -flushcache` (Mac)

---

## Static IP Issues

### Problem: IP address changed after reboot

**Cause:** Using ephemeral IP instead of static

**Solution:**
```bash
# Check if IP is reserved
gcloud compute addresses list

# If not reserved, promote it to static
gcloud compute addresses create teamspeak-static-ip \
  --addresses=YOUR_CURRENT_IP \
  --region=us-west4
```

---

## systemd Service Issues

### Problem: TeamSpeak doesn't start on boot

**Diagnosis:**
```bash
# Check if service is enabled
sudo systemctl is-enabled teamspeak.service

# Check service status
sudo systemctl status teamspeak.service

# View systemd logs
sudo journalctl -u teamspeak.service -n 50
```

**Solutions:**
```bash
# Enable the service
sudo systemctl enable teamspeak.service

# Verify the service file is correct
sudo cat /etc/systemd/system/teamspeak.service

# Reload systemd if you edited the file
sudo systemctl daemon-reload
sudo systemctl restart teamspeak.service
```

---

## Permission Errors

### Problem: "Permission denied" when downloading TeamSpeak files

**Cause:** Trying to write to directory where user lacks permissions

**Solution:**
```bash
# Make sure you're in the teamspeak user's home directory
cd /home/teamspeak

# Or explicitly switch to teamspeak user
sudo -u teamspeak -s
cd ~
```

---

## Firewall Conflicts

### Problem: GCP firewall allows traffic but ufw blocks it

**Diagnosis:**
```bash
# Check ufw status
sudo ufw status verbose

# Check if rule exists
sudo ufw status | grep 9987
```

**Solution:**
```bash
# Add missing rule
sudo ufw allow 9987/udp

# Reload ufw
sudo ufw reload
```

---

## Cost/Billing Issues

### Problem: Unexpected charges

**Check:**
1. Verify you're using e2-micro (not e2-small or larger)
2. Check your region is free-tier eligible (us-west1, us-central1, us-east1, us-west4)
3. Monitor network egress (should be under 1GB/month for normal use)
```bash
# Verify machine type
gcloud compute instances describe teamspeakvm \
  --zone=us-west4-b \
  --format="get(machineType)"

# Should end in "e2-micro"
```

---

## Getting More Help

If you encounter an issue not listed here:

1. Check TeamSpeak logs:
```bash
   sudo -u teamspeak cat /home/teamspeak/teamspeak3-server_linux_amd64/logs/ts3server_*.log
```

2. Check system logs:
```bash
   sudo journalctl -xe
```

3. Verify all services are running:
```bash
   sudo systemctl status teamspeak.service
   sudo ufw status
   gcloud compute instances list
```

4. Open an issue in this repository with:
   - Error message (full text)
   - What you were trying to do
   - Steps you've already tried

---

**Remember:** Most issues are related to firewall rules or IP address changes. Always check these first!