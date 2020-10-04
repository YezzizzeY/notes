---
title: fabric-node-sdk开发初步
date: 2019-08-21 17:26:41
tags: hyperledger fabric
---



<details>
<summary>详细信息</summary>



###### fabric-node-sdk适用于轻量级web应用实践，步骤如下

###### 1.样例项目:fabcar

目录fabric-samples/fabcar有个startFabric.sh，启动一下直接环境，组网，链码都搭好了
其中重要的部分拿出来解释一下,经过一系列更改就可以自己用作启动脚本了：

```
if [ "$CC_SRC_LANGUAGE" = "go" -o "$CC_SRC_LANGUAGE" = "golang"  ]; then
CC_RUNTIME_LANGUAGE=golang
CC_SRC_PATH=github.com/fabcar
elif [ "$CC_SRC_LANGUAGE" = "javascript" ]; then
CC_RUNTIME_LANGUAGE=node # chaincode runtime language is node.js
CC_SRC_PATH=/opt/gopath/src/github.com/fabcar/javascript
elif [ "$CC_SRC_LANGUAGE" = "typescript" ]; then
CC_RUNTIME_LANGUAGE=node # chaincode runtime language is node.js
CC_SRC_PATH=/opt/gopath/src/github.com/fabcar/typescript
echo Compiling TypeScript code into JavaScript ...CC_SRC_PATH=github.com/fabcar     
```



//设置链码位置和sdk语言，后面两个elif用go开发就写死就行，可以直接去掉

```
cd ../basic-network
./start.sh  
```


​                         
//启动5个docker容器：ca，orderer，peer，couchdb,cli并利用basic-network里的配置文件启动一个简单的网络,一个channel命名为mychannel，加入peer到网络

```
docker-compose -f ./docker-compose.yml up -d cli
```



//启动cli

```
docker ps -a


```

//显示已经启动的docker信息

```
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" cli peer chaincode install -n fabcar -v 1.0 -p "$CC_SRC_PATH" -l "$CC_RUNTIME_LANGUAGE"
```


//链码安装

```
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" cli peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n fabcar -l "$CC_RUNTIME_LANGUAGE" -v 1.0 -c '{"Args":[]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```


//链码实例化

```
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" cli peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n credit -c '{"function":"initLedger","Args":[]}'
```




链码调用：初始化账本之后在javascript文件夹下运行npm install之后就可以直接node *.js来调用链码了




2.devmode开发模式下测试 (前提已经写好链码):

进入/fabric-samples/chaincode-docker-devmode

(1)在终端1启动网络

```
docker-compose -f docker-compose-simple.yaml up
```


(2)在终端2编译和启动链码

这步是实际是在节点注册启动了。
进入chaincode容器：

```
docker exec -it chaincode bash
cd {pkgname}
go build
```


启动链码

```
CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME={channelname}:0 ./{chaincode}
```


(3)终端3中使用链码

安装和初始化链码在devmode是有点多余的,后面版本可能会删除。
进入cli容器：

```
docker exec -it cli bash
```


安装

```
peer chaincode install -p chaincodedev/chaincode/{pkgname} -n {chaincodename} -v {version}
```


初始化

```
peer chaincode instantiate -n {chaincodename} -v {version} -c ‘{“Args”:[]}’ -C {channelname}
```


修改a的值

```
peer chaincode invoke -n {chaincodename} -c ‘{“Args”:[]}’ -C {channelname}
```


验证查询a的值

```
peer chaincode query -n {chaincodename} -c ‘{“Args”:[]}’ -C {channelname}
```


(4)使用nodejs调用链码(nodejs实际和容器通信)：

示例：queryall.js，一般就只需要在这个基础上稍加修改就能实现大部分简单功能调用，用nodejs或者别的语言调用这个nodejs，可以搭建一个简单的web服务器

- ```
  /*
  
  - SPDX-License-Identifier: Apache-2.0
    */
    'use strict';
    const { FileSystemWallet, Gateway } = require('fabric-network');
    const fs = require('fs');
    const path = require('path');
    const ccpPath = path.resolve(__dirname, '..', '..', 'basic-network', 'connection.json');
    const ccpJSON = fs.readFileSync(ccpPath, 'utf8');
  const ccp = JSON.parse(ccpJSON);
    async function main() {
    try {
  
    const walletPath = path.join(process.cwd(), 'wallet');
    const wallet = new FileSystemWallet(walletPath);
  console.log(`Wallet path: ${walletPath}`);
    const userExists = await wallet.exists('user1');
    if (!userExists) {
  
  console.log('An identity for the user "user1" does not exist in the wallet');
    console.log('Run the registerUser.js application before retrying');
    return;
  
    }
    const gateway = new Gateway();
    await gateway.connect(ccp, { wallet, identity: 'user1', discovery: { enabled: false } });
  const network = await gateway.getNetwork('mychannel');  ---->channel名称
    const contract = network.getContract('student');  ---->链码名称
  console.log(`Transaction has been evaluated, result is: ${result.toString()}`);
  
  await gateway.disconnect();
  
    } catch (error) {

    console.error(`Failed to submit transaction: ${error}`);
    process.exit(1);
  
    }
    }
    main();
  ```
  
  </details>





