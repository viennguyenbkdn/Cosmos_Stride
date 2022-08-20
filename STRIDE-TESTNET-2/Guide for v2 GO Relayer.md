# Setup v2 GO Relayer between Stride and GAIA on existing specific channels
1. RPC Endpoint of GAIA fullnode
    + If you intend to setup your own node, follow below guideline from kjnode
        - [GAIA node setup guide](https://github.com/kj89/testnet_manuals/tree/main/stride/GAIA/README.md)
    + Otherwise, you can use some RPC public endpoints of GAIA fullnode

2. Your Stride fully synced fullnode 

3. Set indexer to kv on each chain  

        sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.stride/config/config.toml
        sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.gaia/config/config.toml  
        

4. Expose your RPC endpoint to public, then Hermes can be reached (If Hermes and your fullnodes are on same vps, no need to do it)   

        sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.stride/config/config.toml  
        sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.gaia/config/config.toml  
	        

5. Restart fullnode after changing in step 3 & 4
6. Install GO and RUST if not install yet

        ###### GO Installation
        # Check whether GO is installed or not
        go version
        
        # If no result, do below command to install GO
        sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu -y
        ver="1.18.3"
        cd $HOME
        wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
        sudo rm -rf /usr/local/go
        sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
        rm "go$ver.linux-amd64.tar.gz"
        echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
        source ~/.bash_profile
        go version
        
        ###### RUST Installation
        # Check whether RUST is installed or not
        rustc --version
      
        # If no result, install it
        curl https://sh.rustup.rs/ -sSf | sh
        source $HOME/.cargo/env
        
7. Install v2 GO Relayer
```
git clone https://github.com/cosmos/relayer.git
cd relayer/
git checkout v2.0.0-rc4
make install     
which rly
chmod +x $(which rly)
```
8. Setup v2 GO Relayer between Stride and GAIA
```
YOUR_DISCORD_NAME="Put your discord ID"
cd ~ && rly config init --memo "$YOUR_DISCORD_NAME"
        
### Set variable
SRC_CHAIN="STRIDE-TESTNET-4"
SRC_KEY="stride-rly"
SRC_MNEMONIC_PHRASE="24 seed phrases"
        
DST_CHAIN="GAIA"
DST_KEY="gaia-rly"
DST_MNEMONIC_PHRASE="24 seed phrases"

PATH_NAME="YOUR CUSTOMIZE NAME"
        
### Make config data
cd $HOME/.relayer/config
cp config.yaml config.yaml-bak
        
sudo tee $HOME/.relayer/config/config.yaml > /dev/null <<EOF
global:
    api-listen-addr: :5183
    timeout: 300s
    memo: $YOUR_DISCORD_NAME
    light-cache-size: 20
chains:
    GAIA:
        type: cosmos
        value:
            key: $DST_KEY
            chain-id: $DST_CHAIN
            rpc-addr: http://95.216.21.32:23657
            account-prefix: cosmos
            keyring-backend: test
            gas-adjustment: 1.2
            gas-prices: 0.0025uatom
            debug: true
            timeout: 300s
            output-format: json
            sign-mode: direct
            memo-prefix: $YOUR_DISCORD_NAME
    STRIDE-TESTNET-4:
        type: cosmos
        value:
            key: $SRC_KEY
            chain-id: $SRC_CHAIN
            rpc-addr: http://localhost:16657
            account-prefix: stride
            keyring-backend: test
            gas-adjustment: 1.2
            gas-prices: 0.0025ustrd
            debug: true
            timeout: 300s
            output-format: json
            sign-mode: direct
            memo-prefix: $YOUR_DISCORD_NAME
paths:
    $PATH_NAME:
        src:
            chain-id: $DST_CHAIN
            client-id: 07-tendermint-0
            connection-id: connection-0
        dst:
            chain-id: $SRC_CHAIN
            client-id: 07-tendermint-0
            connection-id: connection-0
        src-channel-filter:
            rule: "allowlist"
            channel-list: [channel-0, channel-1, channel-2, channel-3, channel-4]	      
EOF
        
### Import keys into GO Relayer
rly keys restore $SRC_CHAIN $SRC_KEY "$SRC_MNEMONIC_PHRASE"
rly keys restore $DST_CHAIN $DST_KEY "$DST_MNEMONIC_PHRASE"

### Check balance 
rly q balance $SRC_CHAIN
rly q balance $DST_CHAIN
```

9. Make systemd service
```
sudo tee /etc/systemd/system/rlyd.service > /dev/null <<EOF
[Unit]
Description=V2 Go relayer
After=network-online.target

[Service]
User=$USER
ExecStart=$(which rly) start $PATH_NAME --memo "$YOUR_DISCORD_NAME"
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable rlyd
sudo systemctl restart rlyd
```

10. Monitor log until new channel is created
```
sudo journalctl -fu rlyd -o cat
```

11. Try to send raw data between 2 relayers via established channel ID (optional)   
```
### STRIDE to GAIA
rly transact transfer $SRC_CHAIN $DST_CHAIN 1000ustrd $(rly chains address $DST_CHAIN) channel-0 --path $PATH_NAME

2022-08-03T02:14:04.891557Z     info    Successful transaction  {"provider_type": "cosmos", "chain_id": "STRIDE-TESTNET-4", "packet_src_channel": "channel-0", "packet_dst_channel": "channel-0", "gas_used": 88442, "fees": "234ustrd", "fee_payer": "stride122qwd8nyxx4ywyc3c0hgwlq25a82j4vpgd3h94", "height": 63136, "msg_types": ["/ibc.applications.transfer.v1.MsgTransfer"], "tx_hash": "38338B0EC4F30498CA51D7DBE8B3937C15E02961FDFA1A8A008C7F85742312F8"}

### GAIA to STRIDE
rly transact transfer $DST_CHAIN $SRC_CHAIN 100uatom $(rly chains address $SRC_CHAIN) channel-0 --path $PATH_NAME

2022-08-03T02:15:50.495329Z     info    Successful transaction  {"provider_type": "cosmos", "chain_id": "GAIA", "packet_src_channel": "channel-0", "packet_dst_channel": "channel-0", "gas_used": 88763, "fees": "234uatom", "fee_payer": "cosmos122qwd8nyxx4ywyc3c0hgwlq25a82j4vptx3t3e", "height": 121362, "msg_types": ["/ibc.applications.transfer.v1.MsgTransfer"], "tx_hash": "549636B06F7F141E2965A4D467494589E683390634551B558A041A461F2B5DB0"}
```

12. Check transaction of your wallet on explorer to findout the message Update Client for GO-Relayer
![image](https://user-images.githubusercontent.com/91453629/182510531-4a80e5fa-e113-485a-8e29-f71df360fff2.png)
