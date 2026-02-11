# Lessons Learned

Mistakes made, insights gained, and things I'd do differently next time.

## Mistakes and How I Fixed Them

### 1. Locked Myself Out with Firewall Rules

**What happened:**
I restricted the SSH firewall rule to only my home IP, forgetting that Cloud Shell uses a different IP address. This locked me out of SSH from Cloud Shell.

**The mistake:**
```bash
# Only allowed my home IP
gcloud compute firewall-rules update default-allow-ssh \
  --source-ranges=MY_HOME_IP/32
```

**What I learned:**
- Different access methods use different source IPs
- Cloud Shell has its own IP (`34.182.55.97`)
- Local computer has your ISP-assigned IP
- Console SSH button bypasses firewall rules entirely (uses IAP)

**The fix:**
```bash
# Allow both Cloud Shell and home IP
gcloud compute firewall-rules update default-allow-ssh \
  --source-ranges=HOME_IP/32,CLOUD_SHELL_IP/32
```

**Takeaway for ACE exam:** Understanding source IP filtering is critical. Always know where your traffic is coming from.

---

### 2. Confused Ephemeral vs. Static IPs

**What happened:**
Initially created the VM without attaching a static IP. The IP would have changed on every reboot, breaking the DNS record.

**What I learned:**
- **Ephemeral IPs** change when you stop/start a VM
- **Static IPs** are reserved and persist
- You can promote an ephemeral IP to static
- Static IPs have a small cost if not attached to a running resource

**The fix:**
Reserved a static IP first, then attached it to the VM during creation.

**Takeaway for ACE exam:** Know when to use static vs. ephemeral IPs. Static = services that need persistent addressing.

---

### 3. Wrong Working Directory for TeamSpeak User

**What happened:**
Tried to download TeamSpeak files while in my personal user's home directory (`/home/djgukie`). Got "Permission denied" error.

**The mistake:**
```bash
sudo -u teamspeak -s
# Still in /home/djgukie
wget https://...  # Permission denied!
```

**What I learned:**
- Each user has their own home directory with specific permissions
- When you `su` to a different user, you don't automatically change directories
- The `cd ~` command takes you to the current user's home directory

**The fix:**
```bash
sudo -u teamspeak -s
cd ~  # Now in /home/teamspeak
wget https://...  # Works!
```

**Takeaway for ACE exam:** Linux file permissions and user home directories are testable concepts.

---

### 4. Forgot to Install bzip2

**What happened:**
Debian 12 doesn't have `bzip2` installed by default. The `tar` command failed when trying to extract the TeamSpeak archive.

**The error:**
```
tar (child): cannot run bzip2: No such file or directory
```

**What I learned:**
- Different Linux distributions have different default packages
- Ubuntu often has more tools pre-installed
- Debian is more minimal (smaller footprint, but requires more manual setup)

**The fix:**
```bash
sudo apt-get install bzip2 -y
```

**Takeaway for ACE exam:** Know the differences between common Linux distributions and their package management.

---

### 5. Ran Commands in Wrong Environment

**What happened:**
Tried to run `systemctl` commands in Cloud Shell instead of on the actual VM. Got error: "System has not been booted with systemd as init system (PID 1)."

**What I learned:**
- **Cloud Shell** = temporary container for running gcloud commands
- **Your VM** = where your actual services run
- `systemctl` only works on systems using systemd (not in containers)

**The fix:**
SSH into the VM first, then run systemd commands there.

**Takeaway for ACE exam:** Understand the difference between the management plane (Cloud Shell/gcloud) and the workload plane (your VMs).

---

## Things I'd Do Differently

### 1. Use Infrastructure as Code (Terraform)

**Current approach:** Manual commands via gcloud CLI

**Better approach:** Define everything in Terraform:
- Version controlled
- Reproducible
- Easy to tear down and rebuild
- Can deploy to multiple environments

**Future improvement:** Create a `main.tf` file with all resources defined.

---

### 2. Set Up Automated Backups

**Current approach:** No backups configured

**Better approach:**
- Create a snapshot schedule for the boot disk
- Export TeamSpeak database regularly
- Store backups in Cloud Storage

**ACE exam relevance:** Disaster recovery and backup strategies are tested.

---

### 3. Add Monitoring and Alerting

**Current approach:** Manual checking with `systemctl status`

**Better approach:**
- Use Cloud Monitoring to track VM health
- Set up alerts for service failures
- Monitor CPU, memory, and network usage
- Create uptime checks

**Future improvement:** Configure Cloud Monitoring dashboards and alert policies.

---

### 4. Implement Least Privilege IAM

**Current approach:** Using my owner account for everything

**Better approach:**
- Create a service account for VM operations
- Use separate accounts for different tasks
- Enable 2FA on all accounts
- Audit IAM permissions regularly

**ACE exam relevance:** IAM and security best practices are heavily tested.

---

### 5. Document as I Go

**Current approach:** Built everything first, documented after

**Better approach:**
- Take screenshots during setup
- Note down every command as I run it
- Document errors in real-time
- Create a troubleshooting log

**Lesson:** Documentation is easiest when done concurrently, not retrospectively.

---

## Positive Takeaways

### What Worked Well:

1. **Using network tags for firewall rules** - Clean and scalable approach
2. **Defense-in-depth security** - GCP firewall + ufw provides redundancy
3. **systemd for service management** - Auto-start and restart capabilities
4. **Free tier optimization** - e2-micro in us-west4 = $0/month
5. **Custom domain** - Much easier for users than remembering an IP

---

## Skills Developed

### Technical:
- GCP Compute Engine management
- VPC networking and firewall configuration
- Linux system administration
- Service automation with systemd
- DNS configuration
- Security hardening

### Troubleshooting:
- SSH connectivity debugging
- Firewall rule analysis
- Permission and ownership issues
- Service startup problems

### Soft Skills:
- Breaking down complex problems
- Reading error messages carefully
- Systematic debugging approach
- Documentation and knowledge sharing

---

## Next Project Ideas

Building on what I learned:

1. **Load-balanced web application** - Explore instance groups and load balancers
2. **Automated CI/CD pipeline** - Use Cloud Build to deploy code
3. **Multi-tier architecture** - Frontend, backend, database separation
4. **Serverless application** - Cloud Functions + Cloud Run
5. **Monitoring dashboard** - Aggregated logging and metrics

---

## ACE Exam Preparation Impact

This project directly helped me understand:

- **Compute Engine** - Machine types, regions, zones, instance lifecycle
- **VPC Networking** - Firewall rules, static IPs, network tags
- **Security** - Defense in depth, least privilege, IAM
- **Linux Administration** - Essential for managing VMs
- **Cost Optimization** - Free tier eligibility and monitoring
- **Troubleshooting** - Real-world debugging scenarios

**Confidence level:** This single project covered ~30% of the ACE exam objectives.

---

## Final Thoughts

The best way to learn cloud engineering is to **build real things and break them**. Every error message taught me something. Every mistake made me a better engineer.

If you're studying for the ACE exam, don't just watch videos or read documentation—deploy actual infrastructure. You'll remember concepts far better when you've debugged them at 2 AM.

---

**"I hear and I forget. I see and I remember. I do and I understand."** - Confucius