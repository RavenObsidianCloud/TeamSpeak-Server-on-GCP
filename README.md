# TeamSpeak 3 Server on Google Cloud Platform

A production-ready TeamSpeak 3 server deployed on GCP as part of my preparation for the Google Cloud Associate Cloud Engineer (ACE) certification exam.

## 🎯 Project Overview

This project demonstrates the deployment and securing of a voice communication server on Google Cloud Platform, covering key ACE exam domains including Compute Engine, VPC networking, security, and Linux system administration.

**Live Server:** `teamspeak.obsidiancloud.org`

## 🏗️ Architecture

- **Compute Engine:** e2-micro instance (Always Free tier eligible)
- **Operating System:** Debian 12 (Bookworm)
- **Region/Zone:** us-west4-b (Las Vegas)
- **Networking:** Static external IP with custom DNS A record
- **Security:** Multi-layer approach (GCP firewall + OS-level ufw)
- **Service Management:** systemd for automatic startup and restart

## 🔒 Security Features

- **GCP Firewall Rules:**
  - SSH restricted to specific source IPs
  - Only TeamSpeak voice port (UDP 9987) exposed
  - All other ports blocked by default

- **OS-Level Firewall (ufw):**
  - Defense-in-depth strategy
  - Explicit allow rules for SSH and TeamSpeak only
  - Default deny for all other incoming traffic

- **Service Account:** Dedicated non-root user for TeamSpeak process

## 📚 What I Learned

### ACE Exam Topics Covered:
- ✅ **Compute Engine:** VM provisioning, machine types, free tier optimization
- ✅ **VPC Networking:** Firewall rules, source IP filtering, network tags
- ✅ **External IP Management:** Static vs. ephemeral IPs
- ✅ **Cloud DNS Integration:** A record configuration with third-party registrar
- ✅ **Security Best Practices:** Principle of least privilege, defense in depth
- ✅ **Linux Administration:** systemd services, package management, user permissions
- ✅ **Troubleshooting:** SSH connectivity issues, firewall debugging

### Technical Skills:
- Multi-layer firewall configuration
- Service automation with systemd
- DNS record management
- Cloud cost optimization (running on free tier)
- SSH key management and access control
- Cloud Shell vs. local gcloud CLI usage

## 🛠️ Technologies Used

- **Google Cloud Platform**
  - Compute Engine
  - VPC Networking
  - Cloud Firewall
  - Static IP Addresses
  
- **Linux/System Administration**
  - Debian 12
  - systemd
  - ufw (Uncomplicated Firewall)
  - bash scripting

- **Application**
  - TeamSpeak 3 Server (v3.13.7)

## 📖 Documentation

- **[Step-by-Step Setup Guide](https://github.com/RavenObsidianCloud/TeamSpeak-Server-on-GCP/blob/main/Docs/step-by-step-guide.md)** - Complete deployment instructions
- **[Troubleshooting Guide](https://github.com/RavenObsidianCloud/TeamSpeak-Server-on-GCP/blob/main/Docs/Troubleshooting.md)** - Common issues and solutions
- **[Lessons Learned](https://github.com/RavenObsidianCloud/TeamSpeak-Server-on-GCP/blob/main/Docs/lessons-learned.md)** - Mistakes, insights, and improvements

## 💰 Cost Analysis

**Monthly Cost:** $0.00 (free tier)

The e2-micro instance in us-west4 qualifies for GCP's Always Free tier, which includes:
- 1 e2-micro instance per month
- 30 GB standard persistent disk
- 1 GB network egress per month (US regions)

Estimated usage for 15 users with moderate voice chat: Well within free tier limits.

## 🚀 Quick Start
```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/teamspeak-gcp.git

# Follow the detailed setup guide
cd teamspeak-gcp
cat docs/step-by-step-guide.md
```

## 🎓 ACE Certification Connection

This project directly maps to the following ACE exam sections:

**Section 1: Setting up a cloud solution environment**
- Understanding of projects and resource hierarchy
- Managing billing configuration

**Section 2: Planning and configuring a cloud solution**
- Compute Engine resources (machine types, regions/zones)
- Network resources (VPC, firewall rules, static IPs)

**Section 3: Deploying and implementing a cloud solution**
- Deploying Compute Engine resources
- Configuring network connectivity

**Section 4: Ensuring successful operation of a cloud solution**
- Managing Compute Engine resources
- Monitoring and logging (systemd status)

**Section 5: Configuring access and security**
- Managing IAM (service accounts, SSH keys)
- Implementing security controls (firewalls, IP restrictions)

## 🔗 Connect With Me

- **LinkedIn:** [Your LinkedIn]
- **Portfolio:** [Your Website]
- **Certification Progress:** Targeting GCP ACE exam March 2026

## 📝 License

This project is for educational purposes as part of GCP certification preparation.

## 🙏 Acknowledgments

- Google Cloud Platform documentation
- TeamSpeak 3 Server documentation
- ACE study community

---

**Status:** ✅ Production - Server operational since February 2026
