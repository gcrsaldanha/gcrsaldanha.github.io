---
layout: post
title:  "Deploy a Smart Contract on Ethereum Blockchain with Python, Truffle and web3py"
categories: blockchain ethereum python smartcontract
---
This post is also available on [dev.to](https://dev.to/gcrsaldanha/deploy-a-smart-contract-on-ethereum-with-python-truffle-and-web3py-5on)

---
In this tutorial, we'll write a simple smart contract, deploy it to a personal Ethereum blockchain, and call the contract from a Python script.

What you need to have installed before we proceed:
- Python3 v3.5.3 or later, I had some issues using version 3.8 then switched to 3.5.3;
- NodeJS v8.9.4 or later (for installing `truffle`);

The full project code is available at [GitHub](https://github.com/gcrsaldanha/hello-eth)

### TOC
  1. [Project Setup](#1-project-setup)
  2. [Writing the smart contract](#2-writing-the-smart-contract)
  3. [Deploying smart contract to the blockchain](#3-deploying-smart-contract-to-the-blockchain)
  4. [Calling the deployed contract](#4-calling-the-deployed-contract)

---

## 1. Project setup <a name="1-project-setup"></a>

First, let's create a folder for the project:
```bash
$ mkdir hello-eth
$ cd hello-eth
```
Inside `hello-eth` folder, install `truffle`:
```bash
npm install truffle
...
# You should something like this once the installation process is finished
+ truffle@5.1.25
```
We will use the `truffle` CLI tool to initialize an empty smart contract project:
```bash
$ ./node_modules/.bin/truffle init
# if truffle was installed globally (npm install -g truffle): $ truffle init
```
The above command will create the following project structure:
* `contracts/`: Directory for Solidity contracts source code (`.sol` files).
* `migrations/`: Directory for contracts migration files.
* `test/`: Directory for test files. Won't be covered in this tutorial.
* `truffle.js`: Truffle configuration file.

`contracts/` and `migrations/` folders will already contain a `Migration` contract and its deploy script (`1_initial_migration.js`). This contract is used by truffle to keep track of the migrations of our contracts. It won't be covered in this tutorial, so don't worry about it.


## 2. Writing the smart contract <a name="2-writing-the-smart-contract"></a>

Yes, I'll use the `Hello World` example. Create a file called `HelloWorld.sol` inside `contracts/` folder and add the following content to it:
```java
pragma solidity >= 0.5.0 < 0.7.0;

contract HelloWorld {
    function sayHello() public pure returns (string memory) {
        return 'Hello World!';
    }
}
```

From the project root folder, use `truffle` to compile the contract:
```bash
$ ./node_modules/.bin/truffle compile  # or just truffle compile if installed globally...
```
It will output the compiled contract `HelloWorld.sol` as `HelloWorld.json` inside `build/contracts/` folder.

We now have a compiled contract and are ready to deploy it to our running to the (local) blockchain.


## 3. Deploying smart contract to the blockchain <a name="3-deploying-smart-contract-to-the-blockchain"></a>

To deploy our smart contract to the blockchain we first need:
1. a migration script;
2. a blockchain to deploy the contract to;

For the migration script, create a file named `2_deploy_contract.js` inside `migrations/` folder and add the following content to it:
```javascript
var HelloWorld = artifacts.require("HelloWorld");

module.exports = function(deployer) {
    deployer.deploy(HelloWorld);
};
```

Truffle suite contains a personal blockchain that we can use for testing purposes. Open your terminal and run:
```bash
$ truffle develop
```
You should see an output similar to the following:
```bash
Truffle Develop started at http://127.0.0.1:9545/

Accounts:
(0) 0xdfb772fba7631b5bfde93cc3e2b0e488d1a17b2a
...
(9) 0x974779d6a98264043e8bb1c8b0cf93d9c7141a29

Private Keys:
...

truffle(develop)>
```
We can now `migrate` our contract, which calls each migration in `migrations/` (in order), deploying the contracts to the blockchain.
```bash
truffle(develop)> migrate
Starting migrations...
...
2_deploy_contracts.js
=====================

   Deploying 'HelloWorld'
   ----------------------
   > transaction hash:    0xddc3dd045a7b6f70063303ff534d07e93c1dcb7a0a6c15e42fe281c7d2ab53e8
   > Blocks: 0            Seconds: 0
   > contract address:    0xb7afC8dB8EEf302cd30553B39cEa0599093FDE3C
```
`contract address` is the address on the Ethereum network that hosts this contract instance.
Sweet! We now have a Smart Contract deployed on our personal Ethereum blockchain. Next step is call it from a Python script.


## 4. Calling the deployed contract <a name="4-calling-the-deployed-contract"></a>

Under the project root folder (`hello-eth/`) create a file named `app.py` with the following content (pay attention to the comments explaining the script):
```python
import json
from web3 import Web3, HTTPProvider

# truffle development blockchain address
blockchain_address = 'http://127.0.0.1:9545'
# Client instance to interact with the blockchain
web3 = Web3(HTTPProvider(blockchain_address))
# Set the default account (so we don't need to set the "from" for every transaction call)
web3.eth.defaultAccount = web3.eth.accounts[0]

# Path to the compiled contract JSON file
compiled_contract_path = 'build/contracts/HelloWorld.json'
# Deployed contract address (see `migrate` command output: `contract address`)
deployed_contract_address = '0xb7afC8dB8EEf302cd30553B39cEa0599093FDE3C'

with open(compiled_contract_path) as file:
    contract_json = json.load(file)  # load contract info as JSON
    contract_abi = contract_json['abi']  # fetch contract's abi - necessary to call its functions

# Fetch deployed contract reference
contract = web3.eth.contract(address=deployed_contract_address, abi=contract_abi)

# Call contract function (this is not persisted to the blockchain)
message = contract.functions.sayHello().call()

print(message)
```

Now, execute the following commands in your terminal:
```bash
$ pip3 install web3
$ python3 app.py
> 'Hello World!'
```

## Wrapping up
In this tutorial, you learned how to:
- Write a simple Smart Contract in Solidity;
- Create a personal Ethereum Blockchain for tests and development;
- Deploy a contract to the blockchain using `truffle`;
- Call the contract function from a Python application.

---

I will write two follow-up posts in the next few days:
- how to persist data to the blockchain via a Web API using Flask;
- how to use Ganache as the personal blockchain provider (which has a nice GUI to inspect transactions);

If you'd like to be notified when that comes up, make sure to follow me on [Dev.to](https://dev.to/gcrsaldanha))