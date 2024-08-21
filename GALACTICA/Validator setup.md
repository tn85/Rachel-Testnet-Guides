### 1. Install dependencies for building from source

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y

## 2. Install GO
    cd $HOME
    VER="1.21.3"
    wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
    rm "go$VER.linux-amd64.tar.gz"
    [ ! -f ~/.bash_profile ] && touch ~/.bash_profile
    echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
    source $HOME/.bash_profile
    [ ! -d ~/go/bin ] && mkdir -p ~/go/bin

## 3. Set vars
Change `NODE_NAME` to your validdator name

    echo "export WALLET="wallet"" >> $HOME/.bash_profile
    echo "export MONIKER="NODE_NAME"" >> $HOME/.bash_profile
    echo "export GALACTICA_CHAIN_ID="galactica_9302-1"" >> $HOME/.bash_profile
    echo "export GALACTICA_PORT="19"" >> $HOME/.bash_profile
    source $HOME/.bash_profile

# 4. Build `galacticad` binary
    cd $HOME
    rm -rf galactica
    git clone https://github.com/Galactica-corp/galactica
    cd galactica
    git checkout v0.1.2
    make build
    mv $HOME/galactica/build/galacticad $HOME/go/bin

## 5. Initialize the node
```
galacticad config node tcp://localhost:${GALACTICA_PORT}657
```    
```
galacticad config keyring-backend os
```
```
galacticad config chain-id galactica_9302-1
```
```
galacticad init "thinaxinh" --chain-id galactica_9302-1
```

## 6. Download genesis and addrbook
    wget -O $HOME/.galactica/config/genesis.json https://server-4.itrocket.net/testnet/galactica/genesis.json
    wget -O $HOME/.galactica/config/addrbook.json  https://server-4.itrocket.net/testnet/galactica/addrbook.json

## 7. Set seeds and peers
    SEEDS="52ccf467673f93561c9d5dd4434def32ef2cd7f3@galactica-testnet-seed.itrocket.net:46656"
    PEERS="c9993c738bec6a10cfb8bb30ac4e9ae6f8286a5b@galactica-testnet-peer.itrocket.net:11656,64309c18c7bb536f79d566766873ec3dc3dfdd1b@198.7.121.177:22656,9990ab130eac92a2ed1c3d668e9a1c6e811e8f35@148.251.177.108:27456,6f5ea6dbdd258ab7ae6b30c76b5053993beb068f@65.109.52.156:46656,7b0c8f2dfca134ae2d4939cdb6f57353be77113c@135.181.216.54:3460,998472dbf0a6c8a162f9ddd201565ad6f9eabf2c@95.217.89.100:46656,f3cd6b6ebf8376e17e630266348672517aca006a@46.4.5.45:27456,7185be6d31b30299cac8f7b2ddab59c24e5dfb35@62.169.19.141:26656,232050092a90b94b002f34a79de8a3859ed13ee7@5.104.108.184:26656,b352df9a2ed42fd741120598fbecb188dc01a48e@84.247.191.148:26656,6b846b316d704d78f3f9ca46d86cebc5a22de8ae@161.97.111.249:26656,e104cf08023696afa2c364ca386b67c5db0982b5@151.115.60.105:46656,1dfafc2ebc9dc5cc06cb6040132b19c70fa48a9a@144.126.139.139:46655,d6d9badeefbc0bc80d263f3bfc196ab91a1049dc@[2402:1f00:8300:92c::]:13156"
    sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
           -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.galactica/config/config.toml


## 8. Set custom ports in app.toml
    sed -i.bak -e "s%:1317%:${GALACTICA_PORT}317%g;
    s%:8080%:${GALACTICA_PORT}080%g;
    s%:9090%:${GALACTICA_PORT}090%g;
    s%:9091%:${GALACTICA_PORT}091%g;
    s%:8545%:${GALACTICA_PORT}545%g;
    s%:8546%:${GALACTICA_PORT}546%g;
    s%:6065%:${GALACTICA_PORT}065%g" $HOME/.galactica/config/app.toml

