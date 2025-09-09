# 🔐 Network & System Hardening Report

**Date:** 2025-09-09  
**Assessor:** David Martinez  
**Scope:** Local Home Network & Devices (Raspberry Pi, Mac, Gateway, IoT devices)  

---

## 1. Completed Hardening Steps

### 🔹 SSH Security (Mac & Raspberry Pi)
- Created **SSH key pairs** for both Pi and Mac.  
- Disabled password-based authentication.  
- Enforced **key-only login** for remote access.  
- ✅ Eliminates brute-force password attacks.  

### 🔹 Raspberry Pi Firewall & Updates
- Installed and configured **UFW (Uncomplicated Firewall):**
  - Default policy: **deny all incoming** connections.  
  - Default policy: **allow all outgoing** connections.  
  - Allowed only **SSH (port 22/tcp)** for remote management.  
  - Enabled firewall and verified with `ufw status`.  
- Applied full system updates with `apt update && apt full-upgrade`.  
- ✅ Raspberry Pi only exposes what is strictly necessary.  

### 🔹 macOS Hardening
- Disabled unnecessary **Sharing** services (Screen Sharing, Remote Management, File Sharing).  
- Enabled **Firewall** with **Stealth Mode**.  
- Restricted **AirDrop to Contacts Only**.  
- Ensured automatic updates enabled.  
- ✅ Mac is now less discoverable and exposes minimal attack surface.  

### 🔹 Router/Gateway Security
- Disabled **Remote Admin** (LAN-only router management).  
- Disabled **UPnP** (prevents devices from auto-opening ports).  
- Verified **WPA2/WPA3 Wi-Fi with strong password**.  
- ✅ Reduced external attack surface and unauthorized device exposure.  

---

## 2. Next Steps

1. **Network Segmentation**  
   - Move IoT devices (digital picture frame, Chromecast, Google Minis) onto **separate Guest Wi-Fi** or **dedicated IoT SSID**.  
   - If unavailable, consider upgrading to a router with VLAN support.  

2. **Recurring Audit Setup**  
   - Schedule automated **Nmap scans** from the Raspberry Pi (weekly).  
   - Collect and compare results to detect new or unauthorized devices.  

3. **IoT Device Security**  
   - Update firmware for picture frame, Chromecast, and Minis.  
   - Restrict cloud sync features if not needed.  
   - Secure linked accounts with strong, unique passwords.  

4. **Backups**  
   - Export router configuration.  
   - Backup Pi configs and SSH keys to external drive.  
   - Test restore process.  

5. **Future Considerations**  
   - Add **VPN** access for remote management instead of port forwarding.  
   - Deploy **Pi-hole or DNS filtering** for whole-network malware/ad protection.  
   - Explore full VLAN segmentation with custom firewall/router.  

---

## 3. Reflection

I enjoyed learning to create SSH keys and disable passwords so I can remote into my desktop and Pi safely to perform labs. Setting up a lightweight firewall directly in the terminal was also interesting—being able to configure it in text commands versus simply toggling a GUI switch gave me a stronger sense of control.  

I believe the most impactful change will be the one still ahead: segmenting my IoT devices from my main systems. These devices appear to present the greatest vulnerability, especially the digital picture frame, so isolating them will provide the strongest long-term protection.  

---
