# Setup Hermes Relayer between Stride and Juno/GAIA
1. RPC Endpoint of GAIA fullnode, Juno fullnode
    + If you intend to setup your own node, follow below guideline from kjnode
        - [Juno node setup guide](https://github.com/kj89/testnet_manuals/tree/main/juno)
        - [GAIA node setup guide](https://github.com/kj89/testnet_manuals/tree/main/stride/GAIA/README.md)
    + Otherwise, you can use some RPC public endpoints of GAIA or Juno fullnode

2. Your Stride fully synced fullnode 

3. Set indexer to kv on each chain  

        sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.stride/config/config.toml
        sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.juno/config/config.toml  
        sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.gaia/config/config.toml  
        

4. Expose your RPC endpoint to public, then Hermes can be reached (If Hermes and your fullnodes are on same vps, no need to do it)   

        sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.stride/config/config.toml  
        sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.juno/config/config.toml  
        sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.gaia/config/config.toml  
	        

5. Restart fullnode after changing in step 3 & 4

6. Install Hermes binary

		mkdir -p $HOME/.hermes/bin  
        cd $HOME/.hermes/bin  
        wget https://github.com/informalsystems/ibc-rs/releases/download/v1.0.0-rc.0/hermes-v1.0.0-rc.0-x86_64-unknown-linux-gnu.tar.gz
        tar -C $HOME/.hermes/bin/ -vxzf hermes-v1.0.0-rc.0-x86_64-unknown-linux-gnu.tar.gz
        echo 'export PATH="$HOME/.hermes/bin:$PATH"' >> $HOME/.bash_profile
        source $HOME/.bash_profile
        hermes version

7. Make configuration data for Hermes:   
  Note: 
      - Pay attention to following parameters **rpc_addr, grpc_addr, websocket_addr**           
         + If Hermes and your fullnode are on same vps, format of these parameter will be:  rpc_addr='http://localhost:RPC_PORT' (Rerfe step 5)  
         + If Hermes and your fullnode are on different vps, format of these parameter will be: rpc_addr='http://VPS_IP:RPC_PORT' (Refer step 5)         
      - The parameter **key_name** must be same as name of wallet to be added into Hermes later
  
_7.1   Example of Config.toml for JUNO & STRIDE (STRIDE and HERMES are on same VPS, JUNO is on another one)_
```
sudo tee $HOME/.hermes/config.toml > /dev/null <<EOF
[global]
log_level = 'info'

[mode]
[mode.clients]
enabled = true
refresh = true
misbehaviour = true

[mode.connections]
enabled = false

[mode.channels]
enabled = false

[mode.packets]
enabled = true
clear_interval = 100
clear_on_start = true
tx_confirmation = false

[rest]
enabled = true
host = '127.0.0.1'
port = 3000

[telemetry]
enabled = true
host = '127.0.0.1'
port = 3001

[[chains]]
id = 'STRIDE-TESTNET-4'
rpc_addr = 'http://10.10.10.10:16657'
grpc_addr = 'http://10.10.10.10:16090'
websocket_addr = 'ws://10.10.10.10:16657/websocket'
rpc_timeout = '300s'
account_prefix = 'stride'
key_name = 'stride-rly'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 100000
max_gas = 4000000
gas_price = { price = 0.0025, denom = 'ustrd' }
gas_multiplier = 1.2
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '30s'
max_block_time = '30s'
trusting_period = '36000s'
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
 policy = 'allow'
 list = [
   ['transfer', '*']
 ]

[[chains]]
id = 'uni-3'
rpc_addr = 'http://localhost:17657'
grpc_addr = 'http://localhost:17090'
websocket_addr = 'ws://localhost:17657/websocket'
rpc_timeout = '300s'
account_prefix = 'juno'
key_name = 'juno-rly'
store_prefix = 'ibc'
default_gas = 100000
max_gas = 4000000
gas_price = { price = 0.0025, denom = 'ujunox' }
gas_multiplier = 1.2
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '30s'
max_block_time = '30s'
trusting_period = '2days'
trust_threshold = { numerator = '1', denominator = '3' }
address_type = { derivation = 'cosmos' }
[chains.packet_filter]
 policy = 'allow'
 list = [
   ['transfer', '*']
 ]
 EOF
  ```
  
_7.2   Config.toml for GAIA & STRIDE_  
  ```
sudo tee $HOME/.hermes/config.toml > /dev/null <<EOF  
[global]
log_level = 'info'

[mode]
[mode.clients]
enabled = true
refresh = true
misbehaviour = true

[mode.connections]
enabled = false

[mode.channels]
enabled = false

[mode.packets]
enabled = true
clear_interval = 100
clear_on_start = true
tx_confirmation = true

[rest]
enabled = true
host = '127.0.0.1'
port = 3000

[telemetry]
enabled = true
host = '127.0.0.1'
port = 3001

[[chains]]
id = 'STRIDE-TESTNET-4'
rpc_addr = 'http://localhost:16657'
grpc_addr = 'http://localhost:16090'
websocket_addr = 'ws://localhost:16657/websocket'
rpc_timeout = '300s'
account_prefix = 'stride'
key_name = 'stride-rly'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 100000
max_gas = 4000000
gas_price = { price = 0.001, denom = 'ustrd' }
gas_multiplier = 1.2
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '25s'
max_block_time = '30s'
trusting_period = '36000s'
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
  ['ica*', '*'],
  ['transfer', 'channel-0']
]

[[chains]]
id = 'GAIA'
rpc_addr = 'http://localhost:23657'
grpc_addr = 'http://localhost:23090'
websocket_addr = 'ws://localhost:23657/websocket'
rpc_timeout = '300s'
account_prefix = 'cosmos'
key_name = 'gaia-rly'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 100000
max_gas = 4000000
gas_price = { price = 0.001, denom = 'uatom' }
gas_multiplier = 1.2
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '25s'
max_block_time = '30s'
trusting_period = '36000s'
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
  ['ica*', '*'],
  ['transfer', 'channel-0']
]
EOF
  ```   
  
8. Create wallet on each chain, the wallets must have fund. You can use your current wallet which are being used for fullnode/validator also.  
8.1 For new wallet (bypass 8.2 if do 8.1)
    - Create new one then apply faucet  
      ```
      strided keys add stride-rly --output json | jq > /root/.hermes/stride-rly.json
      gaiad keys add gaid-rly --output json | jq > /root/.hermes/gaia-rly.json
      junod keys add juno-rly --output json | jq > /root/.hermes/juno-rly.json
      ```   
  8.2 For current using wallet (bypass 8.1 if you do 8.2)  
    - Create json file of corresponding wallet on each chain with below format       
      
    ###### STRIDE wallet #####
    ### Export json file
    strided keys show WALLET_NAME --output json | jq > /root/.hermes/stride-rly.json
    
    ### Show json file
    cat /root/.hermes/stride-rly.json
    {
        "name":"stride-rly",
        "type":"local",
        "address":"WALLET-ADDRESS",
        "pubkey":"{\"@type\":\"/cosmos.crypto.secp256k1.PubKey\",\"key\":\"PUBLIC-KEY\"}"    
    }
    
    ### Add 24 seed phrases of wallet into json file, after that the file will be as below
    cat /root/.hermes/stride-rly.json       
    {
        "name":"stride-rly",
        "type":"local",
        "address":"WALLET-ADDRESS",
        "pubkey":"{\"@type\":\"/cosmos.crypto.secp256k1.PubKey\",\"key\":\"PUBLIC-KEY\"}",
        "mnemonic":"24 seed phrases"
    }        
    
    ### Do the same for GAIA or Juno wallet as above to create json file
    gaiad keys show WALLET_NAME --output json | jq > /root/.hermes/gaia-rly.json
    junod keys show WALLET_NAME --output json | jq > /root/.hermes/juno-rly.json
 
9. Import wallet into hermes   
    ```
    cd /root/.hermes/
    hermes keys add --chain STRIDE-TESTNET-2 --key-file stride-rly.json
    hermes keys add --chain uni-3 --key-file juno-rly.json
    hermes keys add --chain GAIA --key-file gaia-rly.json
    ```

10. Health check and validate configuration
    ```
    hermes health-check
    hermes config validate
    ```

10. Create systemd service for Hermes and start it.
    ```
    sudo tee /etc/systemd/system/hermesd.service > /dev/null <<EOF
    [Unit]
    Description=hermes
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which hermes) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF

    sudo systemctl daemon-reload
    sudo systemctl enable hermesd
    sudo systemctl restart hermesd
    ```

11. Monitor log until new channel is created
    ```
    sudo journalctl -fu hermesd -o cat
    ```
    ![image](https://user-images.githubusercontent.com/91453629/181575011-a88a3240-ef87-4828-8001-28ded96b78ed.png)

12. Try to send raw data between 2 relayers via established channel ID (optional)   
	```
    ### Example for Juno & Stride
    root@Contabo16g-204:~/.hermes# hermes tx raw ft-transfer --dst-chain STRIDE-TESTNET-2 --src-chain uni-3 --src-port transfer --src-channel channel-154 --amount 1000 --denom ujunox --timeout-height-offset 1000 --number-msgs 1
	2022-07-28T06:05:10.932566Z  INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml'
	2022-07-28T06:05:11.017838Z  INFO ThreadId(09) wait_for_block_commits: waiting for commit of tx hashes(s) 3DA3A81F21F0BC44D4294D2B0D245DB8213E31AB5A205A133C4A14AFC52D6ABB id=uni-3
	Success: [
    	SendPacket(
        	SendPacket - h:3-987116, seq:3, path:channel-154/transfer->channel-32/transfer, toh:2-14940, tos:Timestamp(NoTimestamp)),
    	),
	]

	root@Contabo16g-204:~/.hermes# hermes tx raw ft-transfer --dst-chain uni-3 --src-chain STRIDE-TESTNET-2 --src-port transfer --src-channel channel-32 --amount 1000 --denom ustrd --timeout-height-offset 1000 --number-msgs 1
	2022-07-28T06:07:22.864250Z  INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml'
	2022-07-28T06:07:33.214633Z  INFO ThreadId(09) wait_for_block_commits: waiting for commit of tx hashes(s) 2A31B32C77DE2AE9564C0F41A28BC914DF08040A0B503319E801B78121E1AFEB id=STRIDE-TESTNET-2
	Success: [
    	SendPacket(
        	SendPacket - h:2-13951, seq:4, path:channel-32/transfer->channel-154/transfer, toh:3-988135, tos:Timestamp(NoTimestamp)),
    	),
	]
    
    ### Example for GAIA & Stride
    root@Contabo32g:~/.hermes# hermes tx raw ft-transfer --dst-chain GAIA --src-chain STRIDE-TESTNET-2 --src-port transfer --src-channel channel-0 --amount 1000 --denom ustrd --timeout-height-offset 1000 --number-msgs 1
	2022-07-28T10:54:56.184868Z  INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml'
	2022-07-28T10:54:58.342749Z  INFO ThreadId(10) wait_for_block_commits: waiting for commit of tx hashes(s) 93CD35E9E449ACD34EA74C31D0E9C691709F610003FC6630D1027179DDDE312C id=STRIDE-TESTNET-2
	Success: [
    	SendPacket(
        	SendPacket - h:2-15333, seq:580, path:channel-0/transfer->channel-0/transfer, toh:0-25850, tos:Timestamp(NoTimestamp)),
    	),
	]
	root@Contabo32g:~/.hermes# hermes tx raw ft-transfer --dst-chain STRIDE-TESTNET-2 --src-chain GAIA --src-port transfer --src-channel channel-0 --amount 1000 --denom uatom --timeout-height-offset 1000 --number-msgs 1
	2022-07-28T10:55:45.959731Z  INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml'
	2022-07-28T10:55:46.607686Z  INFO ThreadId(11) wait_for_block_commits: waiting for commit of tx hashes(s) 4BB7B4D671D69F95E69B5D88361C9724BA8FE0B1D2CAABAAC289D43DE2D368EF id=GAIA
	Success: [
    	SendPacket(
        	SendPacket - h:0-24861, seq:692, path:channel-0/transfer->channel-0/transfer, toh:2-16337, tos:Timestamp(NoTimestamp)),
    	),
	]
	root@Contabo32g:~/.hermes#
    ```

14. Check transaction of your wallet on explorer to findout the message Update Client (IBC)
![image](https://user-images.githubusercontent.com/91453629/181577502-5d7b2e12-f7f5-4edb-bd36-b05d2e1cf500.png)
