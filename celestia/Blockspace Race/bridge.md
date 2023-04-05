<div>
<h1 align="left" style="display: flex;"> Celestia Bridge node Setup Setup for Blockspace Race Testnet — blockspacerace-0</h1>
<img src="https://avatars.githubusercontent.com/u/54859940?s=200&v=4"  style="float: right;" width="100" height="100"></img>
</div>

Official documentation:
>- [Validator setup instructions](https://docs.celestia.org/nodes/bridge-node/)


## Hardware Requirements
 - Memory: 8 GB RAM
 - CPU: 6 cores
 - Disk: 500 GB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

## Set up a Celestia bridge node 
### Manual installation

Update packages and Install dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make gcc tar clang pkg-config libssl-dev ncdu -y 
```

install go

```bash
cd $HOME
VER="1.19.1"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm -rf  "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

Download and build binaries

```bash
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.8.2 
make build 
make install 
make cel-key 
```

Config and init app
>Please enable RPC and gRPC on your FULL node, and allow these ports in firewall rules

```bash
celestia bridge init --core.ip <FULL_NODE_IP> --core.grpc.port <FULL_NODE_GRPC_PORT> --core.rpc.port <FULL_NODE_RPC_PORT> --p2p.network blockspacerace
```

Once you start the Bridge Node, a wallet key will be generated for you. You will need to fund that address with Testnet tokens to pay for PayForBlob transactions. You can find the address by running the following command:

~~~bash
cd $HOME/celestia-node
./cel-key list --node.type bridge --keyring-backend test --p2p.network blockspacerace
~~~

Create Service file

```bash
sudo tee /etc/systemd/system/celestia-bridge.service > /dev/null <<EOF
[Unit]
Description=celestia bridge
After=network-online.target

[Service]
User=$USER
ExecStart=$(which celestia) bridge start  --core.ip localhost --core.grpc.port ${CELESTIA_PORT}090 --core.rpc.port ${CELESTIA_PORT}657 --keyring.accname my_celes_key --p2p.network blockspacerace

Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Enable and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable celestia-bridge
sudo systemctl restart celestia-bridge && sudo journalctl -u celestia-bridge -f
```
## Task: Deploy Bridge Node
>To complete task, put your wallet `address` and `Bridge Node ID` on [dahboard](https://celestia.knack.com/theblockspacerace)

This is an RPC call in order to get your node's peerId information. NOTE: You can only generate an auth token after initializing and starting your celestia-node.

~~~bash
NODE_TYPE=bridge
AUTH_TOKEN=$(celestia $NODE_TYPE auth admin --p2p.network blockspacerace)
~~~

Then you can get the peerId of your node with the following curl command:

~~~bash
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
~~~

## Task: Restart Your Node With Metrics Flags for Tracking Uptime
>To complete task, add metric flag, restart node and complete task on [dahboard](https://celestia.knack.com/theblockspacerace)

Update service file

```bash
sudo tee /etc/systemd/system/celestia-bridge.service > /dev/null <<EOF
[Unit]
Description=celestia bridge
After=network-online.target

[Service]
User=$USER
ExecStart=$(which celestia) bridge start  --core.ip localhost --core.grpc.port ${CELESTIA_PORT}090 --core.rpc.port ${CELESTIA_PORT}657 --keyring.accname my_celes_key --p2p.network blockspacerace --metrics.tls=false --metrics --metrics.endpoint otel.celestia.tools:4318
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Restart service

```bash
sudo systemctl daemon-reload
sudo systemctl restart celestia-bridge && sudo journalctl -u celestia-bridge -f
```

## Delete bridge node

~~~bash
sudo systemctl stop celestia-bridge
sudo systemctl disable celestia-bridge
sudo rm /etc/systemd/system/celestia-bridge*
rm -rf $HOME/celestia-node $HOME/.celestia-app $HOME/.celestia-bridge-blockspacerace-0
~~~