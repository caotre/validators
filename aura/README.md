Đăng ký server qua link referral dưới, bạn sẽ tặng mình một ly cafe <3

| NCC           | Link           | Note |
| ------------- |:-------------:     |:-------------: |
| Vultr         | [Đăng ký ngay ](https://www.vultr.com/?ref=6846932)|Áp mã FLYVULTR250 để nhận $250 khi đăng ký qua link referral|
| Hetzner      | [Đăng ký ngay ](https://hetzner.cloud/?ref=fS54U43gi4aS)    |...|
| Digital Ocean | [Đăng ký ngay ](https://m.do.co/c/5ef6c574cded)     |Đăng ký qua link này, bạn được D.O tặng $200 và 60 ngày sử dụng cho tài khoản mới|

# Cập nhật OS và cài các packages cần thiết
```
sudo apt update && sudo apt upgrade -y && sudo apt-get install -y build-essential curl wget jq lz4
```
Thêm bộ nhớ Swap nếu RAM server của bạn ít
```
sudo swapoff -a
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## Cài Go
```
cd $HOME
wget https://golang.org/dl/go1.19.2.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.2.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source ~/.bash_profile
go version
```

## Clone repo và cài binary aurad
```
cd $HOME
git clone https://github.com/aura-nw/aura && cd aura
git pull
git checkout aura_v0.4.4
make install
aurad version
```
## Cập nhật repo
```
cd aura
git pull
git checkout aura_v0.4.x
make install
aurad version
```

## 	Initialize node 
```
aurad init tienthuattoan --chain-id xstaxy-1
```
## Create key
```
aurad keys add tienthuattoan
```
Kết quả trả về
```
aurad keys add tienthuattoan
- name: tienthuattoan
  type: local
  address: aura1620wkw9ku7lcrpjnq3pavfxm2sjrv4s43xlk60
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"Aown5jS69MqAXNGFG0zmJ9Zpx1KuQljxIx0eApixjgvf"}'
  mnemonic: ""

**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.
xxx xxx xxx 
```
Bạn nhớ lưu lại đoạn mnemonic này cẩn thận. Từ mnemonic này sẽ khôi phục lại ví sau này khi cài lại node.

## Tải genesis.json file
```
curl -s https://raw.githubusercontent.com/aura-nw/mainnet-artifacts/main/xstaxy-1/genesis.json >~/.aura/config/genesis.json
```
Kiểm tra shasum file genesis.json
```
jq -S -c -M '' ~/.aura/config/genesis.json | sha256sum
90b9404d38167e3b40f56ddc11a1565f0107b89008742425e44905871699febc
```

## Set minimum gas price 
```
sed -i'' 's/minimum-gas-prices = ""/minimum-gas-prices = "0.001uaura"/' $HOME/.aura/config/app.toml
```


## Set seeds
```
sed -E -i 's/seeds = \".*\"/seeds = \"22a0ca5f64187bb477be1d82166b1e9e184afe50@18.143.52.13:26656,0b8bd8c1b956b441f036e71df3a4d96e85f843b8@13.250.159.219:26656\"/' $HOME/.aura/config/config.toml
```

## Update pruning to save disk space
```
sed -i 's/pruning = "default"/pruning = "custom"/g' \
$HOME/.aura/config/app.toml

sed -i 's/pruning-keep-recent = "0"/pruning-keep-recent = "100"/g' \
$HOME/.aura/config/app.toml

sed -i 's/pruning-interval = "0"/pruning-interval = "10"/g' \
$HOME/.aura/config/app.toml

sed -i 's/indexer = "kv"/indexer = "null"/g' \
$HOME/.aura/config/config.toml
```

## Create service 
```
sudo tee <<EOF >/dev/null /etc/systemd/system/aurad.service
[Unit]
Description=Aura node
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/aurad start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
EOF
```
## Khởi chạy service
```
sudo systemctl daemon-reload
sudo systemctl enable aurad
sudo systemctl restart aurad
sudo journalctl -u aurad -f
```
```
sudo systemctl status aurad
```
## Stop và cài lại node
```
sudo systemctl stop aurad
aurad tendermint unsafe-reset-all
```
## Open ports
Các port 26656, 26657, 1317, 9090 cần phải mở. Ở Vultr sẽ phải mở thủ công, các nhà cung cấp khác, các port đó thường tự mở
```
sudo apt-get install ufw
sudo ufw enable
sudo ufw allow 26656
sudo ufw allow 26657
sudo ufw allow 1317
sudo ufw allow 9090
sudo ufw allow 22
sudo ufw reload
sudo ufw status
```
## Create validator
```
aurad tx staking create-validator \
  --amount=1000000uaura \
  --pubkey $(aurad tendermint show-validator) \
  --moniker="TienThuatToan Capital" \
  --website="https://tienthuattoan.ventures" \
  --details="Guaranteed availability and up-time backed by a professional blockchain infrastructure team. Fast, datacenter-hosted, bare-metal validators" \
  --commission-rate="0" \
  --commission-max-rate="1" \
  --commission-max-change-rate="1" \
  --min-self-delegation="1" \
  --identity E46A1AC121BEAD77 \
  --from=tienthuattoan \
  --chain-id=xstaxy-1
```
NOTE: 1aura = 1000000 uaura
```
--commission-rate                   The initial commission rate percentage (Minimum 5%)
--commission-max-rate               The maximum commission rate percentage (Permanent once set)
--commission-max-change-rate        The maximum commission change rate percentage (per day)(Permanent once set)
--min-self-delegation               The minimum self delegation required on the validator
--moniker                           The validator's moniker
--details                           The validator's details
--website                           The validator's website
--security-contact                  The validators email address
--identity                          The validators keybase.io pgp id
```
## Edit validator
```
aurad tx staking edit-validator \
--chain-id xstaxy-1 \
--from tienthuattoan \
--commission-rate="0.01" \
--new-moniker="TienThuatToan Capital" \
--gas-prices="0.005uaura"
```

## Staking
```
aurad tx staking delegate \
auravaloper1620wkw9ku7lcrpjnq3pavfxm2sjrv4s425w7z3 \
1000000uaura \
--chain-id xstaxy-1 \
--from tienthuattoan

```
## Withdraw reward
```
aurad tx distribution withdraw-rewards auravaloper1620wkw9ku7lcrpjnq3pavfxm2sjrv4s425w7z3 --from aura1620wkw9ku7lcrpjnq3pavfxm2sjrv4s43xlk60 --chain-id xstaxy-1 --gas-prices="0.005uaura" --gas="300000" --commission -y
```
