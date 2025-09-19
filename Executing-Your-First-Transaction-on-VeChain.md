# Executing Your First Transaction on VeChain

## Introduction
Transactions are the foundation of any blockchain; they move value, trigger smart contracts, and record activity on-chain. On VeChain, transactions are powered by its unique two-token system: VET, which represents value, and VTHO, which covers gas fees. This design makes VeChain efficient and predictable for developers and enterprises alike.

In this guide, we’ll walk through how to execute your very first transaction on the VeChain blockchain. You’ll learn how to set up your environment, create a wallet, fund it with test tokens, write a simple transfer script, and verify the transaction, all with step-by-step instructions and without unnecessary complexity.


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
**What This Code Does**

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

### Step 3: Creating Your Digital Wallet

Now it’s time to create a wallet so we can send transactions on the VeChain test network. Create a new file called `transaction.js` in your project folder and paste the following code:

```javascript
const axios = require('axios');
const { Transaction, secp256k1, address } = require('thor-devkit');

async function main() {
    // Your wallet
    const privateKey = Buffer.from('c3cfa011454298cc9114940207561aa31c24aa38875f370b1f8e6231aa0e8598', 'hex');
    const publicKey = secp256k1.derivePublicKey(privateKey);
    const fromAddress = address.fromPublicKey(publicKey);
    
    // Recipient address (sending to a test address)
    const toAddress = '0x7567d83b7b8d80addcb281a71d54fc7b3364ffed';
    
    // Get latest block
    const url = 'https://testnet.vechain.org';
    const blockResponse = await axios.get(`${url}/blocks/best`);
    
    // Build transaction
    const transaction = new Transaction({
        chainTag: 0x27, // Testnet chain tag
        blockRef: blockResponse.data.id.slice(0, 18),
        expiration: 32,
        clauses: [{
            to: toAddress,
            value: '0x8ac7230489e80000', // 10 VET in wei
            data: '0x'
        }],
        gasPriceCoef: 0,
        gas: 21000,
        dependsOn: null,
        nonce: Date.now()
    });
    
    // Sign transaction
    const signingHash = transaction.signingHash();
    const signature = secp256k1.sign(signingHash, privateKey);
    transaction.signature = signature;
    
    // Send transaction
    const rawTx = '0x' + transaction.encode().toString('hex');
    const sendResponse = await axios.post(`${url}/transactions`, { raw: rawTx });
    
    console.log('Transaction sent!');
    console.log('TX ID:', sendResponse.data.id);
    console.log(`View at: https://explore-testnet.vechain.org/transactions/${sendResponse.data.id}`);
}

main();
```
**What This Code Does**

This code does several things all at once: it creates a wallet, builds a transaction, signs it, and sends it to the VeChain test network.

* First, we define a **private key** and derive the corresponding **public key** and **address**. This gives us a wallet we can use to send transactions. The private key is like the key to your house, and the address is like your home’s mailing address—anyone can send tokens to it, but only you can spend them.

* Next, we specify a **recipient address**. In this tutorial, it’s a test address, so we can safely experiment. Before creating the transaction, we fetch the **latest block** from the VeChain test network. This is necessary because each transaction references a recent block to ensure it’s valid and prevents replay attacks.

* We then **build the transaction** using the `Transaction` class from Thor DevKit. Here we define the amount to send, gas settings, and other transaction parameters. After building it, we **sign the transaction** using our private key. Signing proves to the network that the transaction came from the owner of the wallet.

* Finally, we **send the transaction** to the network via an HTTP request. If everything works, the network returns a transaction ID, which we can use to track the transaction on the testnet explorer.

When you run this file with:

```bash
node transaction.js
```
You should see output like this:

```
New Wallet Created!
Address: 0x25c7f54a2bea8d7e32b9611761a088b26faa4e13
Private Key: 0xc3cfa011454298cc9114940207561aa31c24aa38875f370b1f8e6231aa0e8598
Balance: 0x0 wei
```

The balance shows `0x0` (zero in hexadecimal) because this is a new, empty wallet.

> In real applications, you would **never** display or store your private key like this. Think of your private key as the key to your house—anyone with it can take everything inside. For this tutorial, using test money, it’s safe to display. But with real funds, you must keep your private key extremely secure.

Right now, our code creates a **new wallet every time** it runs. In most cases, you’ll want to use the **same wallet** so you can track your balance and transaction history. To do this, we simply save the private key and reuse it in our script.

Update your existing script with the private key your code generated for you, but without the ox prefix. In my Case, it's the code below:

```javascript
const axios = require('axios');
const { secp256k1, address } = require('thor-devkit');

