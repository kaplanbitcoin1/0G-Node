
![0G Node](https://github.com/user-attachments/assets/a0fabd2c-34f3-473f-b2cb-2c569c6c77f5)


## Sistem gereksinimleri
* CPU 8 Cores 
* Ram 16 GB 
* Disk 1 TB SSD
* Ubuntu 22.04



## Hazırsanız başlayalım
### Sunucumuzu güncelleyelim
```shell
sudo apt update && \
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

### Go yükleyelim
```Shell
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

### Binary Oluşturalım
```shell
git clone -b v0.2.3 https://github.com/0glabs/0g-chain.git
./0g-chain/networks/testnet/install.sh
source .profile
```

### Değişkenleri Ayarlayalım
* Moniker ve Wallet kısmını kendimize göre ayarlayalım. (Moniker sizin node isminiz olacak) 
```shell
echo 'export MONIKER="Moniker"' >> ~/.bash_profile
echo 'export CHAIN_ID="zgtendermint_16600-2"' >> ~/.bash_profile
echo 'export WALLET_NAME="Wallet"' >> ~/.bash_profile
source $HOME/.bash_profile
```

### Node'u başlatalım
```shell
cd $HOME
0gchaind config chain-id $CHAIN_ID
0gchaind init $MONIKER --chain-id $CHAIN_ID
```

### Genesis.json dosyasını indirelim 
```shell
curl -Ls https://github.com/0glabs/0g-chain/releases/download/v0.2.3/genesis.json > $HOME/.0gchain/config/genesis.json
```

### Config dosyasını güncelleyelim
```shell
PEERS="96d615925aee68b90bfaf18d461e799fdcb22211@45.10.162.96:26656,7253c5556119b84f581bf3479db33687c2ff5cfe@38.242.143.169:26656,0aa16751b6c1884e755997d08dc17f8582aa9e38@45.10.163.80:26656,85233db31304a69fb2dda924b5de31c22dfcff5a@45.10.161.188:26656,89e272c0e5007e391f420e4f45e1473f91995025@154.26.155.239:26656,df8947d0bd46f24590e5d4bc3c06c59d543572d0@65.109.92.18:36656,d7ca6521ee30f8cf9eaf32e9edee1101e44c48e9@45.10.161.5:26656,364c45b7cab8a095cb59443f3e91fd102ec9eb95@158.220.118.216:26656,c8807bba12fa67676319df8e049ae5fac690cf55@45.159.228.20:26656,cfd099ade96d82908b4ab185eddbf90379579bfc@84.247.149.9:26656,03619b6f90fab32cd5f0cadbe3021e6a3cda16e3@154.26.156.101:26656,b3411cfb89113055dce89277c7cc7029ce451090@195.201.242.107:26656,bed108e9ce56d84a574fa02df90c734281ae19ef@162.55.65.137:27856,057f64f293f0843c849aa3f1f1e20a1a0add29f8@45.159.222.237:26656,369666051d45ed28379db34a80dfdf13e43d3681@5.104.80.63:26656,bc8898c416f7b22e56782eb16803150fd90863b6@81.0.221.180:26656,6970d09a9e004f6132b30db6eb5e27b6bd53a1d8@158.220.89.199:26656,7ecfe8d9404a4e1ea36cba5d546650da2b97bfd2@45.90.122.129:26656,4d98cf3cb2a61238a0b1557596cdc4b306472cb9@95.216.228.91:13456" && \
SEEDS="81987895a11f6689ada254c6b57932ab7ed909b6@54.241.167.190:26656,010fb4de28667725a4fef26cdc7f9452cc34b16d@54.176.175.48:26656,e9b4bc203197b62cc7e6a80a64742e752f4210d5@54.193.250.204:26656,68b9145889e7576b652ca68d985826abd46ad660@18.166.164.232:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.0gchain/config/config.toml

sed -i 's|^\s*#\?\s*laddr\s*=\s*"tcp://127.0.0.1:26657"|laddr = "tcp://0.0.0.0:26657"|' $HOME/.0gchain/config/config.toml

sed -i 's|^\s*#\?\s*api\s*=.*|api = "eth,txpool,personal,net,debug,web3"|' $HOME/.0gchain/config/app.toml

sed -i 's|^\s*#\?\s*address\s*=\s*"127.0.0.1:8545"|address = "0.0.0.0:8545"|' $HOME/.0gchain/config/app.toml

sed -i 's|^\s*#\?\s*ws-address\s*=\s*"127.0.0.1:8546"|ws-address = "0.0.0.0:8546"|' $HOME/.0gchain/config/app.toml
```

### Servis oluşturalım
```shell
sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
[Unit]
Description=0gchaind Validator Node
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/0gchaind start
Environment="G0GC=900"
Environment="G0MELIMIT=40GB"
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Başlatalım
```shell
sudo systemctl daemon-reload
sudo systemctl enable 0gchaind 
sudo systemctl start 0gchaind
```

### Logları kontrol edelim
```shell
sudo journalctl -u 0gchaind -f -o cat
```

### Cüzdan oluşturalım
* Cüzdan isminizi kendinize göre belirleyin
* Çıktıyı kaydetmeyi unutmayalım

```shell
0gchaind keys add cüzdanismi --eth
```



Private Keyimizi dışa aktarıp Metamask'a ekleyelim
```shell
0gchaind keys unsafe-export-eth-key cüzdanismi
```

### Musluktan token isteyelim
[Faucet](https://faucet.0g.ai//)

### Validator oluşturalım
* Moniker, Wallet ve diğer bilgilerinizi değiştirip eklemeyi unutmayalım. 
```shell
0gchaind tx staking create-validator \
  --amount=1000000ua0gi \
  --pubkey=$(0gchaind tendermint show-validator) \
  --moniker=MONIKER \
  --chain-id=zgtendermint_16600-2 \
  --commission-rate=0.05 \
  --commission-max-rate=0.10 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation=1 \
  --from=CüzdanAdı \
  --identity="" \
  --website="" \
  --details="" \
  --node=http://localhost:56657 \
  -y
```



# Yararlı Bilgiler

Eski cüzdanınızı ekleme

```console
0gchaind keys add wallet --eth --recover
```

Cüzdanları Listeleme

```console
0gchaind keys list
```

Cüzdanı Silme

```shell
0gchaind keys delete wallet
```


### Kendine delege etme
* Wallet kısımlarını cüzdan isminizle değiştirelim

```console
0gchaind tx staking delegate $(0gchaind keys show wallet --bech val -a)  1000000ua0gi --from wallet -y
```




Node'u Silme

```console
cd $HOME
sudo systemctl stop 0gchaind
sudo systemctl disable 0gchaind
sudo rm /etc/systemd/system/0gchaind.service
sudo systemctl daemon-reload
sudo rm -f $(which 0gchaind)
sudo rm -rf $HOME/.0gchain
sudo rm -rf $HOME/0g-chain
sudo rm -rf $HOME/go
```

