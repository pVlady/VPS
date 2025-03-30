# Установка Docker
## [Ubuntu 24.04 (noble)](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
1. Set up Docker's apt repository
```bash
# add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# add the repository to apt sources:
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc]  \
      https://download.docker.com/linux/ubuntu noble stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```
2. Install the latest Docker packages
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# test the result
sudo docker run hello-world
```

3. Create the docker group if it doesn't exists
```bash
sudo cat /etc/group    ; the list of all groups
sudo groupadd docker
```
4. Add user to the docker group
```bash
 sudo usermod -aG docker $USER
```
5. Restart the virtual machine for changes to take effect.\
You can also run the following command to activate the changes to groups:
```bash
 newgrp docker
```
