# Digital Frame Network Triage — 2025-09-18

**Summary**

This repository document records the initial triage and capture workflow used to monitor a digital picture frame on a home LAN using a Raspberry Pi (Kali) hard-wired to the gateway. It contains the goals, assumptions, step-by-step procedures (how + why), benign findings from the initial window, and suggested next steps. Sensitive identifiers (IP addresses, MACs, device hostnames, and raw capture contents) are intentionally redacted or referenced as placeholders to protect privacy.

---

## Goals
- Passively observe the network traffic to/from the digital picture frame to discover what services and endpoints it contacts.
- Identify any suspicious outbound activity (unexpected cloud hosts, plaintext credential leaks, or unusual beaconing).
- Maintain evidence hygiene and preserve captures for reproducible analysis.

## Scope & Assumptions
- Monitoring was done with a Raspberry Pi (Kali) **wired** to the gateway/switch. Wi‑Fi monitoring was not performed in this phase.
- The operator has administrative control over the network and devices being tested.
- All sensitive data (IP addresses, MAC addresses, pcap files) are redacted in this public writeup. Replace placeholders with local values when running commands.

## Quick references (placeholders)
- FRAME_IP: replace with the picture frame's IPv4 address in your network.
- FRAME_MAC: replace with the frame's MAC address (OUI).
- CAPTURE_DIR: local directory on the Pi where pcaps are stored (e.g., `/var/log/pcaps` or `/var/logs/pcaps`).

---

## Step-by-step procedure (each is one focused action)

### Step 1 — Re-confirm target identity (IP → MAC → vendor)
**How**
- `ping -c 3 FRAME_IP`
- `arp -n | grep FRAME_IP` or `ip neigh show | grep FRAME_IP`
- `sudo arp-scan --interface=eth0 --localnet` (requires root)

**Why**
- Confirms the target is online and gives the MAC/OUI for accurate L2 filtering. Prevents capturing unrelated hosts.

**Notes**
- If the vendor is unknown locally, look up the OUI externally. Keep OUI info private.

### Step 2 — Choose capture method (SPAN/TAP preferred)
**How**
- Prefer configuring port mirroring (SPAN) on the managed switch/gateway to mirror the frame’s port to the Pi.
- If SPAN/TAP unavailable and you control the device, use a hardware TAP. ARP spoofing is an active last-resort method and is discouraged in production networks.

**Why**
- Passive mirroring avoids altering or disrupting the device’s traffic.

### Step 3 — Prepare the capture host
**How**
- Ensure tools installed: `sudo apt update && sudo apt install -y tcpdump tshark tcpflow zeek`.
- Create capture directory and set ownership: `sudo mkdir -p CAPTURE_DIR && sudo chown $(whoami) CAPTURE_DIR`.
- Confirm interface name and enable promiscuous mode: `ip -br link show` and `sudo ip link set <IF> promisc on`.
- Ensure NTP/time sync: `timedatectl set-ntp true`.

**Why**
- Accurate timestamps and reliable tools are essential for forensic analysis and rotations.

### Step 4 — Quick live probe (short, safe)
**How**
- Run: `sudo timeout 30 tcpdump -i <IF> -n -s 0 -c 200 -vv "host FRAME_IP and not port 22"`.

**Why**
- A short, bounded probe verifies live behavior and avoids huge captures.

**Troubleshooting**
- If `0 packets captured`, try L2 capture by MAC (see next step) or check whether the device is on Wi‑Fi/different VLAN.

### Step 5 — L2 capture by MAC (if IP capture shows nothing)
**How**
- Run: `sudo timeout 20 tcpdump -i <IF> -n -e -s 0 -c 200 'ether host FRAME_MAC' -vv`.

**Why**
- Catches ARP, mDNS/LLMNR, and any non-IP or proxied traffic the IP filter may miss.

### Step 6 — Broader capture including mDNS/IGMP
**How**
- If device emits mostly multicast discovery traffic, capture with: `sudo timeout 60 tcpdump -i <IF> -n -U -w CAPTURE_DIR/frame_capture_now.pcap "host FRAME_IP or udp port 5353 or igmp"`.

**Why**
- Many consumer devices use mDNS/SSDP/IGMP for local discovery and may not show external traffic in short windows.

### Step 7 — Post-capture analysis (tshark & Wireshark)
**How**
- Extract interesting fields:
  - DNS: `tshark -r pcap -Y "dns && !(udp.port==5353)" -T fields -e dns.qry.name`
  - TLS SNI: `tshark -r pcap -Y "tls.handshake.extensions_server_name" -T fields -e tls.handshake.extensions_server_name`
  - HTTP Host: `tshark -r pcap -Y "http.host" -T fields -e http.host`
- Transfer pcap to a workstation and open in Wireshark for GUI analysis.

**Why**
- Reveals external hosts/CDNs and plaintext headers. SNI provides hostnames for TLS connections when available.

### Step 8 — Evidence hygiene
**How**
- Hash pcaps: `sha256sum pcap > pcap.sha256`.
- Keep metadata: capture time, capture method (SPAN/TAP/ARPspoof), filter string, operator identity.

**Why**
- Preserves chain-of-custody and ensures integrity for later review.

---

## What we observed (summary — redacted)
- Short capture windows showed **no observable unicast DNS, TLS SNI, or plaintext HTTP host headers** for the frame in the sampled period. Most multicast (mDNS/IGMP) traffic in the initial capture originated from a different device (a media device) on the LAN.
- No signs of plaintext credential exfiltration or unexpected beaconing were seen in the sampled window.

**Caveat**: Observations are time-limited. Schedule longer or event-driven captures (e.g., while changing photos or during reboot) for higher confidence.

---

## Artifacts (redacted)
- PCAPs: `CAPTURE_DIR/frame_capture_now.pcap` (test), `CAPTURE_DIR/frame_full_<timestamp>.pcap` (full capture) — store safely and hash.
- Logs: structured Zeek/Suricata logs (optional) if enabled.

---

## Next steps (pick one)
- Run a 5–15 minute passive capture while the frame is active.
- Configure SPAN/TAP on your gateway/switch to mirror the frame’s port to the Pi and capture full traffic passively.
- Capture on the Wi‑Fi radio (monitor mode + WPA key) if the frame is a Wi‑Fi client and SPAN is not possible.
- Deploy Zeek or Suricata on the Pi for continuous monitoring and alerts.

---

## Safety & privacy notes
- Only capture traffic you are authorized to inspect. Redact or remove any unrelated personal traffic before sharing pcaps publicly.
- Avoid publishing pcaps or logs with embedded PII (usernames, email addresses, device identifiers) without explicit consent.

---

## How to resume later
- Use the `CAPTURE_DIR` path and repeat Step 6 for a longer capture. Use the same single-step approach: run one command, review output, and proceed.

---

*Document created by cybersecurity mentoring session.*

