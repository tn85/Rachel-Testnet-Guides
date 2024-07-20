
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
read -p "Enter the file name you want to create (leave blank to use tmp123456): " FILE_NAME
if [ -z "$FILE_NAME" ]; then
  FILE_NAME="tmp123456"
  ./0g-storage-client gen
else
  ./0g-storage-client gen --file $FILE_NAME
fi
 ```

## 3.Prepare for CLI
### Input your json-rpc and storage node url
 ```bash
STORAGE_PORT=$(grep -oP '(?<=rpc_listen_address = "0.0.0.0:)\d+(?=")' $HOME/0g-storage-node/run/config-testnet.toml)
NETWORK_ENR_ADDRESS=$(sed -n 's/network_enr_address = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config-testnet.toml)
STORAGE_RPC_ENDPOINT=http://$NETWORK_ENR_ADDRESS:$STORAGE_PORT
BLOCKCHAIN_RPC_ENDPOINT=$(sed -n 's/blockchain_rpc_endpoint = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config-testnet.toml)
LOG_CONTRACT_ADDRESS=$(sed -n 's/log_contract_address = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config-testnet.toml)
MINE_CONTRACT_ADDRESS=$(sed -n 's/mine_contract_address = "\([^"]*\)"/\1/p' $HOME/0g-storage-node/run/config-testnet.toml)
echo -e "STORAGE_RPC_ENDPOINT: $STORAGE_RPC_ENDPOINT\nLOG_CONTRACT_ADDRESS: $LOG_CONTRACT_ADDRESS\nMINE_CONTRACT_ADDRESS: $MINE_CONTRACT_ADDRESS\nBLOCKCHAIN_RPC_ENDPOINT: $BLOCKCHAIN_RPC_ENDPOINT"

echo -e "\033[1;35m-byRachelnguyen\033[0m"


```
### Input your private key 
 ```bash
 read -sp "Enter private key: " PRIVATE_KEY
 echo $PRIVATE_KEY

 ```

## 4.Upload the test file with storage CLI
Your node will submit the file upload and you will need to WAIT until the file completes its process

 ```bash
# Upload the generated test file
cd $HOME/0g-storage-client

./0g-storage-client upload \
--url $BLOCKCHAIN_RPC_ENDPOINT \
--contract $LOG_CONTRACT_ADDRESS \
--key $PRIVATE_KEY \
--node $STORAGE_RPC_ENDPOINT \
--file $FILE_NAME

 ```
## 5.Download the test file with storage CLI

Set file_root_hash you want to download to environment
 ```bash
root=<file_root_hash you want to download>
 ```

 ```bash
# Download the uploaded test file
cd $HOME/0g-storage-client

./0g-storage-client download \
--node $STORAGE_RPC_ENDPOINT \
--root $root \
--file ${FILE_NAME}_dl
 ```
