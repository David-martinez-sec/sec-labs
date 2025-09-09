# 🛡️ Network Hardening Report (Initial Findings)

**Date:** 2025-09-08  
**Assessor:** David Martinez  
**Scope:** Local internal network scan  

---

## 1. Executive Summary
An Nmap scan of the internal network identified **7 active hosts**. Several devices expose unnecessary or insecure services (e.g., Telnet, VNC, UPnP) that increase the attack surface. Immediate focus should be placed on disabling unused services, ensuring devices are patched, and segmenting IoT devices from administrative hosts.  

---

## 2. Key Findings

### 🔹 Gateway Device (Router)
- **Open ports:** DNS, HTTP/HTTPS, UPnP.  
- **Risks:**  
  - UPnP can expose internal devices if misconfigured.  
  - Web admin interface may be accessible if default credentials are weak.  
- **Recommendations:**  
  - Disable UPnP unless required.  
  - Restrict admin access (LAN-only, strong password).  
  - Ensure firmware is updated.  

---

### 🔹 Google IoT Devices
- **Open ports:** HTTP variants (8008, 8009, 8443), other Chromecast/Home service ports.  
- **Risks:**  
  - IoT devices often expose unnecessary services.  
- **Recommendations:**  
  - Place IoT devices on a separate VLAN or guest Wi-Fi network.  
  - Restrict unnecessary inbound traffic.  

---

### 🔹 Unknown Device
- **Observed behavior:** Only filtered ports returned.  
- **Risks:** Could indicate a firewall or a device configured to drop probes.  
- **Recommendations:**  
  - Identify the device via DHCP logs or MAC vendor lookup.  
  - Verify if it should be on the network.  

---

### 🔹 macOS-like Host
- **Open ports:** SSH, Kerberos, Apple remote services, UPnP, VNC.  
- **Risks:**  
  - Multiple remote management protocols exposed.  
  - UPnP again introduces unnecessary exposure.  
- **Recommendations:**  
  - Disable unneeded services (e.g., VNC if not required).  
  - Restrict SSH to known IPs.  
  - Apply patches and enforce strong authentication.  

---

### 🔹 Raspberry Pi (Kali Host)
- **Open ports:** SSH.  
- **Risks:**  
  - SSH exposed → brute force target if weak credentials are used.  
- **Recommendations:**  
  - Enforce key-based authentication.  
  - Change default SSH port or restrict to admin IPs.  
  - Regularly patch and update.  

---

## 3. Overall Recommendations
1. **Network Segmentation**  
   - Isolate IoT devices from critical admin hosts.  
2. **Disable Unused Services**  
   - Close VNC, UPnP, Telnet, RPC if not actively used.  
3. **Update & Patch Management**  
   - Ensure all devices are updated with latest firmware/security patches.  
4. **Access Controls**  
   - Use firewall rules to restrict inbound access.  
   - Apply least privilege — only allow what’s necessary.  
5. **Monitoring & Logging**  
   - Enable router logs, monitor unusual port activity.  
   - Run periodic vulnerability scans in addition to Nmap.  

---

## 4. Next Steps
- Perform **version detection scan** (`nmap -sV`) to identify software versions.  
- Validate device ownership for unknown hosts.  
- Begin patching/disabling services based on priority risks (UPnP, VNC).  
