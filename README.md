# How to get started using Truffle

## Install Truffle and Ganache

## Init

1. Create a project folder
2. In the folder at command line type
```
truffle init
```

## Create .sol Smart Contract

Command below create a smart contract boilerplate named SimpleStorage and puts the file in the contracts/ folder
```
truffle create contract SimpleStorage
```

## Write Solidity Code
Use this as an example:
```

// SPDX-License-Identifier: MIT
pragma solidity >=0.4.21 <0.7.0;

contract SimpleStorage {
  uint storedData;

  function set(uint x) public {
    storedData = x;
  }

  function get() public view returns (uint) {
    return storedData;
  }
}
```

## Compile
```
truffle compile
```
Assuming all went smoothly, Truffle should have compiled your contract and added the resultant output (something referred to as build artifacts that we’ll explore in more detail later) to build/contracts/SimpleStorage.json.

The .json file contains the abi and the bytecode

## Deploy (Truffle calls this "Migrate")
As alluded to earlier, Ganache comes in a number of different flavors. For ease we’re going to start by using the version built directly into Truffle itself (more on standalone Ganache CLI and Ganache UI shortly).

To achieve this we can use Truffle’s develop command which both starts up a **Ganache instance** and provides us with an **interactive REPL with which we can actually interact with our contracts**.

### Start up Ganache local eth dev blockchain
```
truffle develop
```
This gives us 10 pre-funded Eth Accounts and corresponding 10 Private Keys that we own so we can interact with the Ganache blockchain, and a Mnemonic

### Migrate contract

Before we actually migrate our contract we’ll need to create a migration script. This step enables you to granularly instruct Truffle how to migrate your contracts, including things like constructor arguments.

In the migrations directory create a file called **2_deploy_contracts.js** and copy into that file the following:

```
var SimpleStorage = artifacts.require("./SimpleStorage.sol");

module.exports = function(deployer) {
  deployer.deploy(SimpleStorage);
};
```

The numerical prefix of 2_deploy_contracts.js is actually important for two reasons. 
1. First, it dictates the order in which scripts are executed. 
2. Second, it’s the index stored on-chain by the Migration.sol to keep track of successful migrations per its last_completed_migration value.

Now that we have a migration script ready to go we can migrate as follows. Since we’re doing this migration from the Truffle console we started with truffle develop, you can actually omit truffle from your command and just run:

```
truffle(develop)> migrate
```

TODO how to configure where to deploy say to a testnet.

## Interacting with deployed Smart Contract
