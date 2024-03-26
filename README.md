# Aligned-Layer-Guide_Node

Update system

    sudo apt update && sudo apt upgrade -y
    sudo apt install make curl git wget htop tmux build-essential jq make lz4 gcc unzip -y

Install Go

    ver="1.21.5"
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
    rm "go$ver.linux-amd64.tar.gz"
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
    source $HOME/.bash_profile
    go version

Install Node

    cd $HOME
    rm -rf $HOME/aligned_layer_tendermint && git clone --depth 1 --branch v0.0.2 https://github.com/yetanotherco/aligned_layer_tendermint && cd
    $HOME/aligned_layer_tendermint/cmd/alignedlayerd && go build 
    chmod +x alignedlayerd
    sudo mv alignedlayerd /usr/local/bin/
    cd $HOME
    alignedlayerd version

Initialize Node: Replace NodeName with your own moniker.

    alignedlayerd init NodeName --chain-id alignedlayer

Download Genesis & Download addrbook

    curl -Ls https://snap.nodex.one/alignedlayer-testnet/genesis.json > $HOME/.alignedlayer/config/genesis.json
    curl -Ls https://snap.nodex.one/alignedlayer-testnet/addrbook.json > $HOME/.alignedlayer/config/addrbook.json

Seed & Peer & Gasmini &......

    PEERS=a1a98d9caf27c3363fab07a8e57ee0927d8c7eec@128.140.3.188:26656,1beca410dba8907a61552554b242b4200788201c@91.107.239.79:26656,f9000461b5f535f0c13a543898cc7ac1cd10f945@88.99.174.203:26656,ca2f644f3f47521ff8245f7a5183e9bbb762c09d@116.203.81.174:26656
    STAKETAB_PEER=dc2011a64fc5f888a3e575f84ecb680194307b56@148.251.235.130:20656
    sed -i -e 's|^persistent_peers *=.*|persistent_peers = "'$STAKETAB_PEER','$PEERS'"|' $HOME/.alignedlayer/config/config.toml
    sed -i -e 's|^seeds *=.*|seeds = "'$SEEDS'"|' $HOME/.alignedlayer/config/config.toml  

Mini Gas

        sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001stake\"|" $HOME/.alignedlayer/config/app.toml

Create Service

    sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null <<EOF
    [Unit]
    Description=alignedlayerd
    After=network-online.target
    [Service]
    User=root
    ExecStart=$(which alignedlayerd) start
    Restart=always
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF

    cd $HOME
    curl -L https://snap.nodex.one/alignedlayer-testnet/alignedlayer-latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.alignedlayer

    cd $HOME
    sudo systemctl daemon-reload
    sudo systemctl enable alignedlayerd
  

Launch Node

    sudo systemctl restart alignedlayerd
    sudo journalctl -u alignedlayerd -f -o cat
    
