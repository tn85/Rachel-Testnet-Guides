## 0gchain Storage Node Deployment Guide version: ``v0.3.4``

### 1. Install dependencies
   ```bash
   sudo apt-get update
   sudo apt-get install clang cmake build-essential
   ```

### 2. Install GO
   ```bash
   cd $HOME && \
   ver="1.22.0" && \
   wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
   sudo rm -rf /usr/local/go && \
   sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
   rm "go$ver.linux-amd64.tar.gz" && \
   echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
   source ~/.bash_profile && \
   go version
   ```

### 3. Install Rustup
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

### 4. Set Up Environment Variables
   
   ```bash
   read -p "Enter json-rpc: " BLOCKCHAIN_RPC_ENDPOINT && echo "Current json-rpc: $BLOCKCHAIN_RPC_ENDPOINT"
   ENR_ADDRESS=$(curl -s ifconfig.me)
   echo "export ENR_ADDRESS=${ENR_ADDRESS}" >> ~/.bash_profile
   echo 'export ZGS_LOG_DIR="$HOME/0g-storage-node/run/log"' >> ~/.bash_profile
   echo 'export ZGS_LOG_SYNC_BLOCK="802"' >> ~/.bash_profile
   echo 'export LOG_CONTRACT_ADDRESS="0x8873cc79c5b3b5666535C825205C9a128B1D75F1"' >> ~/.bash_profile
   echo 'export MINE_CONTRACT="0x85F6722319538A805ED5733c5F4882d96F1C7384"' >> ~/.bash_profile
   echo "export BLOCKCHAIN_RPC_ENDPOINT=\"$BLOCKCHAIN_RPC_ENDPOINT\"" >> ~/.bash_profile
   
   source ~/.bash_profile
   
   echo -e "\n\033[31mCHECK YOUR STORAGE NODE VARIABLES\033[0m\n\nLOG_CONTRACT_ADDRESS: $LOG_CONTRACT_ADDRESS\nMINE_CONTRACT: $MINE_CONTRACT\nZGS_LOG_SYNC_BLOCK: $ZGS_LOG_SYNC_BLOCK\nBLOCKCHAIN_RPC_ENDPOINT: $BLOCKCHAIN_RPC_ENDPOINT\n\n" 
   ```

   You can check your ENR_ADDRESS again for sure
     
     cd 0g-storage-node
     echo $ENR_ADDRESS

### 5. download binary
   ```bash
   cd $HOME
   sudo apt install git cargo
   git clone https://github.com/0glabs/0g-storage-node.git
   cd $HOME/0g-storage-node
   git stash
   git fetch --all --tags
   git tag -d v0.3.4
   git checkout 5b6a4c716174b4af1635bfe903cd4f82894e0533
   git submodule update --init
   cargo build --release
   ```

### 6. wallet private key check

   ```bash
   read -sp "Enter your private key: " PRIVATE_KEY && echo
   ```

### 7. update node configuration
   ```bash
   sed -i '
   s|^miner_key = ""|miner_key = "'"$PRIVATE_KEY"'"|
   s|^\s*#\?\s*network_dir\s*=.*|network_dir = "network"|
   s|^\s*#\?\s*network_enr_address\s*=.*|network_enr_address = "'"$ENR_ADDRESS"'"|
   s|^\s*#\?\s*network_enr_tcp_port\s*=.*|network_enr_tcp_port = 1234|
   s|^\s*#\?\s*network_enr_udp_port\s*=.*|network_enr_udp_port = 1234|
   s|^\s*#\?\s*network_libp2p_port\s*=.*|network_libp2p_port = 1234|
   s|^\s*#\?\s*network_discovery_port\s*=.*|network_discovery_port = 1234|
   s|^\s*#\s*rpc_listen_address\s*=.*|rpc_listen_address = "0.0.0.0:5678"|
   s|^\s*#\?\s*rpc_enabled\s*=.*|rpc_enabled = true|
   s|^\s*#\?\s*db_dir\s*=.*|db_dir = "db"|
   s|^\s*#\?\s*log_config_file\s*=.*|log_config_file = "log_config"|
   s|^\s*#\?\s*log_directory\s*=.*|log_directory = "log"|
   s|^\s*#\?\s*network_boot_nodes\s*=.*|network_boot_nodes = \["/ip4/54.219.26.22/udp/1234/p2p/16Uiu2HAmTVDGNhkHD98zDnJxQWu3i1FL1aFYeh9wiQTNu4pDCgps","/ip4/52.52.127.117/udp/1234/p2p/16Uiu2HAkzRjxK2gorngB1Xq84qDrT4hSVznYDHj6BkbaE4SGx9oS","/ip4/18.167.69.68/udp/1234/p2p/16Uiu2HAm2k6ua2mGgvZ8rTMV8GhpW71aVzkQWy7D37TTDuLCpgmX"]|
   s|^\s*#\?\s*log_contract_address\s*=.*|log_contract_address = "'"$LOG_CONTRACT_ADDRESS"'"|
   s|^\s*#\?\s*mine_contract_address\s*=.*|mine_contract_address = "'"$MINE_CONTRACT"'"|
   s|^\s*#\?\s*log_sync_start_block_number\s*=.*|log_sync_start_block_number = '"$ZGS_LOG_SYNC_BLOCK"'|
   s|^\s*#\?\s*blockchain_rpc_endpoint\s*=.*|blockchain_rpc_endpoint = "'"$BLOCKCHAIN_RPC_ENDPOINT"'"|
   ' $HOME/0g-storage-node/run/config-testnet.toml
   ```

### 8. create service
   ```bash
   sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
   [Unit]
   Description=ZGS Node
   After=network.target
   
   [Service]
   User=$USER
   WorkingDirectory=$HOME/0g-storage-node/run
   ExecStart=$HOME/0g-storage-node/target/release/zgs_node --config $HOME/0g-storage-node/run/config-testnet.toml
   Restart=on-failure
   RestartSec=10
   LimitNOFILE=65535
   
   [Install]
   WantedBy=multi-user.target
   EOF
   ```

### 9. start the node
   ```bash
   sudo systemctl daemon-reload && \
   sudo systemctl enable zgs && \
   sudo systemctl start zgs && \
   sudo systemctl status zgs
   ```

### 10. show logs by date
   - check the logs file
   ```bash
   ls -lt $ZGS_LOG_DIR
   ```
   - full logs command
   ```bash
   tail -f ~/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d)
   ```
   - tx_seq-only logs command
   ```bash
   tail -f ~/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d) | grep tx_seq:
   ```


   ### Check Peers Connected
```bash
curl -X POST http://localhost:5678 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"zgs_getStatus","params":[],"id":1}'  | jq
```


 