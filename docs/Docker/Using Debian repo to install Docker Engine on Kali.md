## Using Repo to install Docker Engine on Kali

### [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)

Debian Repo 

[DebianReleases - Debian Wiki](https://wiki.debian.org/DebianReleases)

[DebianRepository - Debian Wiki](https://wiki.debian.org/DebianRepository)

[DebianRepository/Unofficial - Debian Wiki](https://wiki.debian.org/DebianRepository/Unofficial)

[Debian - Debian 全球镜像站](https://www.debian.org/mirror/list)

#### [OS requirements](https://docs.docker.com/engine/install/debian/#os-requirements)

To install Docker Engine, you need the 64-bit version of one of these Debian versions:

- Debian Bookworm 12 (stable)
- Debian Bullseye 11 (oldstable)

#### [Install using the apt repository](https://docs.docker.com/engine/install/debian/#install-using-the-repository)

Set up Docker's `apt` repository

```sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install the Docker packages

To install the latest version, run:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

To install a specific version of Docker Engine, start by listing the available versions in the repository:

```bash
# List the available versions:
apt-cache madison docker-ce | awk '{ print $3 }'

5:27.5.0-1~debian.12~bookworm
5:27.4.1-1~debian.12~bookworm
...
```

Select the desired version and install:

```bash
VERSION_STRING=5:27.5.0-1~debian.12~bookworm
sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

### Install Docker Engine on Kali

Change to Debian Bookworm 12 (stable)

```sh
#!/bin/bash

# Add Docker's official GPG key:
apt-get update
apt-get install -y ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  bookworm stable" | \
tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update

# Install docker
apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Changed docker-compose
echo "Use docker compose up -d"
echo "Installation Complete"
```

Without GPG public key

```sh
#!/bin/bash

echo -e "[+] Updating System"
apt-get update
apt-get install -y ca-certificates curl

# Add the Docker repository to Apt sources
echo -e "[+] Add the Docker repository to Apt sources"
echo "deb [arch=$(dpkg --print-architecture) trusted=yes] https://download.docker.com/linux/debian bookworm stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update

# Install Docker and dependencies
echo -e "[+] Install Docker and dependencies"
apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo -e "[+] Installation Complete"
```

### [Uninstall Docker Engine](https://docs.docker.com/engine/install/debian/#uninstall-docker-engine)

1. Uninstall the Docker Engine, CLI, containerd, and Docker Compose packages:

   ```bash
   sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
   ```

2. Images, containers, volumes, or custom configuration files on your host aren't automatically removed. To delete all images, containers, and volumes:

   ```bash
   sudo rm -rf /var/lib/docker
   sudo rm -rf /var/lib/containerd
   ```

3. Remove source list and keyrings

   ```bash
   sudo rm /etc/apt/sources.list.d/docker.list
   sudo rm /etc/apt/keyrings/docker.asc
   ```

4. Auto Remove

   ```bash
   #!/bin/bash
   
   apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras -y
   rm -rf /var/lib/docker
   rm -rf /var/lib/containerd
   rm -rf /etc/apt/sources.list.d/docker.list
   rm -rf /etc/apt/keyrings/docker.asc
   ```

### Use a proxy server with the Docker CLI

#### [Configure the Docker client](https://docs.docker.com/engine/cli/proxy/#configure-the-docker-client)

You can add proxy configurations for the Docker client using a JSON configuration file, located in `~/.docker/config.json`. Builds and containers use the configuration specified in this file.

```json
{
 "proxies": {
   "default": {
     "httpProxy": "http://proxy.example.com:3128",
     "httpsProxy": "https://proxy.example.com:3129",
     "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
   }
 }
}
```

#### [Set proxy using the CLI](https://docs.docker.com/engine/cli/proxy/#set-proxy-using-the-cli)

Instead of [configuring the Docker client](https://docs.docker.com/engine/cli/proxy/#configure-the-docker-client), you can specify proxy configurations on the command-line when you invoke the `docker build` and `docker run` commands.

Proxy configuration on the command-line uses the `--build-arg` flag for builds, and the `--env` flag for when you want to run containers with a proxy.

```bash
docker build --build-arg HTTP_PROXY="http://proxy.example.com:3128" .
docker run --env HTTP_PROXY="http://proxy.example.com:3128" redis
```

### [Daemon proxy configuration](https://docs.docker.com/engine/daemon/proxy/)

#### [systemd unit file](https://docs.docker.com/engine/daemon/proxy/#systemd-unit-file)

If you're running the Docker daemon as a systemd service, you can create a systemd drop-in file that sets the variables for the `docker` service.

```sh
#!/bin/bash

# Create a systemd drop-in directory for the docker service
mkdir -p /etc/systemd/system/docker.service.d

# Create a file named /etc/systemd/system/docker.service.d/http-proxy.conf that adds the HTTP_PROXY environment variable
touch /etc/systemd/system/docker.service.d/proxy.conf

# If you are behind an HTTPS proxy server, set the HTTPS_PROXY environment variable
# Multiple environment variables can be set; to set both a non-HTTPS and a HTTPs proxy
tee /etc/systemd/system/docker.service.d/proxy.conf <<EOF
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:7897"
Environment="HTTPS_PROXY=https://proxy.example.com:7897"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
EOF

# Flush changes and restart Docker
systemctl daemon-reload
systemctl restart docker

# Verify that the configuration has been loaded and matches the changes you made, for example
systemctl show --property=Environment docker
```

### Using Ubuntu Repo

```bash
#!/bin/bash
# 卸载冲突包（可选）
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
     apt-get remove $pkg -y
done

# 添加Docker官方GPG密钥
 apt-get update
 apt-get install -y ca-certificates curl
 install -m 0755 -d /etc/apt/keyrings
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
 chmod a+r /etc/apt/keyrings/docker.asc

# 添加APT源（手动指定版本代号，例如 jammy）
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu jammy stable" | \
   tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装指定版本（手动指定版本字符串）
 apt-get update
VERSION_STRING="5:20.10.24~3-0~ubuntu-jammy"  # 这里直接写死
 apt-get install -y \
    docker-ce=$VERSION_STRING \
    docker-ce-cli=$VERSION_STRING \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin
```

