# Setup IBC Relayer between Stride and Juno/GAIA
**A. Preparation**
  1. RPC Endpoint of GAIA fullnode, Juno fullnode
    + If you intend to setup your own node, follow below guideline from kjnode
        - [Juno node setup guide](https://github.com/kj89/testnet_manuals/tree/main/juno)
        - [GAIA node setup guide](https://github.com/kj89/testnet_manuals/tree/main/stride/GAIA/README.md)
    + Otherwise, you can use some RPC public endpoint of GAIA or Juno fullnode
  2. Your Stride fully synced fullnode
  3. Create wallet on each chain, the wallets must have fund. You can use your current wallet which using for fullnode also.
  4. Set indexer to kv on each chain
        ```
        sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.stride/config/config.toml
        sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.juno/config/config.toml
        sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.gaia/config/config.toml
        ```
  5. Expose your RPC endpoint to public, then Hermes can be reached (If Hermes and your fullnodes are same vps, no need to do it)
        ```
        sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.stride/config/config.toml
        sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.juno/config/config.toml
        sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.gaia/config/config.toml
        ```
  6. Restart fullnode after changing in step 5 & 6
  7. Install Hermes binary
        ```
        mkdir -p $HOME/.hermes/bin
        cd $HOME/.hermes/bin
        wget https://github.com/informalsystems/ibc-rs/releases/download/v1.0.0-rc.0/hermes-v1.0.0-rc.0-x86_64-unknown-linux-gnu.tar.gz
        tar -C $HOME/.hermes/bin/ -vxzf hermes-v1.0.0-rc.0-x86_64-unknown-linux-gnu.tar.gz
        echo 'export PATH="$HOME/.hermes/bin:$PATH"' >> $HOME/.bash_profile
        source $HOME/.bash_profile
        hermes version
        ```
  8. Make configuration data for Hermes:   
  Note: 
      - Pay attention to below parameter **rpc_addr, grpc_addr, websocket_addr**           
         + If Hermes and your fullnode are on same vps, format of these parameter will be:  rpc_addr='http://localhost:RPC_PORT' (Rerfe step 5)  
         + If Hermes and your fullnode are on different vps, format of these parameter will be: rpc_addr='http://VPS_IP:RPC_PORT' (Refer step 5)         
      - The parameter **keyname** must be same as name of wallet to be added into Hermes later
  
  _8.1   Example of Config.toml for JUNO & STRIDE (STRIDE and HERMES are on same VPS, JUNO is on another one)_
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
id = 'STRIDE-TESTNET-2'
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
gas_price = { price = 0.001, denom = 'ustrd' }
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
gas_price = { price = 0.001, denom = 'ujunox' }
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
  
  _8.2   Config.toml for GAIA & STRIDE_  
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
id = 'STRIDE-TESTNET-2'
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
  
  
