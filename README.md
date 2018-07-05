# ethereum
[Screen教學](https://github.com/TitanLi/ethereum/blob/master/screen.md)

```10.0.0.83```
```sudo docker run -tid --name titan-eth-node1 -p 8545:8545 ubuntu:16.04 /bin/bash```
## 開啟node1、node2 Docker container
* node1
```
$ sudo docker run -tid --name node1 ubuntu:16.04 /bin/bash
$ sudo docker ps
$ sudo docker exec -ti 7aaf5c474ca4 /bin/bash
```
* node2
```
$ sudo docker run -tid --name node2 ubuntu:16.04 /bin/bash
$ sudo docker ps
$ sudo docker exec -ti dbc20b4329e7 /bin/bash
```
## 在node1、node2裡安裝 Ethereum 與 geth
```
# apt-get update
# apt-get install software-properties-common -y
# add-apt-repository -y ppa:ethereum/ethereum
# apt-get update
# apt-get install ethereum -y
```
## 在node1、node2裡安裝需要使用到的套件
* vim用來編輯檔案
```
# apt-get install vim -y
```
* 安裝nodeJS環境
```
# apt-get update
# apt-get install build-essential libssl-dev -y
# apt-get install curl -y
# curl https://raw.githubusercontent.com/creationix/nvm/v0.25.0/install.sh | bash
# source ~/.profile
# nvm --version
# nvm install v8
# nvm alias default 8
```
* 安裝git
```
# apt-get install git -y
```
## 在node1建立創世區塊
```
# puppeth
+-----------------------------------------------------------+
| Welcome to puppeth, your Ethereum private network manager |
|                                                           |
| This tool lets you create a new Ethereum network down to  |
| the genesis block, bootnodes, miners and ethstats servers |
| without the hassle that it would normally entail.         |
|                                                           |
| Puppeth uses SSH to dial in to remote servers, and builds |
| its network components out of Docker containers using the |
| docker-compose toolset.                                   |
+-----------------------------------------------------------+
Please specify a network name to administer (no spaces or hyphens, please)
> apple

Sweet, you can set this via --network=apple next time!

INFO [06-03|08:22:17] Administering Ethereum network           name=apple
WARN [06-03|08:22:17] No previous configurations found         path=/root/.puppeth/apple

What would you like to do? (default = stats)
 1. Show network stats
 2. Configure new genesis
 3. Track new remote server
 4. Deploy network components
> 2

Which consensus engine to use? (default = clique)
 1. Ethash - proof-of-work
 2. Clique - proof-of-authority
> 1

Which accounts should be pre-funded? (advisable at least one)
> 0x

Specify your chain/network ID if you want an explicit one (default = random)
> 100

What would you like to do? (default = stats)
 1. Show network stats
 2. Manage existing genesis
 3. Track new remote server
 4. Deploy network components
> 2

 1. Modify existing fork rules
 2. Export genesis configuration
 3. Remove genesis configuration
> 2

//輸入匯出檔案的檔名，可自行定義
Which file to save the genesis into? (default = apple.json)
> 

> ctrl c
```
會產生apple.json

複製apple.json到另一個container

## 在node1建立視覺化介面
```
# git clone https://github.com/cubedro/eth-netstats
# cd eth-netstats
# npm install
# npm install -g grunt-cli
```

## 在node1、node2加入視覺化介面的節點
```
# git clone https://github.com/cubedro/eth-net-intelligence-api
# cd eth-net-intelligence-api
# npm install
# npm install -g pm2
```

## 在node1、node2查看IP
```
# ip -4 addr
```

## 在node1更改：/eth-net-intelligence-api/app.json
```
# vim ./eth-net-intelligence-api/app.json

//node1 IP
"RPC_HOST"        : "172.17.0.2"
"INSTANCE_NAME"   : "node-1"
//node1 IP
"WS_SERVER"       : "http://172.17.0.2:3000"
//密碼
"WS_SECRET"       : "101"
```

## 在node2更改：/eth-net-intelligence-api/app.json
```
# vim ./eth-net-intelligence-api/app.json

//node2 IP
"RPC_HOST"        : "172.17.0.3"
"INSTANCE_NAME"   : "node-2"
//node1 IP
"WS_SERVER"       : "http://172.17.0.2:3000"
//密碼
"WS_SECRET"       : "101"
```

## 在node1進入eth-netstats，啟動視覺化介面
```
# cd eth-netstats
# grunt
# screen
ctrl + A + D
# WS_SECRET=101 npm start

//[detached from 6679.pts-1.93ba2461887e]
```

## 在node1,node2啟動 API Service
```
# cd eth-net-intelligence-api
# pm2 start app.json
```

## 在node1
* IP:172.17.0.2
* 進入有apple.json的目錄
* [detached from 15979.pts-1.93ba2461887e]
```
# geth --datadir .etherum/ init apple.json
# geth --nodiscover --networkid 100 --datadir .etherum/ --rpc --rpcapi eth,net,web3 --rpcaddr=0.0.0.0 console
//使用grpc API
# geth --nodiscover --networkid 100 --datadir .etherum/ --rpc --rpcapi admin,debug,miner,personal,txpool,eth,net,web3 --rpcaddr=0.0.0.0 console
> admin.nodeInfo
> admin.addPeer("enode://421694075ed7dc89257d2560d790c7fc608651c80b4ca3d6f405028ebd19acd84a9a7f899b6f122284f07f9f9b3ca178ec96c3484916b5c494c048f8d671108d@172.17.0.3:30303")
//查看結果
> admin.peers
[{
    caps: ["eth/63"],
    id: "421694075ed7dc89257d2560d790c7fc608651c80b4ca3d6f405028ebd19acd84a9a7f899b6f122284f07f9f9b3ca178ec96c3484916b5c494c048f8d671108d",
    name: "Geth/v1.8.10-stable-eae63c51/linux-amd64/go1.10",
    network: {
      inbound: true,
      localAddress: "172.17.0.2:30303",
      remoteAddress: "172.17.0.3:59946",
      static: false,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 524288,
        head: "0xa9d7e0b39e945d323f936a5415280b5ddbc04651ea6047b098767a50bf728999",
        version: 63
      }
    }
}]
//建帳戶：
> personal.newAccount()
//密碼
Passphrase: 
Repeat passphrase:
"0x6a890f7cb6cc354c44101cba5e93469a72935329"
//查看結果
> eth.accounts
//啟動
> miner.start()
```
## 在node2
* IP:172.17.0.3
* 進入有apple.json的目錄
```
# geth --datadir .etherum/ init apple.json
# geth --nodiscover --networkid 100 --datadir .etherum/ --rpc --rpcapi eth,net,web3 --rpcaddr=0.0.0.0 console
> admin.nodeInfo
> admin.addPeer("enode://72b752a011393397dd7bb4d90fc0fc5a1a00d156621cc5b2c4f399391517cfc258efbeccd9662979077a857e832a0c614e6995980be8360ebf724f8fb613684c@172.17.0.2:30303")
//查看結果
> admin.peers
[{
    caps: ["eth/63"],
    id: "72b752a011393397dd7bb4d90fc0fc5a1a00d156621cc5b2c4f399391517cfc258efbeccd9662979077a857e832a0c614e6995980be8360ebf724f8fb613684c",
    name: "Geth/v1.8.10-stable-eae63c51/linux-amd64/go1.10",
    network: {
      inbound: false,
      localAddress: "172.17.0.3:59946",
      remoteAddress: "172.17.0.2:30303",
      static: true,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 524288,
        head: "0xa9d7e0b39e945d323f936a5415280b5ddbc04651ea6047b098767a50bf728999",
        version: 63
      }
    }
}]
//建帳戶：
> personal.newAccount()
//密碼
Passphrase: 
Repeat passphrase: 
"0xf6dceb4114a9873b3f8f75cc9445f4405b624385"
//查看結果
> eth.accounts
//啟動
> miner.start()
```