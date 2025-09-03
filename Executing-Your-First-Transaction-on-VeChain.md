# Executing Your First Transaction on VeChain

## Introduction
Transactions are the foundation of any blockchain; they move value, trigger smart contracts, and record activity on-chain. On VeChain, transactions are powered by its unique two-token system: VET, which represents value, and VTHO, which covers gas fees. This design makes VeChain efficient and predictable for developers and enterprises alike.

In this guide, we’ll walk through how to execute your very first transaction on the VeChain blockchain. You’ll learn how to set up your environment, create a wallet, fund it with test tokens, write a simple transfer script, and verify the transaction—all with step-by-step instructions and without unnecessary complexity.


### What You Will Need

Before sending your first transaction on VeChain, make sure you have the following:

* Node.js and npm – Installed on your machine to run JavaScript code.

* Code editor – Such as VS Code, to write and run your scripts.

* VeChain testnet account – A wallet address to send and receive tokens. 

* Test VET and VTHO – Faucet tokens to cover transfers and gas fees. You can get some in [Vechain Faucet](https://faucet.vecha.in/)

## Transactions in VeChain

On the VeChainThor blockchain, a transaction is simply an instruction from a user that gets recorded on-chain. Just like on other blockchains, this instruction could transfer tokens between wallets, deploy a new smart contract, or call an existing smart contract function. In short, every meaningful change on VeChain happens through a transaction.

### Transaction Types in VeChain

VeChainThor supports two types of transactions:

1. **Legacy Transactions**

Legacy Transactions are the original transactions that were in use before typed transactions were introduced. In the legacy transaction model, you have to manually choose a gas price, which is how much you’re willing to pay to send a transaction. Any transaction that starts with a byte between `0x7f` and `0xfe` is considered legacy.

3. **Dynamic Fee Transactions (VIP-251)**

Dynamic fee transactions were introduced in VIP-251 and marked with type `0x51`. These use a more flexible fee model, similar to Ethereum’s modern EIP-1559 system. This transaction model doesn't set a fixed gas price. Instead, the blockchain has a built-in base fee that changes automatically. This base fee goes up if the network is busy and goes down if it’s less crowded. The adjustment happens with every new block, based on how much gas was used compared to the target.

You can learn more about these transaction types in [VeChain Documentation](https://docs.vechain.org/core-concepts/transactions/transaction-calculation)

### Key Fields in a Transaction

When you create a transaction on VeChainThor, the body includes several important fields:

* **ChainTag** → Identifies the blockchain (prevents replay attacks across chains).
* **BlockRef** → A reference to a recent block.
* **Expiration** → How long (in blocks) the transaction remains valid before expiring.
* **Clauses** → The most unique part of VeChain transactions. A single transaction can include multiple clauses, and each clause has:

  * **To** → The recipient address (or empty if deploying a contract).
  * **Value** → Amount of VET to send.
  * **Data** → Optional data (e.g., smart contract call).
* **GasPriceCoef** → Used in fee calculation for legacy transactions.
* **Gas** → Maximum gas the transaction is allowed to consume.
* **MaxFeePerGas** → The absolute maximum fee per unit of gas (dynamic fee model).
* **MaxPriorityFeePerGas** → A tip paid directly to block producers (dynamic fee model).
* **DependsOn** → Lets you make a transaction depend on another (it won’t execute unless the other succeeds).
* **Nonce** → A random number to make each transaction unique.
* **Reserved** → Contains:

  * **Features** → A flag (e.g., set to `1` for designated gas payer, VIP-191).
  * **Unused** → Must stay empty (reserved for future use).
 
Now that we understand what transactions are in VeChain, let’s learn how to execute our first on-chain transaction.

### Step 1: Setting Up Your Development Environment

* Installing Node.js
  
VeChain smart contracts are written in Solidity, just like Ethereum, but to interact with the blockchain, sending transactions, reading data, or building apps, you use JavaScript.

JavaScript lets us write scripts and programs on our computer or web apps that talk to VeChain nodes using libraries. Node.js allows you to run JavaScript on your computer, not just in web browsers. To check if it’s already installed, open your terminal in VS Code and type:

```bash
node --version
```
If you see a version number, like `v24.7.0`, you’re all set. If not, go to [nodejs.org](https://nodejs.org/) and download the **LTS version**, then follow the installation instructions.

* Creating Your Project Folder

Before we start writing scripts, let’s create a dedicated folder for our VeChain project. In your terminal, run these commands one by one:

```bash
mkdir my-first-vechain-transaction
cd my-first-vechain-transaction
npm init -y
```
Here’s the explanation of what each command does:

1. `mkdir my-first-vechain-transaction` → Creates a new folder for your project.
2. `cd my-first-vechain-transaction` → Moves you into that folder.
3. `npm init -y` → Initializes the folder as a Node.js project. The `-y` flag automatically accepts all default settings.

After running `npm init -y`, you should see a `package.json` file in your folder. This file keeps track of your project’s settings and dependencies, like the one below.
```bash
{
  "name": "my-first-vechain-transaction",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs",
  "dependencies": {
    "@vechain/sdk-network": "^2.0.4"
  }
}
```
* Installing VeChain Tools

To interact with the VeChain blockchain, we need some specialized tools. Run the following commands in your project folder:

```bash
npm install thor-devkit axios
npm install @vechain/sdk-network
```

Here’s what each package does:

* **thor-devkit** → Thor DevKit is VeChain’s official toolkit. It helps you build, sign, and manage transactions to interact with the VeChainThor blockchain. Users can construct transactions with various clauses and gas settings. (transactions and ABI)

* **axios** → A library for making HTTP requests, which we’ll use to communicate with VeChain nodes.
  
* **@vechain/sdk-network** → Provides network utilities for connecting to VeChain testnet or mainnet and managing nodes.

With these tools installed, your project is ready to start sending transactions on VeChain.

### Step 2: Connecting to the VeChain Network

Before sending your first transaction, we’ll make sure our project can connect to the VeChain test network. The testnet is a safe environment that mimics the main blockchain, but uses test tokens instead of real VET. This way, you can experiment without risking real funds.

Create a new file called `VeChain.js` in your project folder and add the following code:

```javascript
const axios = require('axios');

async function main() {
    // Connect to VeChain's test network
    const url = 'https://testnet.vechain.org';
    
    // Request information about the latest block
    const response = await axios.get(`${url}/blocks/best`);
    
    console.log('Connected to VeChain!');
    console.log('Latest block:', response.data.number);
    console.log('Block ID:', response.data.id);
}

main();
```
What This Code Does

First, we import **axios**, a library that lets our script make HTTP requests to the VeChain test network. Then, we define an asynchronous function called `main()`. Using `async` here is important because the network takes some time to respond, and we don’t want our program to freeze while waiting.

Inside the function, we set the URL for VeChain’s test network (`https://testnet.vechain.org`) and ask the network for information about the latest block using `axios.get()`. If the request succeeds, the script prints a confirmation message, the latest block number, and the block ID.

Run the file in your terminal using the command below:

```bash
node VeChain.js
```

If everything is working, you should see output similar to this:

```
Connected to VeChain!
Latest block: 22680669
Block ID: 0x015a145d1c3afc304fd5e7a4e467afe0c41544de15be5fa93b7083486bfa342f
```

This confirms that your project is successfully communicating with the VeChain test network.
