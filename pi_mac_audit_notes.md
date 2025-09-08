# 🔒 Session Notes — Raspberry Pi Rig, SSH Keys, and Mac Audit
**Date:** 2025-09-08

---

## 🖥️ Raspberry Pi Pentesting Rig

### Flashing & Configuration
- Chose **Kali Linux ARM image** for the Pi to run pentesting tools.
- Boot configuration required:
  - Creating a `wpa_supplicant.conf` in the boot partition with Wi-Fi details.
  - Option to add multiple networks (`home` + `hotspot`) with `priority` values.
- Debugging:
  - Pi failed to connect when `wpa_supplicant.conf` was renamed to `.bak`, meaning the config was invalid.
  - Fixed by cleaning up SSID/password and ensuring the file lived in `/boot` on first boot.

### Networking
- Identified Pi IP using network scanning tools.
- Confirmed phone hotspot was assigning addresses in a private range.
- Reserved the Pi’s IP on the gateway to make it static for easy SSH.

---

## 🔑 SSH Key Management

### Concepts
- **Password authentication**: relies on typing a password; vulnerable to brute force.
- **Key authentication**: uses a public/private key pair; far more secure.
- Keys can be protected with a passphrase, and optionally cached with an SSH agent.

### Termux → Mac
- Generated an Ed25519 keypair in Termux.
- Copied public key into `~/.ssh/authorized_keys` on the Mac.
- Tested and confirmed passwordless login from phone to Mac.

### Mac → Pi
- Generated a keypair on the Mac.
- Used `ssh-copy-id` to copy key to the Pi.
- Verified login without needing the default Pi user password.

### Permissions
- `chmod 700 ~/.ssh` → owner has full access to `.ssh` directory.
- `chmod 600 ~/.ssh/authorized_keys` → owner can read/write, no one else has access.
- Critical because SSH refuses to use keys if permissions are too loose.

### Hardening
- Disabled password logins after verifying key logins work:
  - On Pi: edited `/etc/ssh/sshd_config` → `PasswordAuthentication no`.
  - On Mac: edited `/etc/ssh/sshd_config` → `PasswordAuthentication no`.
- Restarted SSH services to apply changes.
- Verified connections still worked using keys.

---

## 🛡️ macOS Security Audit

### SSH Access Review
- Used `last` to check login history:
  - Entries showed personal hotspot logins.
  - No external or unknown IPs present.
- Confirmed no successful SSH logins beyond personal sessions.

### Remote Agent Review
- Queried unified logs for `screensharingd`, `ARDAgent`, and `appleVNCServer`:
  - No remote GUI logins detected since software removal date.
- Checked `/Library/LaunchDaemons` for leftover management agents.
- Confirmed no suspicious entries.

### MDM Enrollment
- Verified with:
  ```bash
  sudo profiles status -type enrollment
  ```
  - Result: **Not enrolled**.

### Port Hardening
- With SSH keys in place and password auth disabled, brute-force attempts are blocked.
- Firewall settings checked to ensure Remote Login (SSH) is allowed only where intended.
- No evidence of unauthorized access after management removal.

---

## ✅ Key Takeaways
- Pi rig is now correctly configured with a static IP and secured by SSH keys.
- Both Mac and Pi reject password authentication, relying solely on cryptographic keys.
- macOS audit shows no post-employment remote access; logs confirm only self-initiated SSH sessions.
- System hardened: strong key-based authentication, unnecessary remote management services absent, and enrollment removed.

---
