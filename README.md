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

    minimum-gas-prices = "0.15udym"
    sed -i -e 's/external_address = \"\"/external_address = \"'$(curl httpbin.org/ip | jq -r .origin)':26656\"/g' ~/.dymension/config/config.toml
    seeds = "c6cdcc7f8e1a33f864956a8201c304741411f219@3.214.163.125:26656"
    persistent_peers = "ebc272824924ea1a27ea3183dd0b9ba713494f83@dymension-testnet-peer.autostake.net:27086,9111fd409e5521470b9b33a46009f5e53c646a0d@178.62.81.245:45656,f8a0d7c7db90c53a989e2341746b88433f47f980@209.182.238.30:30657,1bffcd1690806b5796415ff72f02157ce048bcdd@144.76.67.53:2580,c17a4bcba59a0cbb10b91cd2cee0940c610d26ee@95.217.144.107:20556,e6ea3444ac85302c336000ac036f4d86b97b3d3e@38.146.3.199:20556,b473a649e58b49bc62b557e94d35a2c8c0ee9375@95.214.53.46:36656,db0264c412618949ce3a63cb07328d027e433372@146.19.24.101:26646,281190aa44ca82fb47afe60ba1a8902bae469b2a@dymension.peers.stavr.tech:17806,290ec1fc5cc5667d4e072cf336758d744d291ef1@65.109.104.118:60556,d8b1bcfc123e63b24d0ebf86ab674a0fc5cb3b06@51.159.97.212:26656,55f233c7c4bea21a47d266921ca5fce657f3adf7@168.119.240.200:26656,139340424dddf85e54e0a54179d06875013e1e39@65.109.87.88:24656"
 
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
