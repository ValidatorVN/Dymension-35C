# Dymension-35C

Cấu hình khuyến cáo:

    4 Core
    16gb Ram
    500gb SSD
    Băng thông 100mbps

Cài đặt chung:

    sudo apt update && sudo apt upgrade -y
    
Cài đặt các package cần thiết:

    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y
    
Cài đặt Golang:

    ver="1.19.1" 
    cd $HOME 
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" 

    sudo rm -rf /usr/local/go 
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" 
    rm "go$ver.linux-amd64.tar.gz"
    
Chuyển thư mục Golang:

    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
    source $HOME/.bash_profile
    go version
    
1/ Tải về bộ github dự án:

    git clone https://github.com/dymensionxyz/dymension.git --branch v0.2.0-beta
    cd dymension
    make install
    
2/ Kiểm tra lại xem chính xác thông tin phiên bản:

    dymd version --long
    
    name: dymension
    server_name: dymd
    version: v0.2.0-beta
    commit: 987e33407911c0578251f3ace95d2382be7e661d
    
3/ Tạo tên moniker, có thể thay đổi tên của bạn:

    dymd init <MONIKER> --chain-id 35-C
    
Thêm thông tin cho node về các tuỳ biến gas, seed, peer... etc:

    sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001udym"|g' $HOME/.dymension/config/app.toml
    sed -i -e 's/external_address = \"\"/external_address = \"'$(curl httpbin.org/ip | jq -r .origin)':26656\"/g' ~/.dymension/config/config.toml
    sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.dymension/config/app.toml
    sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.dymension/config/app.toml
    sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.dymension/config/app.toml
    sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.dymension/config/app.toml
    sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.dymension/config/config.toml
 
 5/ Khởi tạo hệ thống:
 
    sudo tee /etc/systemd/system/dymd.service > /dev/null << EOF
    [Unit]
    Description=Dymension testnet Node
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which dymd) start
    Restart=on-failure
    RestartSec=10
    LimitNOFILE=10000
    [Install]
    WantedBy=multi-user.target
    EOF
    
Lệnh chạy hệ thống và kiểm tra logs:
  
    sudo systemctl daemon-reload
    sudo systemctl enable dymd
    sudo systemctl start dymd

    sudo journalctl -u dymd -f --no-hostname -o cat
    
6/ Tạo ví và tạo validator:

    dymd keys add wallet
    
    dymd tx staking create-validator \
    --amount=1000000udym \
    --pubkey=$(dymd tendermint show-validator) \
    --moniker="Node & Validator VietNam" \
    --identity=6CB6AC3E672AAB9D \
    --details="https://t.me/NodeValidatorVietNam" \
    --chain-id=35-C \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.05 \
    --min-self-delegation=1 \
    --fees=1000udym \
    --from=wallet \
    -y
