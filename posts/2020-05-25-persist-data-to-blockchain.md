---
layout: post
title:  "Persist data to the Ethereum blockchain using Python, Truffle and Ganache"
categories: blockchain ethereum python smartcontract
---
This post is also available on [dev.to](https://dev.to/gcrsaldanha/persist-data-to-the-ethereum-blockchain-using-python-truffle-and-ganache-47lb)

---
The [previous post](https://dev.to/gcrsaldanha/deploy-a-smart-contract-on-ethereum-with-python-truffle-and-web3py-5on) demonstrated how to write a simple smart contract with Solidity and deploy it to the Ethereum Blockchain.
This tutorial will show how to update the contract to save some data in the blockchain as well as how to inspect the blockchain to see our transactions.

We'll work on the same files from the previous tutorial, [which can be found here](https://github.com/gcrsaldanha/hello-eth/tree/v1.0).

If not in the mood of going along with the tutorial, the finished code is [available here](https://github.com/gcrsaldanha/hello-eth/tree/v2.0).

### TOC
  1. [Updating our contract](#1-updating-our-contract)
  2. [Executing a transaction](#2-executing-a-transaction)
  3. [Inspecting the blockchain with Ganache](#3-inspecting-the-blockchain-with-ganache)

---

## 1. Project setup <a name="1-updating-our-contract"></a>

Open `HelloWorld.sol` and add the following:
```diff
 pragma solidity >= 0.5.0;

 contract HelloWorld {
+    string public payload;
+
+    function setPayload(string memory content) public {
+        payload = content;
+    }
+
     function sayHello() public pure returns (string memory) {
         return 'Hello World!';
     }
 }
```
As we did before, spin up `truffle` local blockchain with `truffle develop` CLI:
```bash
$ truffle develop
Truffle Develop started at http://127.0.0.1:9545/

Accounts:
(0) 0x55660bb0918f9aade4db7b7ef7f71639750c2262
...
Private Keys:
(0) ead0d6b318832f278b9a2ee1a6a6ab0156ab439d898394b6887d023bc929159c
...
$ truffle(develop)>
```
Now, we need to **update** our contract, what under Truffle's Suite is performed with the **migrate** command. Write `migrate` within Truffle's console.
```bash
Compiling your contracts...
===========================
> Compiling ./contracts/HelloWorld.sol
> Compiling ./contracts/Migrations.sol
...
2_deploy_contracts.js
=====================

   Replacing 'HelloWorld'
   ----------------------
   > transaction hash:    0x6f55fc4a165225c6911d6e0acda8b4ec75b0962370eab66ab94aefd0e0cfd084
   > Blocks: 0            Seconds: 0
   > contract address:    0x85534E0ec31b63d635CcBa1fb6b953a9C47a9022
...
```
The contract is now updated with a public method `setPayload` deployed at `0x85534E0ec31b63d635CcBa1fb6b953a9C47a9022` (contract address). Write this address somewhere else, we will need it soon.
We can verify that the contract was properly updated by fetching the contract instance and checking for `setPayload` method existence, as well as the value of `payload` variable:
```bash
# We're still inside Truffle's console
$ truffle(develop)> let instance = await HelloWorld.deployed()
$ truffle(develop)> instance.setPayload
[Function] {
  call: [Function],
  sendTransaction: [Function],
  estimateGas: [Function],
  request: [Function]
}
$ truffle(develop)> instance.payload.call()
''  # Empty, as expected!
```

## 2. Executing a transaction <a name="2-executing-a-transaction"></a>

When we `call` a function that only  returns a value, such as `sayHello`, the blockchain state is not altered. Since now we want to save a value to the blockchain, its state must be altered and it is done via a `transaction` execution.
We want to execute this transaction from our `Python` script, the same we used for calling the `sayHello` method in the previous post.
Add the following to the bottom of `app.py`:
```python
# executes setPayload function
tx_hash = contract.functions.setPayload('abc').transact()
# waits for the specified transaction (tx_hash) to be confirmed
# (included in a mined block)
tx_receipt = web3.eth.waitForTransactionReceipt(tx_hash)
print('tx_hash: {}'.format(tx_hash.hex()))
```
Also, update the `deployed_contract_address` in `app.py` with the new contract address (output from `migrate`). If you don't remember it, go back to the Truffle console and call `instance.address`.

On another terminal session, execute our Python script:
```bash
python app.py
Hello World!
tx_hash: 0x2b7bf989fd0fc692d0ef976b3293c39958ab7de64a08a3f66c0d758b871ad8e6
```
Not fun, right? How do we know that the data was persisted? Go back to the Truffle console and call `payload` variable:
```bash
$ truffle(develop)> instance.payload.call()
'abc'  # cool! It changed!
```
This means that our transaction was **mined**, thus being added to a block that was successfully added to the blockchain.

## 3. Inspecting the blockchain with Ganache<a name="3-inspecting-the-blockchain-with-ganache"></a>

There are various ways for inspecting the blockchain. For example, using the Truffle console:
```bash
# Fetch blockchain's latest mined block
$ truffle(develop)> block = await web3.eth.getBlock("latest")
$ truffle(develop)> block
{
  number: 6,  # YMMV
  (...)
  transactions: [
    '0x2b7bf989fd0fc692d0ef976b3293c39958ab7de64a08a3f66c0d758b871ad8e6'
  ]
}
```
`transactions` lists all the transactions that were added to this block. In this case, there is one transaction that should match `tx_hash` that was printed when we ran `app.py`.

To make our lives easier, there is a GUI tool for inspecting the blockchain and its transactions: [Ganache](https://www.trufflesuite.com/ganache). As per their website:
> Quickly fire up a personal Ethereum blockchain which you can use to run tests, execute commands, and inspect state while controlling how the chain operates.

After successfully installing and opening it, you should see the following image:
![Ganache initial screen](https://dev-to-uploads.s3.amazonaws.com/i/ifjel8i76vgwo3whrzzw.png)
Select "Quickstart" and you'll be welcomed with a screen listing some accounts and its respective addresses (very similar to how `truffle develop` works). There will be also a `RPC Server` address, such as `http://127.0.0.1:8545`, write that down. Click "Save" (top-right corner) so this Ganache workspace is persisted.

When quick-starting Ganache, it created another local blockchain for us. We can now tell `truffle` to use Ganache's blockchain (which has a nice UI). To do so, open `truffle-config.js` and replace it with the following:
```javascript
module.exports = {
  networks: {
    development: {
      // host and port should match the RPC Server address
      // as seen in Ganache
      host: "127.0.0.1",
      port: 8545,
      network_id: "*"
    }
  }
};
```

Go to the Contracts tab, and select `Link Truffle Projects`. Click `Add Project` then select the `truffle-config.js` file from `hello-eth/` root project folder. Click `Save and Restart` at the upper-right corner.
![Add Project screen of Ganache](https://dev-to-uploads.s3.amazonaws.com/i/ceksfiejbzmcf9a04ry1.png)

If you go to the Contracts tab now, you should see two contracts:
* HelloWorld: Not Deployed
* Migrations: Not Deployed

Open another terminal session, go to the project root level (where `truffle-config.js` is located), and type `truffle migrate`. It will deploy our contracts to Ganache's blockchain. Copy the `HelloWorld` contract address and update the `deployed_contract_address` in `app.py` with its new value, as well as the variable `blockchain_address` with `http://127.0.0.1:8545`.
```diff
- blockchain_address = 'http://127.0.0.1:9545'  # truffle blockchain
+ blockchain_address = 'http://127.0.0.1:8545'  # ganache blockchain
- deployed_contract_address = '0x85534E0ec31b63d635CcBa1fb6b953a9C47a9022'
+ deployed_contract_address = '0x433D7E91B659CB4f1bbBb8dacd2B1cBBa7F82F1A'
```

Under Contracts tab now, both contracts (`HelloWorld` and `Migrations`) should have an address. It means they're deployed! woohoo!
Now, run `app.py` again to execute a new transaction on our newest blockchain and then click on `HelloWorld` contract:
![HelloWorld contract showing transactions](https://dev-to-uploads.s3.amazonaws.com/i/rybnj2vyjbwa6ahzb55z.png)
*Notice the `TX HASH`, it should be the same as printed when you ran `app.py`.*

And if you click on the transaction itself:
![Showing transaction information](https://dev-to-uploads.s3.amazonaws.com/i/r7rp44rvlagcmftw7hy5.png)
*You can see both the function that was called and the inputs.*

Run `app.py` a few more times, change `abc` (in `tx_hash = contract.functions.setPayload('abc').transact()`) to something else, then inspect your contract again. It's YOUR blockchain now, act up to it!

## Wrapping up

In this tutorial, you learned how to:

* Write a method that changes the blockchain state;
* Use Truffle CLI to interact with a deployed smart contract;
* Use Ganache app both as the blockchain provider as well as its inspector;

---

If something is not clear or if there are any suggestions, please leave a comment below! As usual, you can always reach out to me on [Telegram](http://t.me/gcrsaldanha)