```
sed -i.bak -e "s%:26658%:${GALACTICA_PORT}658%g;
s%:26657%:${GALACTICA_PORT}657%g;
s%:6060%:${GALACTICA_PORT}060%g;
s%:26656%:${GALACTICA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${GALACTICA_PORT}656\"%;
s%:26660%:${GALACTICA_PORT}660%g" $HOME/.galactica/config/config.toml
```

## 9. Config pruning
    sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.galactica/config/app.toml
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.galactica/config/app.toml
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.galactica/config/app.toml

## 10. Set minimum gas price, enable prometheus and disable indexing
    sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "10agnet"|g' $HOME/.galactica/config/app.toml
    sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.galactica/config/config.toml
    sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.galactica/config/config.toml

## 11. Create service file
    sudo tee /etc/systemd/system/galacticad.service > /dev/null <<EOF
    [Unit]
    Description=Galactica node
    After=network-online.target
    [Service]
    User=$USER
    WorkingDirectory=$HOME/.galactica
    ExecStart=$(which galacticad) start --home $HOME/.galactica --chain-id galactica_9302-1
    Restart=on-failure
    RestartSec=5
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF

## 12. Reset and download snapshot

```
galacticad tendermint unsafe-reset-all --home $HOME/.galactica
```    
    if curl -s --head curl https://server-4.itrocket.net/testnet/galactica/galactica_2024-08-21_1997389_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
      curl https://server-4.itrocket.net/testnet/galactica/galactica_2024-08-21_1997389_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.galactica
        else
      echo "no snapshot founded"
    fi

## 13. Start the node
    sudo systemctl daemon-reload
    sudo systemctl enable galacticad
    sudo systemctl restart galacticad && sudo journalctl -u galacticad -f

Make sure your node are fully synsynced before proceeding with the next steps
Once your node is fully synced, the output from above will print "false"
 
     galacticad status 2>&1 | jq 
  
   <img width="642" alt="Ảnh màn hình 2024-08-21 lúc 17 15 17" src="https://github.com/user-attachments/assets/b77bcf33-8b33-4356-b2ba-d4ae5cdbae37">

## 14. Create a wallet for your validator
To create a new wallet, use the following command. don’t forget to save the mnemonic
    
    galacticad keys add $WALLET

You can add --recover flag to restore existing key instead of creating

Save wallet and validator address
   
    WALLET_ADDRESS=$(galacticad keys show $WALLET -a)
    VALOPER_ADDRESS=$(galacticad keys show $WALLET --bech val -a)
    echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
    echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
    source $HOME/.bash_profile

## 15. Extract the HEX address to request some tokens from the faucet

    echo "0x$(galacticad debug addr $(galacticad keys show $WALLET -a) | grep hex | awk '{print $3}')"
    
## 16. Request tokens from the faucet

Faucet link: https://faucet-reticulum.galactica.com/

## 17. Check wallet balance

      galacticad query bank balances $WALLET_ADDRESS 

## 18. Create a validator

RÊMMBER TO CHANGE YOUR NODE NAME

    galacticad tx staking create-validator \
    --amount 100000agnet \
    --from $WALLET \
    --commission-rate 0.1 \
    --commission-max-rate 0.2 \
    --commission-max-change-rate 0.01 \
    --min-self-delegation 1 \
    --pubkey $(galacticad tendermint show-validator) \
    --moniker "YOUR_NODE_NAME" \
    --identity "" \
    --website "" \
    --details "" \
    --chain-id galactica_9302-1 \
    --gas 200000 --gas-prices 10agnet \
    -y

  # Useful commands

  ## Query your validator

     galacticad q staking validator $(galacticad keys show $WALLET --bech val -a)
 
  ## Delegate tokens to your validator

    galacticad tx staking delegate $(galacticad keys show $WALLET --bech val -a) <AMOUNT>agnet \
    --from wallet \
    --chain-id= galactica_9302-1 \
    --gas=auto \
    --gas-adjustment=1.4 \
    --gas-prices 300000agne \
    -y
















  
  

