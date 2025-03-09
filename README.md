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

## 📌 Step 1: Configure Wi-Fi on Proxmox
### 1. **Edit the Network Configuration**
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