async function main() {
    // Use your saved private key (without the 0x prefix)
    const privateKey = Buffer.from('c3cfa011454298cc9114940207561aa31c24aa38875f370b1f8e6231aa0e8598', 'hex');
    
    // Derive the public key and wallet address from your private key
    const publicKey = secp256k1.derivePublicKey(privateKey);
    const walletAddress = address.fromPublicKey(publicKey);
    
    console.log('Your Wallet:');
    console.log('Address:', walletAddress);
    
    // Check wallet balance
    const url = 'https://testnet.vechain.org';
    const response = await axios.get(`${url}/accounts/${walletAddress}`);
    console.log('Balance:', response.data.balance, 'wei');
}

main();
```

When you run this code, it **always uses the same wallet**, so your address and balance remain consistent across runs.

This makes it easier to track test VET and build more advanced transactions, without creating a new wallet each time.

### Step 4: Getting Test Money from the Faucet

Since we’re working on the VeChain test network, we can get free VET to practice with. Here’s how:

1. **Go to the VeChain testnet faucet:** [https://faucet.vecha.in/](https://faucet.vecha.in/)
2. **Install VeChain Sync2 Wallet** – The faucet will ask you to install it. This wallet is necessary to receive test tokens.
3. **Create a wallet in Sync2** – After installing, create a new wallet and go back to the faucet site to connect your wallet.
4. **Request test VET** – Once connected, you can claim tokens. After the faucet sends them, make sure to send some to the wallet address you generated in your script.

After you receive the test VET, run your balance-checking script again. 

```bash
node transaction.js
```
You should see your balance change from `0x0` to something like `0x1b1ae4d6e2ef500000`, which is **500 VET** in hexadecimal notation.

This confirms that your wallet can now hold test tokens and is ready for sending transactions.

### Step 5: Sending Your First Transaction

Create a new file called `sendtransaction.js` in your project folder and paste the following code:

```javascript
const axios = require('axios');
const { Transaction, secp256k1, address } = require('thor-devkit');

async function main() {
    // Your wallet's private key
    const privateKey = Buffer.from('c3cfa011454298cc9114940207561aa31c24aa38875f370b1f8e6231aa0e8598', 'hex');
    const publicKey = secp256k1.derivePublicKey(privateKey);
    const fromAddress = address.fromPublicKey(publicKey);
    
    // Recipient address (test address)
    const toAddress = '0x7567d83b7b8d80addcb281a71d54fc7b3364ffed';
    
    // Get information about the latest block
    const url = 'https://testnet.vechain.org';
    const blockResponse = await axios.get(`${url}/blocks/best`);
    
    // Build the transaction object
    const transaction = new Transaction({
        chainTag: 0x27,                // Identifies the VeChain testnet
        blockRef: blockResponse.data.id.slice(0, 18),  // Links to a recent block
        expiration: 32,                // Expires after 32 blocks (~5 minutes)
        clauses: [{
            to: toAddress,             // Who receives the VET
            value: '0x8ac7230489e80000',  // 10 VET in hexadecimal wei
            data: '0x'                 // No extra data
        }],
        gasPriceCoef: 0,               // Gas price coefficient (0 for testnet)
        gas: 21000,                     // Maximum gas allowed
        dependsOn: null,                // Independent transaction
        nonce: Date.now()               // Unique number to prevent replay attacks
    });
    
    // Sign the transaction with your private key
    const signingHash = transaction.signingHash();
    const signature = secp256k1.sign(signingHash, privateKey);
    transaction.signature = signature;
    
    // Convert to raw format and send to the network
    const rawTx = '0x' + transaction.encode().toString('hex');
    const sendResponse = await axios.post(`${url}/transactions`, { raw: rawTx });
    
    console.log('Transaction sent!');
    console.log('TX ID:', sendResponse.data.id);
    console.log(`View at: https://explore-testnet.vechain.org/transactions/${sendResponse.data.id}`);
}

