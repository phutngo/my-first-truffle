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

[Why migrate.sol](https://medium.com/@blockchain101/demystifying-truffle-migrate-21afbcdf3264)

seems like a weird  model to force us to deploy an extra smart contract

## Interacting with deployed Smart Contract

Ultimately, there’s a number of ways in which you can interact with on-chain contracts (web3js ethersjs), but for the sake of ease in this instance we’ll be doing it directly from the Truffle console.

Let’s create an instance of our deployed contract via the following. Note that behind the scenes, Truffle is referencing the build artifacts (which in turn store the aforementioned contract address), that's why this is an async Javascript call.

```
truffle(develop)> let storage = await SimpleStorage.deployed()
```

You can now interact via the returned **storage** object, for example. Let's do that now, calling the set contract method, writing a new value in the contract state. As you’ll see, given this invocation results in a change of on-chain state, we actually get a transaction “receipt” returned.

```
truffle(develop)> storage.set(42)

```

Run the following to get the originally stored number. 
```
truffle(develop)> (await storage.get()).toNumber()
```

## Ganache GUI

 1. Ganache GUI starts it’s own chain instance on port 7545
 2. Note that by default truffle develop starts on 9545 
 3. ganache-cli on 8545

### Connect Ganache to Truffle
1. Open Ganache GUI
2. Click New Workspace
3. Click Add Project and add the truffle-config.js file.

### Migrate/Deploy contracts to Ganache GUI

Next up we’re going to migrate our contracts (with a few twists) to the chain instance instantiated by Ganache UI on port 7545. This will give us a great way to visually inspect what's happening not only on our testnet, but also with the contract itself, as you'll see in a moment.

Before we can migrate, we’ll need to update our truffle-config.js file to include the new network as a destination. Because we used truffle init to create our project, it handily includes a number of commented destinations under the networks entry. As such you’ll be able to scroll down and uncomment (currently lines 45-49 at the time of writing). Note that we have to change port to 7545

```
  development: {
  host: "127.0.0.1",
  port: 7545,
  network_id: "*",
  },
```

Awesome, we now have a new network we can migrate to! For reference, this same principle applies when migrating to public networks (such as testnets or mainnet; the Ethereum of equivalent of staging and production environments).

Go ahead and run the following, noting the use of the --network flag that allows us to specify a given network that we want to target.

```
truffle migrate --network development
```

Checkout Ganache GUI to see the 2 new deployed contracts (Migrations and SimpleStorage)

### Update the Smart Contract with events

Last, let’s update our contract to include an event that is emitted every time a new value is set (we'll learn about events in more detail later in this section). Copy and paste the following over your existing SimpleStorage.sol.

```
// SPDX-License-Identifier: MIT
pragma solidity >=0.4.21 <0.7.0;

contract SimpleStorage {
  uint storedData;

  event setEvent(uint newValue);

  function set(uint x) public {
    storedData = x;
    emit setEvent(x);
  }

  function get() public view returns (uint) {
    return storedData;
  }
}
```

We’ll now make use of the --reset flag when re-running the migration command to forcibly replace the contract. Note that this will result in a new contract address.

```
truffle migrate --network development --reset
```

Let’s now jump back into the Truffle console, this time using the console command (vs develop which also spins up a ganache instance, which would be redundant this time).

open new terminal then

```
truffle console --network development
```

Like earlier, we can now send the following to set the value within our SimpleStorage.

```
let contract = await SimpleStorage.deployed()
contract.set(888)
```

If all has been successful, you’ll now see both a reference to setEvent in the logged output. In addition, you’ll also be able to navigate to the events tab within Ganache UI and also see it there.