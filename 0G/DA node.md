## DA Node Installation
Before you install the DA node, make sure you have more than 10 A0GI in your wallet. Click the button below to request from the faucet.

## Required Hardware Specifications
| Required | Specification |
|-|-
| CPU | 8 Cores |
| Memory | 16 GB |
| Storage | 1 TB NVMe SSD |
| Bandwidth | 100mbps |
| OS | Linux |

-----------------------------------------------------------------


### 1. Install Dependencies
```bash
sudo apt-get update
sudo apt-get install git cargo clang cmake build-essential pkg-config openssl libssl-dev protobuf-compiler
```

### 2. Install Rustup
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

You will be asked for Rust installation options, just use the default one. Press `1` then `enter` to continue the installation process.
![CleanShot 2024-04-13 at 15 07 52@2x](https://github.com/BlockchainsHub/Testnet/assets/77204008/bcb81284-8235-4cf2-a4f1-50821044cc21)

After the installation process is complete, run the following command, to restart the shell.
```bash
. "$HOME/.cargo/env"
```

### 3. Build Binary
```bash
git clone -b v1.0.2 https://github.com/0glabs/0g-da-node.git
cd $HOME/0g-da-node
git stash
git fetch --all --tags
git checkout aa8eb2120d3977000aa68da38776fc375c5f18a4 
git submodule update --init
cargo build --release
./dev_support/download_params.sh
sudo mv "$HOME/0g-da-node/target/release/server" /usr/local/bin
```

### 4. Create DB Directory
```bash
mkdir -p "$HOME/0g-da-node/db"
```

### 5. Create Config File
```bash
cp $HOME/0g-da-node/config_example.toml $HOME/0g-da-node/config.toml
```

### 6. Generate a BLS Private Key
Run the command below to generate a BLS private key. **Please keep the generated BLS private key carefully**.
```bash
cargo run --bin key-gen
```
> **DO NOT FORGET TO SAVE YOUR BLS PRIVATE KEY!**

### 7. Set Up Environment Variables
Run the command below and input your validator node IP and port in this format `http://x.x.x.x:8545`. 
```bash
read -p "Enter your validator node IP and port for eth_rpc_endpoint configuration: " BLOCKCHAIN_RPC_ENDPOINT

read -p "Enter your DA node IP and port for socket_address configuration: " SOCKET_ADDRESS

read -p "Enter your BLS private key for signer_bls_private_key configuration: " SIGNER_BLS_PRIVATE_KEY

read -p "Enter your private key for signer_eth_private_key configuration: " SIGNER_ETH_PRIVATE_KEY

read -p " Enter your private key for miner_eth_private_key: " MINER_ETH_PRIVATE_KEY
```

```bash
echo 'export ZGDA_CONFIG_FILE="$HOME/0g-da-node/config.toml"' >> ~/.bash_profile

echo 'export DB_DIR="$HOME/0g-da-node/db"' >> ~/.bash_profile

echo 'export ZGDA_ENCODER_PARAMS_DIR="$HOME/0g-da-node/params"' >> ~/.bash_profile

echo 'export BLOCKCHAIN_RPC_ENDPOINT="'$BLOCKCHAIN_RPC_ENDPOINT'"' >> ~/.bash_profile

echo 'export SOCKET_ADDRESS="'$SOCKET_ADDRESS'"' >> ~/.bash_profile

echo 'export SIGNER_BLS_PRIVATE_KEY="'$SIGNER_BLS_PRIVATE_KEY'"' >> ~/.bash_profile

echo 'export SIGNER_ETH_PRIVATE_KEY="'$SIGNER_ETH_PRIVATE_KEY'"' >> ~/.bash_profile

source ~/.bash_profile
```

### 8. Update Config File
```bash
sed -i "s|^\s*#\?\s*data_path\s*=.*|data_path = \"$DB_DIR\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*encoder_params_dir\s*=.*|encoder_params_dir = \"$ZGDA_ENCODER_PARAMS_DIR\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*eth_rpc_endpoint\s*=.*|eth_rpc_endpoint = \"$BLOCKCHAIN_RPC_ENDPOINT\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*socket_address\s*=.*|socket_address = \"$SOCKET_ADDRESS\"|" "$ZGDA_CONFIG_FILE"

sed -i 's|^\s*#\?\s*da_entrance_address\s*=.*|da_entrance_address = "0x857C0A28A8634614BB2C96039Cf4a20AFF709Aa9"|' "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*signer_bls_private_key\s*=.*|signer_bls_private_key = \"$SIGNER_BLS_PRIVATE_KEY\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*signer_eth_private_key\s*=.*|signer_eth_private_key = \"$SIGNER_ETH_PRIVATE_KEY\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*miner_eth_private_key\s*=.*|miner_eth_private_key = \"$MINER_ETH_PRIVATE_KEY\"|" "$ZGDA_CONFIG_FILE"

sed -i "s|^\s*#\?\s*start_block_number\s*= .*|start_block_number = "940000"|" "$ZGDA_CONFIG_FILE"
```

### 9. Create Service File
Create a service file to run the DA node in the background.
```bash
sudo tee /etc/systemd/system/zgda.service > /dev/null <<EOF
[Unit]
Description=0G DA Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/server --config $HOME/0g-da-node/config.toml
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 10. Start DA Node
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable zgda && \
sudo systemctl start zgda && \
sudo systemctl status zgda
```

-----------------------------------------------------------------

## Useful Commands
### Check Log
```bash
sudo journalctl -u zgda -f -o cat
```

### Restart the Node
```bash
sudo systemctl restart zgda
```

### Stop the Node
```bash
sudo systemctl stop zgda
```

### Delete the Node
```bash
sudo systemctl stop zgda
sudo systemctl disable zgda
sudo rm /etc/systemd/system/zgda.service
sudo rm /usr/local/bin/server
rm -rf $HOME/0g-da-node
```

-----------------------------------------------------------------

<p align="center">
  &copy; 2024 BlockHub. All rights reserved.
</p>
