# Setup Hermes Relayer between Stride and GAIA on existing specific channels
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
        
        git clone https://github.com/cosmos/relayer.git
        cd relayer/
        git checkout v2.0.0-rc4
        make install     
        which rly

8. Setup v2 GO Relayer between Stride and GAIA

        YOUR_DISCORD_NAME="Put your discord ID"
        cd ~ && rly config init --memo "$YOUR_DISCORD_NAME"
        
        ### Set variable
        SRC_CHAIN="STRIDE-TESTNET-2"
        SRC_KEY="stride-rly"
        SRC_MNEMONIC_PHRASE="notable object move space find nose chunk picture episode scrap pupil field crash parent anger then muscle impose ahead strong settle youth lazy flower"
        
        DST_CHAIN="GAIA"
        DST_KEY="gaia-rly"
        DST_MNEMONIC_PHRASE="notable object move space find nose chunk picture episode scrap pupil field crash parent anger then muscle impose ahead strong settle youth lazy flower"
        
        ### Make config data
        cd $HOME/.relayer/config
        cp config.yaml config.yaml-bak
        
        sudo tee $HOME/.relayer/config/config.toml > /dev/null <<EOF
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
              rpc-addr: http://185.249.225.35:23657
              grpc-addr: http://185.249.225.35:23661
              account-prefix: cosmos
              keyring-backend: test
              gas-adjustment: 1.2
              gas-prices: 0.0025uatom
              debug: true
              timeout: 300s
              output-format: json
              sign-mode: direct
              memo-prefix: $YOUR_DISCORD_NAME
        STRIDE-TESTNET-2:
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
          STRIDE-GAIA:
            src:
              chain-id: $SRC_CHAIN
              client-id: 07-tendermint-0
              connection-id: connection-0
            dst:
              chain-id: $DST_CHAIN
              client-id: 07-tendermint-0
              connection-id: connection-0
            src-channel-filter:
              rule: "allowlist"
              channel-list: [channel-0, channel-1, channel-2, channel-3, channel-4]
        EOF
        
        
        
        ### Import keys into GO Relayer
        rly keys restore $SRC_CHAIN $SRC_KEY "$SRC_MNEMONIC_PHRASE"
        rly keys restore $DST_CHAIN $DST_KEY "$DST_MNEMONIC_PHRASE"
