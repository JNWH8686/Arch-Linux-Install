# Docker + Gotify Installation Guide (Ubuntu)

Docker installation wiki: https://docs.docker.com/engine/install/ubuntu/

### Uninstall Any Potential Conflicting Packages

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

### Update Apt

```bash
sudo apt update 
```

### Set up Docker's `apt` Repository

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Add the Repository to Apt Sources

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Update Apt again:

```bash
sudo apt update
```

### Install Docker Packages

Install the latest version:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verify the installation is running:
```bash
sudo docker run hello-world
```

# Gotify Installation
 [Official instillation page](https://gotify.net/docs/install)  
**Gotify** is a notification message service.

### Create Application Directory

```bash
mkdir -p ~/gotify
cd ~/gotify
```

### Create `docker-compose.yml`

Use a text editor to create the file with the following content (**change the default password**):

```yaml
services:
  gotify:
    image: gotify/server:latest
    container_name: <container_name>
    ports:
      - "8080:80"
    environment:
      GOTIFY_DEFAULTUSER_PASS: "<password>"  # replace <password>
      TZ: "America/Chicago"
    volumes:
      - "./gotify_data:/app/data"
    restart: unless-stopped
```


Verify the .yml file:

```bash
ls docker-compose.yml
```

You should see:

```
docker-compose.yml
```

---

### Start Gotify and have it run in the background in detached mode

```bash
sudo docker compose up -d
```

Verify that it is running:

```bash
sudo docker ps
```


### Gotify GUI

Open in a browser:

```
http://localhost:8080/
```

Default login credentials:

```
Username: admin
Password: <password>
```
You should see an admin account, along with having the ability to create additional accounts with the GUI

[Example](https://imgur.com/gQtCRtE)

### Sending a Message

* Create an application
* Copy the application token

Send a message:
```bash
curl "http://localhost:8080/message?token=<token>" \
     -F "title=[title]" \
     -F "message=[message]"
```

[Example result](https://imgur.com/6u0177J)
