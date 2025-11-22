
# Digital Ocean Setup & WireGuard Installation
---

# Digital Ocean

## Create a Digital Ocean Account
 Create a new [Digital Ocean](https://www.digitalocean.com/?refcode=d33d59113ab6&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=CopyPaste)
 account and claim a **$200 credit** for two months.  
   *(Payment method is required to claim credits once you create the account.)*
   *Remember to delete your droplet and data before Digital Ocean pocket checks you*  



## Create an Ubuntu 24.04 Droplet

Go to **Droplet > Create Droplets** 

## Droplet configuration
Region: Atlanta (Closest Location)

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
sudo apt update && sudo apt upgrade -y
```

---

## Docker Installation

Refer to the [Official Docker Installation Wiki](https://docs.docker.com/engine/install/ubuntu/) or [Docker section of my Docker-Gotify Instillation Guide](https://jnwh8686.github.io/Arch-Linux-Install/docker-gotify)

### Remove Potential Conflicting Packages

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
    sudo apt-get remove $pkg
done
```

### Set Up Docker's `apt` Repository

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl -y

sudo install -m 0755 -d /etc/apt/keyrings

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
# Test Docker installation functionality
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

Open: `http://<Droplet_Public_IP>:51821`
    
Complete initial setup:
    
    - Set **Admin Username** & **Password**
        
    - Host: Droplet Public IP
        
    - Port: 51820
        
Create a new client for VPN access.
    

## Testing with [IpLeak.net](https://ipleak.net/)

### Mobile (iOS)

* Download **WireGuard** from the App Store.
    
* Show the QR code in Web UI and scan it with your phone camera.
    
* Connect/Create VPN in device settings.
    

[IOS WireGuard Demo (off)](https://imgur.com/ydpGoXa)

[IOS WireGuard Demo (on)](https://imgur.com/aYRFqu6)

### Laptop (Windows)

* Download the `.conf` file from the new client.
    
* Install **WireGuard for Windows**.
    
* Import the `.conf` file into the desktop application and activate the tunnel.
    

[Windows Laptop WireGuard Demo](https://i.imgur.com/i50FnhB.png)

---

### Notes

```bash
# Check container logs if there are issues and follow the link displayed. Most likely the issue is with the environment of the .yaml file
docker logs wg-easy

# Ensure firewall allows:
# UDP port 51820
# TCP port 51821
# if not set to insecure in the .yaml file, you need another Docker container to set up a password hash
# consult wiki for reverse proxy setup
```
