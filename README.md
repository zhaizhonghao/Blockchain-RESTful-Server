> # Node RESTful API Based on Hyperledger Fabric

## Fabric基本环境的搭建
* 操作系统选择：CentOS 7(选择该操作系统可以省去很多的麻烦)
* 参照《Hyperledger Fabric手把手实战区块链实验指导书》的**Fabric基础环境搭建**和**Fabric Kafka生产环境部署**（该指导书写得相当地详细，只需要执照上面的指令一条一条地部署，可以完成hyperledger所需的基本环境的搭建，本人已经试验过2次，都有效）
## nodejs的环境搭建
***所有的操作都在Peer0.org1.example.com（172.32.0.40）这台机器上***
* 安装nvm
（1）下载并安装nvm的命令(这里的是v0.33.11，也可以选择更高的合适的版本)：`
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
`
（2）设置nvm自动运行
```shell
    echo "source ~/nvm/nvm.sh" >> ~/.bashrc
    source ~/.bashrc
```
* 通过nvm安装合适的node版本
（1）使用nvm查看最近的可安装的node版本：`nvm list-remote`
（2）使用nvm安装相应的node版本(这里安装的是v12.18.0)：`nvm install v12.18.0`
## RESTful服务器的部署
* 从github下载自己写的node代码, 该代码就放在`/root`目录下就可以：
```
    git clone https://github.com/zhaizhonghao/Blockchain-RESTful-Server.git
```
* 将**Fabric基本环境的搭建**中生成的`/kafkapeer `目录下的`crypto-config`文件夹拷到刚才下载的`Blockchain-RESTful-Server`文件夹中
* 通过npm下载依赖环境
```
    npm install
```
* 针对不同的网络一定要注意修改profiles中的配置文件
（1）修改两个[组织名]-client.yaml对于的组织名。本次实验只有两个组织org1和org2
（2）修改dev-connection.yaml里面的内容
首先是，organizations部分，这里有两个提供peer节点的组织org1和org2，要和之前的配置Fabric配置时的配置文件中的配置保持一致。
其次是peers部分，该部分配置peer节点能够拥有的功能。
下面是orderers部分，模仿github下载下来的格式进行配置，主要是替换其中ip地址，以及tlsCACerts下面的path，千万别漏了grpcOptions，（这次配置的时候就漏了，因此出了错）。
最后，是peers的地址的配置，同样模仿github下载下来的格式进行配置，主要是替换其中ip地址，以及tlsCACerts下面的path。

```
---
#
# The network connection profile provides client applications the information about the target
# blockchain network that are necessary for the applications to interact with it.
name: "dev-network"

#
# Describe what the target network is/does.
#
description: "A development enviornment setup"

#
# Schema version of the content. Used by the SDK to apply the corresponding parsing rules.
#
version: "1.0"

#
# list of participating organizations in this network
#
organizations:
  Org1:
    mspid: Org1MSP
    peers:
      - peer0.org1.example.com
      - peer1.org1.example.com

  Org2:
    mspid: Org2MSP
    peers:
      - peer0.org2.example.com

#
# [Optional]. But most apps would have this section so that channel objects can be constructed
# based on the content below. If an app is creating channels, then it likely will not need this
# section.
#
channels:
  # name of the channel
  mychannel:
    # List of orderers designated by the application to use for transactions on this channel. 
    orderers:
      - orderer0.example.com
      - orderer1.example.com
      - orderer2.example.com

    # Required. list of peers from participating orgs
    peers:
      # Org1 peer - with roles played by the peer
      peer0.org1.example.com:
        # Roles for which this peer may be used
        endorsingPeer: true
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: true

      # Org1 peer - with roles played by the peer
      peer1.org1.example.com:
        # Roles for which this peer may be used
        endorsingPeer: false  # SDK will NOT send request for endorsements to this peer
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: false    # SDK will NOT allow event subscribers for this peer


      # Org2 peer - with roles played by the peer
      peer0.org2.example.com:
        # Roles for which this peer may be used
        endorsingPeer: false  # SDK will NOT send request for endorsements to this peer
        chaincodeQuery: true
        ledgerQuery: true
        eventSource: false    # SDK will NOT allow event subscribers for this peer


      
#
# List of orderers to send transaction and channel create/update requests to. For the time
# being only one orderer is needed. 
#
orderers:
  orderer0.example.com:
    url: grpcs://172.32.0.36:7050

    # these are standard properties defined by the gRPCs library
    # they will be passed in as-is to gRPCs client constructor
    grpcOptions:
      ssl-target-name-override: orderer0.example.com

    # In dev environment the Orderer is NOT enabled for TLS
    tlsCACerts:
      path: ../crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

  orderer1.example.com:
    url: grpcs://172.32.0.37:7050

    # these are standard properties defined by the gRPCs library
    # they will be passed in as-is to gRPCs client constructor
    grpcOptions:
      ssl-target-name-override: orderer1.example.com

    # In dev environment the Orderer is NOT enabled for TLS
    tlsCACerts:
      path: ../crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

  orderer2.example.com:
    url: grpcs://172.32.0.38:7050

    # these are standard properties defined by the gRPCs library
    # they will be passed in as-is to gRPCs client constructor
    grpcOptions:
      ssl-target-name-override: orderer2.example.com

    # In dev environment the Orderer is NOT enabled for TLS
    tlsCACerts:
      path: ../crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

#
# List of peers to send various requests to, including endorsement, query
# and event listener registration.
#
peers:
  peer0.org1.example.com:

    url: grpcs://172.32.0.40:7051

    grpcOptions:
       ssl-target-name-override: peer0.org1.example.com

    tlsCACerts:
       path: ../crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/tlscacerts/tlsca.org1.example.com-cert.pem

  peer1.org1.example.com:

    url: grpcs://172.32.0.41:7051

    grpcOptions:
       ssl-target-name-override: peer1.org1.example.com

    tlsCACerts:
      path: ../crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp/tlscacerts/tlsca.org1.example.com-cert.pem

  peer0.org2.example.com:

    url: grpcs://172.32.0.42:7051

    grpcOptions:
      ssl-target-name-override: peer0.org2.example.com

    tlsCACerts:
      path: ../crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/tlscacerts/tlsca.org2.example.com-cert.pem
```
* 开启服务器

（1）安装forever工具（forever工具可以让我们的node服务器持久地在后台运行）
```
    npm install forever -g
```
（2）让系统开放3000端口
```
    firewall-cmd --zone=public --add-port=3000/tcp --permanent
    firewall -cmd --reload
```
（3）开启服务器，让其在后台运行
进入`Blockchain-RESTful-Server`文件夹中，运行
```
    forever start app.js
```



***
## DOC for API

* Initiate the credStore

* Query information of the specific peer
* Query the height of the lastest block
* Query the block information by the block height
* Query the hash of the genesis block
* Check the instantiated chaincodes in the specific channel
* Query the information about the transaction by TxID
* Import the identity into the wallet
* List the identities existing in the wallet
* Export the chosen identity from the wallet
* Install the chaincode in the specific channel
* Instantiate the chaincode in the specific channel
* Query the specific channel
* Invoke the specific channel

## TODO
* Create the channel
* Join the channel
* Generate the Identity
* Add a new organization
* Add a new peer
* Add a new user
* Add a new client