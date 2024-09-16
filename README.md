# Ora Protocol Node Guide

## System Requirements

| Component        | Minimum Requirement        | Recommended Requirement  |
|------------------|----------------------------|--------------------------|
| **Operating System** | Any OS with Docker installed | Any OS with Docker installed |
| **CPU**           | 1-core CPU                 | 1-core CPU               |
| **RAM**           | 8 GB RAM                   | 12 GB RAM                |

## Step 1: Dependecies
### 1- Install Packages
```console
sudo apt update & sudo apt upgrade -y
sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y
```
### 2- Install Docker
```console
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
docker version
```
### 3- Install Docker-Compose
```console
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)

curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
docker-compose --version
```
## Step 2: Obtain Alchemy API
- Create a new EVM address
- Request sepolia faucet to this address
- Visit [alechemy site](https://dashboard.alchemy.com/apps) and copy the following endpoints 
1. Ethereum mainnet https endpoint 
2. Ethereum mainnet wss endpoint
3. Sepolia Ethereum https endpoint
4. Sepolia Ethereum wss endpoint

## Step 3: Install Tora
### 1- Create Tora directory
```console
mkdir tora
cd tora
```

### 2- Create .env file
```console
nano .env
```

**Paste the following code in .env**
* `PRIV_KEY`: Replace a Metamask wallet private key
* Make sure you have enough ETH on Sepolia & Mainnet to pay very little transactions gas
* `MAINNET_WSS`, `MAINNET_HTTP`,`SEPOLIA_WSS`, `SEPOLIA_HTTP`: Paste your `Alchemy` URLs
* `CONFIRM_CHAINS`: I have chosen `'["sepolia","mainnet"]'` , if you don't want mainnet transactions you can set it to: `'["sepolia"]'`
```
############### Sensitive config ###############

# private key for sending out app-specific transactions
PRIV_KEY=""

############### General config ###############

# general - execution environment
TORA_ENV=production

# general - provider url
MAINNET_WSS=""
MAINNET_HTTP=""
SEPOLIA_WSS=""
SEPOLIA_HTTP=""

# redis global ttl, comment out -> no ttl limit
REDIS_TTL=86400000 # 1 day in ms 

############### App specific config ###############

# confirm - general
CONFIRM_CHAINS='["sepolia","mainnet"]'
CONFIRM_MODELS='[13]' # 13: OpenLM ,now only 13 supported
# confirm - crosscheck
CONFIRM_USE_CROSSCHECK=true
CONFIRM_CC_POLLING_INTERVAL=3000 # 3 sec in ms
CONFIRM_CC_BATCH_BLOCKS_COUNT=300 # default 300 means blocks in 1 hours on eth
# confirm - store ttl
CONFIRM_TASK_TTL=2592000000
CONFIRM_TASK_DONE_TTL = 2592000000 # comment out -> no ttl limit
CONFIRM_CC_TTL=2592000000 # 1 month in ms
```
* `CTRL+X+Y+ENTER` to save and exit

### 3- Create docker-compose file
```console
nano docker-compose.yml
```

**Paste following code**
```
# ora node docker-compose
version: '3'
services:
  confirm:
    image: oraprotocol/tora:confirm
    container_name: ora-tora
    depends_on:
      - redis
      - openlm
    command: 
      - "--confirm"
    env_file:
      - .env
    environment:
      REDIS_HOST: 'redis'
      REDIS_PORT: 6379
      CONFIRM_MODEL_SERVER_13: 'http://openlm:5000/'
    networks:
      - private_network
  redis:
    image: oraprotocol/redis:latest
    container_name: ora-redis
    restart: always
    networks:
      - private_network
  openlm:
    image: oraprotocol/openlm:latest
    container_name: ora-openlm
    restart: always
    networks:
      - private_network
  diun:
    image: crazymax/diun:latest
    container_name: diun
    command: serve
    volumes:
      - "./data:/data"
      - "/var/run/docker.sock:/var/run/docker.sock"
    environment:
      - "TZ=Asia/Shanghai"
      - "LOG_LEVEL=info"
      - "LOG_JSON=false"
      - "DIUN_WATCH_WORKERS=5"
      - "DIUN_WATCH_JITTER=30"
      - "DIUN_WATCH_SCHEDULE=0 0 * * *"
      - "DIUN_PROVIDERS_DOCKER=true"
      - "DIUN_PROVIDERS_DOCKER_WATCHBYDEFAULT=true"
    restart: always

networks:
  private_network:
    driver: bridge
```
* `CTRL+X+Y+ENTER` to save and exit

## Step 4: Run Tora Node
```console
docker compose up -d
```

## Step 5: Check logs
```console
cd $HOME && cd tora
docker compose logs -f
```

* To test that the node is running correctly, request inference from OpenLM on **Sepolia** here: https://www.ora.io/app/opml/openlm

* Shortly after the request is initiated, the Tora validator will log receive event in tx .... This indicates that the Tora validator has received the request and started to perform inference.
![image](https://github.com/user-attachments/assets/bd308f81-7eb8-443a-a554-acfb58acbe12)

* Now you know, whoever use Ora OLM on `Sepolia` & `Mainnet`, You will verify a transaction and get these logs, and will receive points as rewards
## --------------- The End! -----------------------------

