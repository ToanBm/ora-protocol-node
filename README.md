# Ora Protocol Node Guide

## System Requirements

| Component        | Minimum Requirement        | Recommended Requirement  |
|------------------|----------------------------|--------------------------|
| **Operating System** | Any OS with Docker installed | Any OS with Docker installed |
| **CPU**           | 1-core CPU                 | 1-core CPU               |
| **RAM**           | 8 GB RAM                   | 12 GB RAM                |

## Prerequisites

- Create a new EVM address
- Request sepolia faucet to this address
- Visit [alechemy site](https://dashboard.alchemy.com/apps) and copy the following endpoints 
1. Ethereum mainnet https endpoint 
2. Ethereum mainnet wss endpoint
3. Sepolia Ethereum https endpoint
4. Sepolia Ethereum wss endpoint

## Installation
### Install Packages
```console
sudo apt update & sudo apt upgrade -y
sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y
```
### Install Docker
```console
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
docker version
```
### Install Docker-Compose
```console
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)

curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
docker-compose --version
```
### Tora Installation
```console
git clone https://github.com/ora-io/tora-docker-compose && cd tora-docker-compose
```
```console
cp .env.example .env && nano .env
```
Edit your info:
```console
PRIV_KEY=""

MAINNET_WSS=""
MAINNET_HTTP=""
SEPOLIA_WSS=""
SEPOLIA_HTTP=""
```
* Ctrl X, Y, Enter to save and exit.
### Start Node
```console
sysctl vm.overcommit_memory=1
```
```console
docker compose up -d
```
## Node Status

- You can check logs using this command
```console
docker logs ora-tora -f -n 100
```
- If you see something like this, it means, it is working fine

![image](https://github.com/user-attachments/assets/d0f46d5f-159d-40a4-8cf7-21111f899a6f)

- You can also check via [Sepolia Explorer](https://sepolia.etherscan.io), If you see your wallet address is interacting with this contract `0x0A0f4321214BB6C7811dD8a71cF587bdaF03f0A0` , it means it is good

![image](https://github.com/user-attachments/assets/c256b783-8786-4123-8931-a85051e646db)



