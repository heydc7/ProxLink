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
