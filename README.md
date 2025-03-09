# ProxLink
ProxLink is a remote server management solution for Proxmox, enabling seamless access and control over your Proxmox VE instances via secure networking. With ProxLink, you can manage your virtual environments from anywhere, leveraging tools like Tailscale and intuitive networking configurations for hassle-free remote connectivity.


## Diagram

```
                          INTERNET
                             |
                     +-------+-------+
                     |  Router/Gateway|
                     |   192.168.1.1  |
                     +-------+-------+
                             |
                        Wi-Fi Network
                             |
                 +-----------+-----------+
                 | Proxmox Host          |
                 | Wi-Fi Interface (wlp2s0) 
                 | 192.168.1.100/24      |
                 +-----------+-----------+
                             |
              +--------------+--------------+
              | Virtual Bridge (vmbr0)      |
              | 192.168.2.1/24              |
              +--------------+--------------+
                             |
               +------+     +------+     +------+
               | VM 1 |     | VM 2 |     | VM 3 |
               |192...|     |192...|     |192...|
               +------+     +------+     +------+
                             |
              +--------------+--------------+
              | Virtual Network (vnet1)     |
              | 192.168.3.1/24              |
              +--------------+--------------+
                             |
             NAT (via iptables) on Proxmox Host
             Source: 192.168.3.0/24
             Outgoing: wlp2s0
```

# ProxLink - Proxmox Remote Server

ProxLink is a remote server setup for **Proxmox VE**, optimized for secure and efficient virtualization. This guide provides details on the hardware configuration and network setup to ensure seamless remote access and performance.

---

## 🖥️ Hardware Configuration

#### **1️⃣ Beelink SER5 Pro Mini PC**
- **Processor**: AMD Ryzen 7 5800H (8C/16T, Up to 4.4GHz)
- **RAM**: 32GB DDR4
- **Storage**: 1TB NVMe SSD
- **Graphics**: Integrated AMD Radeon (Supports 4K@60Hz Output)
- **Networking**:
  - **WiFi 6** (Primary connection)
  - **Bluetooth 5.2**
  - **Ethernet via WiFi Extender**
- **Operating System**: Proxmox VE 8.x (Replaces Windows 11 Pro)

#### **2️⃣ WiFi Extender (Ethernet Connection)**
- Extends network coverage for Proxmox.
- Provides an **Ethernet bridge** for better stability.

#### **3️⃣ Cooling System (Optional)**
- If available, an external cooling fan or additional thermal management can be used to maintain performance under high loads.

## ⚙️ Software & Networking Setup

### **1️⃣ Install Proxmox VE**
1. Download **Proxmox VE** ISO: [Proxmox Official Site](https://www.proxmox.com/en/downloads)
2. Create a bootable USB using **Rufus** or **Balena Etcher**.
3. Boot from USB and install **Proxmox VE**.

### **2️⃣ Configure Network**
Edit the network configuration file:
```bash
nano /etc/network/interfaces
```

## Configuration
### Step 1: Configure Wi-Fi on Proxmox
Modify the Proxmox network settings to use **Wi-Fi only**.

```bash
nano /etc/network/interfaces
```

Add the following configuration:

```
auto lo
iface lo inet loopback

# Wi-Fi Interface Configuration
auto wlp2s0
iface wlp2s0 inet static
    address 192.168.1.100/24
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
    wpa-ssid "Your_SSID"
    wpa-psk "Your_PASSWORD"

# Virtual Bridge for Internal Networking
auto vmbr0
iface vmbr0 inet static
    address 192.168.2.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

# Virtual Network with NAT for VMs
auto vnet1
iface vnet1 inet static
    address 192.168.3.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    post-up echo 1 > /proc/sys/net/ipv4/ip_forward
    post-up iptables -t nat -A POSTROUTING -s '192.168.3.0/24' -o wlp2s0 -j MASQUERADE
    post-down iptables -t nat -D POSTROUTING -s '192.168.3.0/24' -o wlp2s0 -j MASQUERADE
```

### 2. **Restart the networking:**

```bash
systemctl restart networking
ip addr show wlp2s0
ip route
```


