
# Digital Ocean Setup & WireGuard Installation

## Create a Digital Ocean Account
1. Create a new Digital Ocean account and claim a **$200 credit** for two months.  
   *(Payment method is required to claim credits.)*  

[Digital Ocean Referral Link](https://www.digitalocean.com/?refcode=d33d59113ab6&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=CopyPaste)



## Create an Ubuntu 24.04 Droplet

Go to **Droplet > Create Droplets** and select:

## Droplet configuration
Region: Atlanta
Datacenter: Atlanta Datacenter 1 (ATL1)
VPC Network: default-atl1

## Operating System
Image: Ubuntu 24.04 (LTS) x64

## CPU & Disk
CPU Options: Premium Intel
Disk: NVMe SSD
Size: 8 GB / 2 Intel CPUs / 160 GB NVMe SSD / 5 TB transfer (~$48/mo)


### Authentication Method

Choose **Password** and create a root password:

```
<password>
```


## Droplet Web Console

*  Copy your droplet's **Public IPv4** address.

*  Connect via SSH or paste the IP for terminal commands.
### Update & Upgrade Packages

```bash
# Update package lists and upgrade installed packages
sudo apt update && sudo apt upgrade -y
```

---

## Docker Installation

Refer to the [Docker Installation Wiki](https://docs.docker.com/engine/install/ubuntu/) or [Docker-Gotify Instillation Guide](https://jnwh8686.github.io/Arch-Linux-Install/docker-gotify)

### Remove Potential Conflicting Packages

```bash
# Remove older Docker, Podman, or container runtime packages
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
    sudo apt-get remove $pkg
done
```

### Set Up Docker's `apt` Repository

```bash
# Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl -y

# Prepare keyrings folder
sudo install -m 0755 -d /etc/apt/keyrings

# Add Docker's official GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Add Docker Repository

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

### Update Apt Again

```bash
sudo apt update
```

### Install Docker Packages

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### Verify Docker Installation

```bash
# Test Docker installation
sudo docker run hello-world
```

---

# WireGuard Installation

### Create WireGuard Folder

```bash
mkdir -p /opt/wg-easy
cd /opt/wg-easy
```

### Create `docker-compose.yaml`

```bash
nano docker-compose.yaml
```

Paste the following configuration:

```yaml
volumes:
  etc_wireguard:

services:
  wg-easy:
    environment:
      - HOST=0.0.0.0
      - INSECURE=true
      # Optional:
      # - PORT=51821

    image: ghcr.io/wg-easy/wg-easy:15
    container_name: wg-easy

    networks:
      wg:
        ipv4_address: 10.42.42.42
        ipv6_address: fdcc:ad94:bacf:61a3::2a

    volumes:
      - etc_wireguard:/etc/wireguard
      - /lib/modules:/lib/modules:ro

    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"

    restart: unless-stopped

    cap_add:
      - NET_ADMIN
      - SYS_MODULE

    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=0
      - net.ipv6.conf.all.forwarding=1
      - net.ipv6.conf.default.forwarding=1

networks:
  wg:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config:
        - subnet: 10.42.42.0/24
        - subnet: fdcc:ad94:bacf:61a3::/64
```

**Write out and exit** (`Ctrl+O` > `Enter` > `Ctrl+X`).

### Start WireGuard in Detached Mode

```bash
docker compose up -d
```

### Confirm Running Containers

```bash
docker ps
```

---

## Access WireGuard Web UI

1. Open: `http://<Droplet_Public_IP>:51821`
    
2. Complete initial setup:
    
    - Set **Admin Username** & **Password**
        
    - Host: Droplet Public IP
        
    - Port: 51820
        
3. Create a new client for VPN access.
    

## Testing

### Mobile (iOS)

1. Download **WireGuard** from the App Store.
    
2. Show the QR code in Web UI and scan it.
    
3. Connect/Create VPN in device settings.
    

_Add screenshots here._

### Laptop (Windows)

1. Download the `.conf` file from the new client.
    
2. Install **WireGuard for Windows**.
    
3. Import the `.conf` file and activate the tunnel.
    

_Add screenshots here._

### Notes & Tips

```bash
# Check container logs if there are issues
docker logs wg-easy

# Ensure firewall allows:
# UDP port 51820
# TCP port 51821
# if not set to insecure you need another docker container to set up password hash
# consult wiki for reverse process setup
```
