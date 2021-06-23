# As Strong As Its Weakest Link: How to Break Blockchain DApps at RPC Service

##  overview 

Blockchain remote procedure call (RPC) services emerge as an  intermediary connecting the DApps to a blockchain network.

Conduct a systematic measurement study on nine real-world  RPC services which control most DApp clients’ connection  to the Ethereum mainnet

Discover the previously unknown behaviors: load balancing and gas limiting; strategies are proposed to evade the protection

The effectiveness of DoERS attacks on deployed  RPC services with minimal service interruption

Propose mitigation techniques against DoERS without  dropping service usability, via unpredictable load balancing,  performance anomaly detection, and others

## Background

### *DApps*

"**decentralized applications running atop operational blockchain systems**"

- decentralized finances (DeFi)
- online gaming
- information-security infrastructures
- ...

### *RPC*

***Remote Procedure Call*** service that translates the clients’ requests to cryptocurrency transactions or
queries to a blockchain P2P network![1](\img\1.png)

#### category

maintained directly by the DApp owner (i.e., an in-house RPC node)

a set of nodes hosted by a third party (i.e., a third-party RPC service)

#### providers

Majority

- nine service providers supporting the Ethereum’s JSON-RPC interface (**evaluated in this work**) 
- blockchain.info  : Bitcoin’s JSON-RPC
- dfuse.io: EOSIO’s Chain API
- greymass.com : EOSIO’s Chain API
- stellar.org: Stellar Horizon

#### shortcoming

less decentralized: one to hundreds of nodes, DoS attack could lead to the collapse of the
whole DApp ecosystem

Harms:

- Bitcoin exchanges
- mining pools
- launched against a victim RPC service: allowing a service competitor to steal customers from the victim

 For instance:

- registering an Ethereum domain; purchasing a CryptoKitty: 

delay others’ bidding transactions and win the auction at an unfairly low price

- hash-time-lock contract (HTLC):

defer the withdrawal of the deposit after the expiration of the lock (so-called grieving [51]) by denying the RPC service used by the withdrawal, thus retaining the deposit.

## DoERS

Menace of DoERS

Measurement and findings

Attacks

Mitigation

the research formulation is presented in **Section III**

measurement of DoERS exploitability among RPC services are presented in **Section IV**

Related works are presented in **Section VII** 

responsible disclosure in **Section VIII** 

conclusion in **Section IX**

### *Threat model*

**an attacker:** send crafted RPC requests to deny the RPC service

**an Ethereum RPC service**: a single Ethereum node or a group of Ethereum nodes behind a frontend infrastructure to accept RPC requests

**a benign client**: send regular RPC requests

### *Description of DoERS*

Constructed based on an exploitable **smart contract** that contains **resource-consuming procedures**

Exploitable functions that aim at **depleting CPU, memory and IO resources**

```javascript
contract DoERS-C {
    
function exhaustCPU(uint256 payload_size1) public returns (bool){
 bytes32 target=0xf...f;
 for (uint256 i=0; i<payload_size1; ++i){
 target = keccak256(abi.encodePacked(target));}
 return true;}
 bytes32[] storage;
    
 function exhaustIO(uint256 payload_size1) public returns(bool){
 for (uint256 j=0; j<payload_size2; ++j) {
 storage.push(0xf...f);}
 return true;}
    
 function exhaustMem(uint256 payload_size3) pure public returns(bool) {
 bytes32[] memory mem = new bytes32[](payload_size3);
 mem[payload_size3-1] = 0xf...f;//"CODECOPY" allocate memory
 return true;}}
```

Executed in two steps: 

1. Client deploys the DoERS-C smart contract to Ethereum.
2. Send one or multiple eth_call RPCs to the victim node to trigger one of the three exhaustXX functions in DoERS-C

Limitation: the configurations of Ethereum nodes may thwart the above basic attack: Ethereum’s Gas
limit, timeout, rate limiting, load balancing, as well as other unknown mechanisms inside the black box RPC services.

## Measurement of DoERS exploitability

### *load balancer*

**Measurement mechanisms**

First, send an orphan transaction with nonce+ 2 and gas price to a target RPC service

