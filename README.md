# Hardened Network Sandbox Environment

A secure, isolated virtual machine environment running Ubuntu Server on VirtualBox, hardened using isolated network adapters, Uncomplicated Firewall (`ufw`), and SSH key-pair authentication.

---

## Network Architecture
- **Hypervisor:** Oracle VM VirtualBox
- **Operating System:** Ubuntu Server
- **Adapter 1 (NAT):** Internet access for system updates
- **Adapter 2 (Host-Only):** `192.168.XX.XXX` (`enp0s8`) — Private management network

---

## Configuration Steps

### 1. Firewall Rules (`ufw`)
Blocked all inbound traffic by default and allowed only inbound SSH on port 22:

```bash
# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH traffic
sudo ufw allow ssh

# Enable firewall
sudo ufw enable

# Verify firewall configuration
sudo ufw status verbose 
```

### 2. SSH Key-Pair Setup
Generated an Ed25519 SSH key pair on the Windows host and transferred the public key to the VM:

```powershell
# Generate key pair (Windows Host)
ssh-keygen -t ed25519

# Append public key to VM authorized_keys
Get-Content ~/.ssh/id_ed25519.pub | ssh mbajwa@192.168.XX.XXX "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### 3. Disabling Password Authentication
Hardened OpenSSH server configuration (/etc/ssh/sshd_config) to require key-based login:

```plaintext
PasswordAuthentication no
```

Applied the configuration:
```bash
sudo systemctl restart ssh
```

## Verification

* **Key Access:** `ssh mbajwa@192.168.XX.XXX` logs in seamlessly using key authentication.
* **Password Access:** `ssh -o PubkeyAuthentication=no mbajwa@192.168.XX.XXX` returns `Permission denied (publickey)`.
* **Port Exposure:** `sudo ufw status verbose` confirms port `22/tcp` is the only open port.
