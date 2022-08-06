# Install

## Update packages
```Bash
sudo apt update && sudo apt upgrade
```
## Install dependencies
```Bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils ncdu git jq liblz4-tool
```
### Install go
```Bash
wget "https://golang.org/dl/go1.18.2.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go1.18.2.linux-amd64.tar.gz"
rm "go1.18.2.linux-amd64.tar.gz"
echo "export GOROOT=/usr/local/go" >> ~/.bash_profile
echo "export GOPATH=$HOME/go" >> ~/.bash_profile
echo "export GO111MODULE=on" >> ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
```
## Download and build binaries
```Bash
git clone https://github.com/cosmos/relayer.git
cd relayer && git checkout v2.0.0-rc3
make install
```
## Make dir
```Bash
cd $HOME
mkdir -p $HOME/.relayer/config
```
## Create config.yaml
```Bash
sudo tee $HOME/.relayer/config/config.yaml > /dev/null <<EOF
global:
    api-listen-addr: :5183
    timeout: 10s
    memo: glukoss.Inc#9485
    light-cache-size: 20
chains:
    gaia:
        type: cosmos
        value:
            key: default
            chain-id: GAIA
            rpc-addr: http://78.107.234.44:36657
            account-prefix: cosmos
            keyring-backend: test
            gas-adjustment: 1.2
            gas-prices: 0.001uatom
            debug: true
            timeout: 20s
            output-format: json
            sign-mode: direct
    stride:
        type: cosmos
        value:
            key: default
            chain-id: STRIDE-TESTNET-2
            rpc-addr: http://38.242.221.166:26657
            account-prefix: stride
            keyring-backend: test
            gas-adjustment: 1.2
            gas-prices: 0.001ustrd
            debug: true
            timeout: 20s
            output-format: json
            sign-mode: direct
paths:
    stride-gaia:
        src:
            chain-id: STRIDE-TESTNET-2
            client-id: 07-tendermint-0
            connection-id: connection-0
        dst:
            chain-id: GAIA
            client-id: 07-tendermint-0
            connection-id: connection-0
        src-channel-filter:
            rule: allowlist
            channel-list:
                - channel-0
                - channel-1
                - channel-2
                - channel-3
                - channel-4
EOF
```
## Restore keys
```Bash
rly keys restore stride default "mnemonic"
rly keys restore gaia default "mnemonic"
```
## Check chains, clients, connections
```Bash
rly paths list
```
## Create service
```Bash
sudo tee /etc/systemd/system/rlyd.service > /dev/null <<EOF
[Unit]
Description=relayer_go
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which rly) start stride-gaia --log-format logfmt --processor events
Restart=on-failure
RestartSec=10
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
EOF
```
## Registry service
```Bash
sudo systemctl daemon-reload
sudo systemctl enable rlyd
sudo systemctl restart rlyd && sudo journalctl -u rlyd -f
```
