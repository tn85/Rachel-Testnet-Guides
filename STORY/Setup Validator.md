# Story Validator Install Guide

## System Requirements
| Category | Requirements |
| ------------ | ------------ |
| CPU | 4 cores |
| RAM | 8+ GB |
| Bandwidth | 10 MBps for Download / Upload |
| Disk | 200+ GB |


### 1. Install dependencies
```bash
sudo apt update
sudo apt-get update
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 -y
```
### 2. Download Story-Geth binary
```bash
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
tar -xzvf geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
fi
sudo cp geth-linux-amd64-0.9.2-ea9f0d2/geth $HOME/go/bin/story-geth
source $HOME/.bash_profile
story-geth version
```

### 3. Download Story binary
```bash
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar -xzvf story-linux-amd64-0.9.11-2a25df1.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
fi
sudo cp story-linux-amd64-0.9.11-2a25df1/story $HOME/go/bin/story
source $HOME/.bash_profile
story version
```

### 4. Init Iliad node
```bash
story init --network iliad --moniker "Your_moniker_name"
```

### 5. Create story-geth service file
```bash
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

### 6. Create story service file
```bash
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

### 7. Reload and start story-geth
```bash
sudo systemctl daemon-reload && \
sudo systemctl start story-geth && \
sudo systemctl enable story-geth && \
sudo systemctl status story-geth
```

### 8. Reload and start story
```bash
sudo systemctl daemon-reload && \
sudo systemctl start story && \
sudo systemctl enable story && \
sudo systemctl status story
```

### 9. Check logs
```bash
sudo journalctl -u story-geth -f -o cat
```

- Wait a minute for connect peers

```bash
sudo journalctl -u story -f -o cat
```

### 10. Check sync status
```bash
curl localhost:26657/status | jq
```
![image](https://github.com/user-attachments/assets/9f418369-e7b0-43d8-be84-7508f49b2801)

- Waiting for your node `catching_up`
  is `false`
  you can create validator.

### 11. Create validator

### Export validator Public Key & Private key
By default, when you run `story init`
a validator key is created for you. To view your validator key, run the following command:

```bash
story validator export
```

In addition, if you want to export the derived EVM private key of your validator into the default data config directory, please run the following:

```bash
story validator export --export-evm-key
```
- ## Important: Please keep your private key in a safe place Your EVM Private Key saved to: `/root/.story/story/config/private_key.txt`
- ## Faucet link: https://faucet.story.foundation/

### Create validator
```bash
story validator create --stake 1000000000000000000 --private-key "your_private_key"
```

### Validator Staking
```bash
story validator stake \
   --validator-pubkey "VALIDATOR_PUB_KEY_IN_BASE64" \
   --stake 1000000000000000000 \
   --private-key xxxxxxxxxxxxxx
```
- Replace `VALIDATOR_PUB_KEY_IN_BASE64`

### Check your Validator on Explorer

- ## Get your validator info:
```bash
curl -s localhost:26657/status | jq -r '.result.validator_info'
```
## Result:

```bash
{ 
    "address": "D6F92FD7D0460AA9E4CF4D299FE479E93395DCF3", 
    "pub_key": { 
          "type": "tendermint/PubKeySecp256k1",
          "value": "A+46wEmBx5QQscNOKhmJgaAQjdr85s1OzvNimMiaysp3" 
    }, 
     "voting_power": "15000" 
}
```




 