Second, send the second orphan transaction, with the same nonce+ 2 but paying gas price−1. If the second transaction fails, no load valancing is detected.

Third, send RPC queries to eth_getTransactionByHash(txHash), after waiting for a time period, sends another one. If both return successfully, no load balancing is detected.

Forth, send getBlockNumber RPC requests, if  anomaly detected (one smaller than last one), load balancing detected.

**Results**

![2](\img\2.png)

- Type i) No load balancing 
- Type ii-a) No load balancing detected when RPC queries are sent with the same API key and from different IPs
- Type ii-b) No load balancing detected when RPC queries are sent from the same IP but with different API keys;
- Type iii)  Comprehensive load balancing detected

**Attack Strategies Evading Load Balancer**

- Type i) straightforward way
- Type ii-a) use same API Keys
- Type ii-b)  set up a token faucet, giving away free M-Tokens, internally calls exhaustXX
- Type iii)  “flash attack” (e.g., one minute for ServiceX6)

### *Gas Limits*

**Measurement mechanisms**

First, the program starts with an initial guess on the target arrayLength value

Then grows the guess exponentially until the first exception is observed.

Second, binary-search the Gas-limit corresponding value of arrayLength, gain target value V

Finally, run estimateGas() with function exhaustMem under V 

```javascript
float rpc_gasLimit(IP rpcNode){
 int lengthLower=0; int lengthUpper=500;//0/500 block gas
 while (lengthUpper - lengthLower > 1){
 arrayLength = (lengthLower + lengthUpper) / 2;
 try{
 rpcNode.eth_call(exhaustMem,arrayLength);
 } (Exception e) {
 if(e instanceOf OutofGasException){
 lengthUpper = arrayLength;
 } else { //no gas limits
 return 0;}
 } else {
 lengthLower = arrayLength;}}
 return localNode.estmateGas(exhaustMem,arrayLength);}
```

**Results**

four out of nine services configure the Gas limit: ServiceX3/ServiceX6/ServiceX9/ServiceX8 respectively set Gas limit at 50/10/5/1.5 block gas

**Attack Strategies Evading Gas Limit**

send multiple DoERS requests at a sufficiently high rate

## **COUNTERMEASURE**

### *Known Countermeasures*

#### Gas Limits

necessary but not sufficient

#### Contract Banning

upload a smart contract at the invocation time of eth_call

### *Proposed Countermeasures*

**The root cause of DoERS** : an **open-membership** RPC service that allows for **free execution** of **arbitrary smart contract programs** on its peers **shared** by different DApps

**Change root condition:** 

- removing open-membership (e.g., by authenticating DApp clients based on their true identities)
- charging the contract execution triggered by eth_call
- limiting the computation expressiveness (e.g.,prohibiting loops)
- avoiding any sharing of a RPC node among DApps
- etc

**Goal:** encounter a fundamental trade-off between DoERS security and service usability

#### Unpredictable yet consistency-preserving load balancing

one balancer handles only transactions while the other forwards only RPC queries:

**load balancer** : nternally maintain a secret true-random number or the current workload that decides the destination backend peer a RPC query should be forwarded to

**transaction-only balancer** : distributes the requests under the constraint of preserving consistency

#### Performance anomaly detection plus security deposit

require security deposit from any potential clients

**Key condition:** the performance monitor can distinguish malicious DoERS requests from benign RPCs

#### Interruptible EVM instructions

**Problem**: atomic EVM instructions: success of single- request DoERS

**Solution**: execution of a single instruction (e.g., CODECOPY in exhaustMem) to be interrupted by timeout

**Specific way**: change EVM’s instruction scheduling algorithm and to enforce the maximal memory size allocated by a single CODECOPY call.

## CONCLUSION

The **first measurement study** on the security of **Ethereum’s RPC-enabled nodes under DoS** attacks.

**No gas limits**: can be crashed by the proposed DoERS attack

**Configured Gas limits**: a properly configured DoERS attack can cause a latency increase by 2.1× ∼ 50×

**mitigation**: beyond simply limits the Gas at zero Ether cost

unpredictable load balancing

performance anomaly detection

interruptible EVM instructions.