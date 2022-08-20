Guide is used to setup ICQ between GAIA and Strided
1. Download and compile to binary file
```
cd $HOME
wget https://github.com/Stride-Labs/interchain-queries/archive/refs/tags/v0.0.4.zip
unzip v0.0.4.zip
cd interchain-queries-0.0.4/
go build
chmod +x interchain-queries
mv interchain-queries /usr/local/bin/
```

2. Make configuration file for ICQ
```
cd $HOME && mkdir .icq
sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: STRIDE-TESTNET-4
chains:
  GAIA:
    key: gaia-icq
    chain-id: GAIA
    rpc-addr: http://95.216.21.32:23657
    grpc-addr: http://95.216.21.32:23090
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 1uatom
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  STRIDE-TESTNET-4:
    key: stride-icq
    chain-id: STRIDE-TESTNET-4
    rpc-addr: http://localhost:16657
    grpc-addr: http://localhost:16090
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 1ustrd
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF
```
_**Edit RPC,GPRC and Port if you using your own GAIA/STRIDE nodes**_

3. Create v2 GO relayer as below link
 - [V2 GO relayer](https://github.com/viennguyenbkdn/Cosmos_Stride/blob/ICQ/STRIDE-TESTNET-2/Guide%20for%20v2%20GO%20Relayer.md)

4. Import these wallets of STRIDE and GAIA chains which are being used for V2 GO Relayer
```
interchain-queries keys restore --chain GAIA gaia-icq
interchain-queries keys restore --chain STRIDE-TESTNET-4 stride-icq
```

5. Create systemd service for ICQ
```
sudo tee /etc/systemd/system/icqd.service > /dev/null <<EOF
[Unit]
Description=ICQ 
After=network-online.target

[Service]
User=$USER
ExecStart=$(which interchain-queries) run
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable icqd
sudo systemctl restart icqd
```

6. Monitor journal log and wait a moment
```
journalctl -u icqd -f -o cat
```
- Output will be as below
```
Started V2 Go relayer.
store/bank/key
height parsed from GetHeightFromMetadata= 0
store/bank/key
height parsed from GetHeightFromMetadata= 0
Fetching client update for height height 164764
Fetching client update for height height 164764
Requerying lightblock
Requerying lightblock
ICQ RELAYER | query.Height= 0
ICQ RELAYER | res.Height= 164763
ICQ RELAYER | query.Height= 0
ICQ RELAYER | res.Height= 164763
Send batch of 4 messages
1 ClientUpdate message
1 SubmitResponse message
1 ClientUpdate message
1 SubmitResponse message
Sent batch of 2 (deduplicated) messages
```
- Check txh on your wallet from the explorer: https://explorer.theamsolutions.info/ (easy to track detail message)
![image](https://user-images.githubusercontent.com/91453629/183116210-ab7ff7f8-ce6f-4e29-9433-c6ddeb9b96aa.png)

- Detail message will be shown
![image](https://user-images.githubusercontent.com/91453629/183116801-8fff472c-c03d-4b9e-a4b9-982bbf95dd43.png)


