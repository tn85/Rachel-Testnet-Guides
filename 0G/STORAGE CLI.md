
# 0G Storage CLI
### Storage CLI is a client tool that interact with Storage node

## 1. Build Storage CLI with source code
 ```bash
 git clone https://github.com/0glabs/0g-storage-client.git
cd 0g-storage-client
go build
 ```

## 2. Create a Test file for uploading with storage CLI
 ```bash
cd $HOME/0g-storage-client
./0g-storage-client gen
# this command will create a test file "tmp123456" in your location
# add --file <file-name> to get a specific file name.filetype
 ```

## 3.Prepare for CLI
### Input your json-rpc and storage node url
 ```bash
STORAGE_PORT=$(grep -oP '(?<=rpc_listen_address = "0.0.0.0:)\d+(?=")' $HOME/0g-storage-node/run/config.toml)
STORAGE_RPC_ENDPOINT=http://$(wget -qO- eth0.me):$STORAGE_PORT
BLOCKCHAIN_RPC_ENDPOINT=$(sed -n 's/blockchain_rpc_endpoint = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config.toml)
LOG_CONTRACT_ADDRESS=$(sed -n 's/log_contract_address = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config.toml)
MINE_CONTRACT_ADDRESS=$(sed -n 's/mine_contract_address = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config.toml)
JSON_PORT=$(sed -n '/\[json-rpc\]/,/^address/ s/address = "0.0.0.0:\([0-9]*\)".*/\1/p' $HOME/.0gchain/config/app.toml)
JSON_RPC_ENDPOINT=http://$(wget -qO- eth0.me):$JSON_PORT
echo -e "STORAGE_RPC_ENDPOINT: $STORAGE_RPC_ENDPOINT\nLOG_CONTRACT_ADDRESS: $LOG_CONTRACT_ADDRESS\nMINE_CONTRACT_ADDRESS: $MINE_CONTRACT_ADDRESS\nBLOCKCHAIN_RPC_ENDPOINT: $BLOCKCHAIN_RPC_ENDPOINT\nJSON_RPC_ENDPOINT: $JSON_RPC_ENDPOINT"

echo -e "\033[1;35m-byRachelnguyen\033[0m"
```
### Input your private key 
 ```bash
 read -sp "Enter private key: " PRIVATE_KEY && echo
 ```

## 4.Upload the test file with storage CLI
Change the file name to upload any other file ( --file <file-name>)
Your node will submit a the file upload and you will need to `WAIT` until the file complete its process 

 ```bash
# upload test file generated
cd $HOME/0g-storage-client

./0g-storage-client upload \
--url $BLOCKCHAIN_RPC_ENDPOINT \
--contract $LOG_CONTRACT_ADDRESS \
--key $PRIVATE_KEY \
--node $STORAGE_RPC_ENDPOINT \
--file tmp123456

 ```
## 5.Download the test file with storage CLI
 ```bash
# set file_root_hash you want to download to environment
root=<file_root_hash you want to download>
 ```

 ```bash
# download test file uploaded
cd $HOME/0g-storage-client

./0g-storage-client download \
--node $STORAGE_RPC_ENDPOINT \
--root $root \
--file tmp123456_dl
 ```
