# 🧪 Raspberry Pi IoT Network Segmentation Lab

### 📅 Date  
October 2025  

---

## 🎯 Goal & Motivation
The goal was to segment my network so that IoT devices could be isolated from my home network, while also learning about Linux and network configuration.  
I discovered that my Xfinity gateway offered limited options for VLANs or network segmentation, so I decided to use my **Raspberry Pi 4** as a small router to create a dedicated IoT subnet.

---

## ⚙️ Setup & Environment
I used a **Raspberry Pi 4** running **Kali Linux ARM**, configured for **headless SSH access**.  
The SD card image was prepared with a `wpa_supplicant.conf` file to enable Wi-Fi connectivity on boot.  
Once online, I connected to the Pi via SSH over Wi-Fi.  
The Pi was also **wired to my Xfinity gateway via Ethernet**, though the wired connection initially lacked a static configuration.

---

## 🧭 Process Overview
After establishing an SSH session, I inspected my network interfaces with commands like:

```bash
ip -br a
networkctl status eth0
```

I observed that **eth0** was *unmanaged* while **wlan0** was connected to my home Wi-Fi via NetworkManager.  
The first task was to configure **eth0** with a static IP so it could act as the upstream connection for the isolated IoT network.  

Initially, I tried managing it through **NetworkManager**, but this system was actually using **systemd-networkd** through Netplan.  
After switching to Netplan, I created a configuration file to assign a static IP to eth0 and remove the dynamically assigned one.  
Once applied, the Ethernet interface held a stable static IP that would later serve as the gateway interface.

---

## 🔧 Enabling IP Forwarding
After configuring the Ethernet interface, I enabled IP forwarding so the Pi could route packets between interfaces.  
This required editing the system’s kernel parameters:

```bash
sudo nano /etc/sysctl.conf
```

Inside the file, I added:

```
net.ipv4.ip_forward=1
```

Then applied the change with:

```bash
sudo sysctl -p
```

This allowed the kernel to forward packets between interfaces — a key step in transforming the Pi into a functioning router.

---

## 🌐 Configuring dnsmasq for DHCP and DNS
To provide IP addresses and DNS resolution to IoT devices, I installed **dnsmasq**, a lightweight DHCP/DNS service.

Configuration file:
```
/etc/dnsmasq.conf
```

Contents:
```
interface=wlan0
bind-interfaces
dhcp-range=10.0.10.10,10.0.10.100,255.255.255.0,24h
dhcp-option=3,10.0.10.1
dhcp-option=6,8.8.8.8,1.1.1.1
```

This setup ensured any IoT device connecting to the Pi’s Wi-Fi would receive an IP in the **10.0.10.0/24** range, with the Pi (`10.0.10.1`) serving as both gateway and DNS forwarder.

---

## 📡 Setting up hostapd (Wi-Fi Access Point)
I installed and configured **hostapd** to turn the Pi’s Wi-Fi radio (`wlan0`) into an access point.

Configuration file:
```
/etc/hostapd/hostapd.conf
```

Example content:
```
interface=wlan0
driver=nl80211
ssid=IoT_Net
hw_mode=g
channel=6
wpa=2
wpa_passphrase=StrongIoTpass123
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```

This created a 2.4 GHz Wi-Fi network (`IoT_Net`) for IoT devices.  
DHCP was disabled in hostapd so dnsmasq would handle IP distribution.

On first test, the SSID appeared intermittently, but clients received self-assigned (`169.254.x.x`) addresses instead of the intended `10.0.10.x`.  
This showed that **dnsmasq** wasn’t responding to DHCP requests — likely a timing or interface ownership issue — leading to several debugging rounds involving NetworkManager, startup dependencies, and service restarts.

---

## ⚙️ Challenges & Lessons Learned
This project revealed that **Linux networking is highly modular** and driven by many daemons working together.  
I learned that each interface can have its own configuration file and that multiple services — like **NetworkManager**, **systemd-networkd**, and **wpa_supplicant** — can compete for control if not explicitly separated.

Other key lessons:
- **Sequencing matters:** services like dnsmasq must start *after* their interfaces are fully initialized.  
- **Restarting services after config edits** is essential for changes to apply.  
- **Verifying interface ownership** (who manages eth0/wlan0) quickly narrows down networking issues.  
- Even when a project doesn’t fully succeed, each stage deepens understanding of how Linux routes, assigns IPs, and exposes configuration layers.

---

### 🧩 Reflection
Although I didn’t fully complete the network segmentation setup before the Pi lost connectivity, I walked away with a working knowledge of:
- how Linux configures and links interfaces,  
- how to read and edit service configuration files,  
- and how routing, DHCP, and access-point management interact.  

I now feel more confident recognizing configuration files, tracing service control, and rebuilding this setup from scratch in a more structured way.

---

*End of Lab Log*
