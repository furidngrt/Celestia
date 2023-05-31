<p align="center"><img height="150" height="auto" src="https://user-images.githubusercontent.com/63885192/228786938-4112e207-17e4-43c4-bec6-ec93e137df79.jpg"></p>

# Celestia Light Node

Light nodes ensure data availability. This is the most common way to interact with the Celestia network.

## Hardware requirements


· Memory: 2 GB RAM

· CPU: Single core

· Disk: 5 GB SSD storage

### Update & install the necessary dependencies

```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y

```

### Install GO 1.19.1

```
ver="1.19.1"

cd $HOME

wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"

sudo rm -rf /usr/local/go

sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"

rm "go$ver.linux-amd64.tar.gz"
```

### Add the /usr/local/go/bin directory to the $PATH:

```
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Installing the Celestia node

```
cd $HOME
rm -rf celestia-node
git clone https://github.com/celestiaorg/celestia-node.git
cd celestia-node/
git checkout tags/v0.6.0
make install
make cel-key
```

### Init Light Node

```
celestia light init --p2p.network blockspacerace
```
### Save Pharse 

```
./cel-key list --node.type light --p2p.network blockspacerace
```

### Create Services

```
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-lightd.service
[Unit]
Description=celestia-lightd Light Node
After=network-online.target

[Service]
User=$USER
ExecStart=/usr/local/bin/celestia light start --core.ip https://rpc-blockspacerace.pops.one --core.rpc.port 26657 --core.grpc.port 9090 --keyring.accname my_celes_key --metrics.tls=false --metrics --metrics.endpoint otel.celestia.tools:4318 --gateway --gateway.addr localhost --gateway.port 26659 --p2p.network blockspacerace
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

### Start Services Celestia

```
systemctl enable celestia-lightd
systemctl start celestia-lightd
```

### Chceck Logs

```
journalctl -u celestia-lightd.service -f
```

### Retrieve Node ID celestia

```
AUTH_TOKEN=$(celestia light auth admin --p2p.network blockspacerace)

curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```

#### your ID 12D3xxxxx Save


#### Explorer : https://tiascan.com/light-nodes

#### If you get a message header: not found, please try these steps:

1. stop your light node
2. go to ``.celestia-light-blockspacerace-0 directory``
3. delete /data folder, make sure you DO NOT delete the keys folder
4. init your light node again
5. start the light node

### Update

#### stop your node first
```
cd celestia-node
```
```
git fetch
```
```
git checkout v0.8.1
```
```
make build
```
```
sudo make install
```
 
### for light nodes: 
```
cd $HOME
```
```
cd .celestia-light-blockspacerace-0
```
```
sudo rm -rf blocks index data transients
```

### init your light node:
```
celestia light init --p2p.network blockspacerace
```
start your node again
```
systemctl start celestia-lightd
```

### Update

Instructions to upgrade for light nodes:

1. stop your node
2. Upgrade go:

```
if ! [ -x "$(command -v go)" ]; then
  ver="1.20.3"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bash_profile
  source ~/.bash_profile
fi
```

3. `cd celestia-node`
4. `git fetch`
5. `git checkout v0.9.1`
6. `make build`
7. `sudo make install`
8. `celestia light config-update --p2p.network blockspacerace`
9. start your node again

### Delete Node

```
systemctl stop celestia-lightd
rm -rf /etc/systemd/system/celestia-lightd.service
rm -rf celestia-node
rm -rf .celestia-app
rm -rf .celestia-light-blockspacerace-0
rm -f $(which celestia-lightd)
rm -rf $HOME/.celestia-lightd
rm -rf $HOME/celestia-lightd
```
