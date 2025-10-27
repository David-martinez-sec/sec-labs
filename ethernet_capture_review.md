# Ethernet Capture Review (Raspberry Pi)

## Overview
I was able to capture Ethernet activity on my Raspberry Pi using **tshark**. I used the following command to handle log rotation and avoid storage issues:

```bash
sudo tshark -i eth0 -b filesize:16384 -b files:20 -w /var/log/caps/
```

This command captures packets on the `eth0` interface, rotating files every 16MB and keeping up to 20 capture files to prevent disk space from being consumed by continuous traffic.

---

## Initial Review
After the capture, I reviewed both the head and tail of the logs to check for any unusual activity. Nothing stood out — no suspicious packets, broadcasts, or external communications were detected.

---

## Deep Dive with Wireshark
I then copied the files to my Mac for deeper analysis using **Wireshark**. Upon inspection, all visible activity appeared to be **normal local area network chatter**, including:

- Multicast and broadcast discovery (mDNS, SSDP)
- IPv6 router advertisements and neighbor solicitations
- ARP requests between local devices
- Routine SSH traffic between my Mac and the Pi

To refine the analysis, I applied several filters in Wireshark to hide background noise such as:

```wireshark
!(mdns || arp || icmpv6 || icmp || udp.port == 5353 || udp.port == 1900)
```

This helped isolate unicast traffic, but even then, nothing anomalous presented itself.

---

## Conclusion
The capture revealed only expected LAN behavior — primarily normal service discovery, ARP lookups, and IPv6 maintenance traffic. No suspicious or foreign packets were identified during the monitoring window.
