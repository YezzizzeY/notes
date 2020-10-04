---
title: fabric 环境搭建
date: 2019-08-21 16:47:40
tags: hyperledger fabric
---
<details>
  <summary>详细信息</summary>


##### Fabric版本：1.4

##### Linux系统: Ubuntu 18.04

##### 所需工具: docker, docker-compose, go, gcc, git, nodejs

##### 搭建步骤：

##### 1.在$GOPATH/src/github.com/hyperledger下载源码: 

```
git clone https://github.com/hyperledger/fabric.git
```

##### 2.切换版本至1.4.0  

```
$ cd ~/go/src/github.com/hyperledger/fabric
```

    $ git checkout v1.4.0
##### 3.scripts目录下有个bootstrap.sh，运行得到cryptogen等工具

##### 4.下载fabric-samples

##### 5.在fabric-samples中的first-network里打开终端，运行

   

```
 ./byfn.sh up
```


​    之后成功会有以下一系列的显示

+ ```
  ##### 测试生成证书：

  /home/yezzi/gopath/src/fabric-samples/first-network/../bin/cryptogen

  ###### #

  ##### Generate certificates using cryptogen tool

  ###### #

  生成创世节点：

  /home/yezzi/gopath/src/fabric-samples/first-network/../bin/configtxgen

  ###### #

  ###### ###  Generating Orderer Genesis block

  ###### #

  生成channel工具：

  ###### #

  ### Generating channel configuration transaction 'channel.tx'

  ###### #

  启动测试节点：

  ###### #

  ###### #    Generating anchor peer update for Org1MSP

  ###### #

  ###### #

  ###### #    Generating anchor peer update for Org2MSP

  ###### #

  启动docker:

  Recreating orderer.example.com ... 
  Recreating orderer.example.com
  Recreating peer0.org1.example.com ... 
  Creating peer1.org1.example.com ... 
  Recreating peer0.org1.example.com
  Creating peer0.org2.example.com ... 
  Creating peer1.org2.example.com ... 
  Creating peer0.org2.example.com
  Creating peer1.org1.example.com
  Recreating peer0.org1.example.com ... done
  Recreating cli ... 
  Recreating cli ... done

  创建channel:

  Channel name : mychannel

  - peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    Creating channel...
  - res=0
  - set +x
    2019-08-21 09:09:35.810 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
    2019-08-21 09:09:36.006 UTC [cli.common] readBlock -> INFO 002 Received block: 0
    ===================== Channel 'mychannel' created ===================== 

  节点加入channel：

  2019-08-21 09:09:36.228 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2019-08-21 09:09:36.369 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  ===================== peer0.org1 joined channel 'mychannel' ===================== 

  2019-08-21 09:09:39.586 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2019-08-21 09:09:39.707 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  ===================== peer1.org1 joined channel 'mychannel' ===================== 

  2019-08-21 09:09:42.895 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2019-08-21 09:09:43.030 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  ===================== peer0.org2 joined channel 'mychannel' ===================== 

  2019-08-21 09:09:46.246 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2019-08-21 09:09:46.388 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  ===================== peer1.org2 joined channel 'mychannel' ===================== 

  更新channel信息：

  2019-08-21 09:09:49.576 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2019-08-21 09:09:49.633 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
  ===================== Anchor peers updated for org 'Org1MSP' on channel 'mychannel' ===================== 

  链码测试：
  Install chaincode on peer0.org2...

  - peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
  - res=0
  - set +x
    2019-08-21 09:09:57.592 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
    2019-08-21 09:09:57.592 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
    2019-08-21 09:09:58.080 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" > 
    ===================== Chaincode is installed on peer0.org2 ===================== 

  Instantiating chaincode on peer0.org2...

  - peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -l golang -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P 'AND ('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'
  ```

  

至此，您的fabric环境就搭好啦, 期间运行到哪个步骤报错就可以按照自己理解去找原因

</details>