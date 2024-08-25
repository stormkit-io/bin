# Self-Hosting Stormkit on a Ubuntu Server

<!-- These steps are taken from: https://docs.docker.com/engine/install/ubuntu/ -->

## Install Docker on Ubuntu

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

# Install docker packages
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test if docker is running
sudo docker run hello-world

# Add user to the docker group
# This will allow using docker without sudo
# You will need to exit and re-ssh into your instance for these
# changes to take effect
sudo usermod -a -G docker $USER
```

## Install Stormkit

```bash
# Make sure you're in $HOME folder
cd

# Create a stormkit folder
mkdir -p stormkit

# Create an env file and follow steps in the README.md file
touch .env

# Copy the contents of `docker-compose.yaml` in this repository
touch docker-compose.yaml
```

## With Docker Swarm

```bash
# Initialize the Swarm manager node (or join if you already have a manager node)
docker swarm init

# Deploy the changes
docker compose config | sed '/published:/ s/"//g' | sed "/name:/d" | docker stack deploy -c - stormkit
```

## With Docker Compose

```bash
docker compose up -d
```