main();
```
Let’s break down what each part of the transaction means:

* **chainTag** tells the network which VeChain network we’re using. Think of it like an **area code** for phone numbers.
* **blockRef** links our transaction to a recent block, ensuring it can’t be replayed on a different network.
* **expiration** sets how long the transaction is valid. If it isn’t processed within 32 blocks (about 5 minutes), it becomes invalid—like a **check that expires**.
* **clauses** is where the actual transfer happens. Each clause is an instruction. Here, we’re saying: “send this amount of VET to this address.” The `value` is in **hexadecimal wei**, which computers prefer.
* **gas** is like a delivery fee that pays the network to process the transaction.
* **nonce** is a unique number that prevents someone from copying and replaying your transaction.

When you run this script with:

```bash
node sendtransaction.js
```

You should see output like this:

```
Transaction sent!
TX ID: 0x6a7c24b0253caafe422512d8fd4299da765737281f939034df5d5d72036c2202
View at: https://explore-testnet.vechain.org/transactions/0x6a7c24b0253caafe422512d8fd4299da765737281f939034df5d5d72036c2202
```

Click the **explorer link** to see your transaction on the blockchain!.

### Step 6: Checking Your Final Balance

After sending your first transaction, it’s important to **verify that your balance updated correctly**. Let’s check that 10 VET was sent from your wallet.

Create a new file called `checkBalance.js` and add the following code:

```javascript
const axios = require('axios');

async function checkBalance() {
    const walletAddress = '0x25c7f54a2bea8d7e32b9611761a088b26faa4e13';
    const url = 'https://testnet.vechain.org';
    
    const response = await axios.get(`${url}/accounts/${walletAddress}`);
    const balanceInWei = BigInt(response.data.balance);
    const balanceInVET = Number(balanceInWei) / 1e18;
    
    console.log('Final Balance:', balanceInVET, 'VET');
}

checkBalance();
```

This script connects to the **VeChain test network** and fetches the balance of your wallet. The balance returned by the network is in **wei**, the smallest unit of VET, so we convert it to VET by dividing by `1e18`.

When you run this script with:

```bash
node checkBalance.js
``` 
You should see your **final balance** printed in VET. If your transaction succeeded, your balance should have **decreased by 10 VET**, reflecting the amount sent in your previous transaction.


### Common Issues and Solutions

If your transaction fails, here are the most common reasons:

* Insufficient balance: You need enough VET not just for the amount you're sending, but also for the gas fee. On the testnet, gas is often free, but on the mainnet, you'd need extra VET for fees.
  
* Invalid address or private key: VeChain addresses must be exactly 42 characters long (including the 0x prefix). If you type an address wrong, the transaction will fail.
  
* Expired transaction: If the network is busy and your transaction isn't processed within 32 blocks, it expires. Simply try again with a new transaction.
Wrong network: Make sure you're using the testnet URL for testing. The mainnet URL is different and uses real money.

# Conclusion

Congratulations! You've just completed your first blockchain transaction on VeChain. You've learned how to create a wallet, receive funds, and send them to another address. These fundamental skills are the building blocks for all blockchain applications.

Remember, what we did today with test money works exactly the same way with real VET on the mainnet. The only differences are the network URL and the fact that real money is involved. Always be extra careful with private keys when dealing with real funds.


If you run into any issues while following this guide, feel free to **reach out to me at**: [udogodwin2k22@gmail.com](mailto:udogodwin2k22@gmail.com). I’ll be happy to help!

