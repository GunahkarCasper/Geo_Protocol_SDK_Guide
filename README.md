Geo Protocol SDK - Step by Step Testnet Guide
This guide explains step by step how to install the Geo SDK generate an ID and register your data on the Geo Testnet Blockchain network.

Stage 1: Preparations
Before writing any code you need to prepare your wallet.

Get a Private Key:
Copy your private key and store it in a safe place.
Do not share this key with anyone!
https://www.geobrowser.io/export-wallet


Get Test Tokens (Faucet):
Copy your wallet address.
Paste your address and request tokens.
https://faucet.conduit.xyz/geo-test-zc16z3tcvf


Stage 2:Installation
Open the terminal and run the following commands to create the project files and install the required packages:
```javascript
npm init -y
npm install @geoprotocol/geo-sdk viem ox
```

Stage 3:SDK Test (Step 1)
First,lets write a simple script to verify that the SDK works and can generate an ID.

!!! Create a file named `step1-test.mjs` and paste the following code into it:
```javascript
import { IdUtils } from "@geoprotocol/geo-sdk";

const myEntityId = IdUtils.generate();
console.log(myEntityId);
```
Run the test:
```javascript
node step1-test.mjs
```
Phase 4: Final Deployment Script (Step 2)
Now we will create the main script that:

1-Connects to your wallet.
2-Checks/Creates a Personal Space.
3-Publishes your profile to IPFS.
4-Submits the transaction to the Blockchain.
!!! Create a file named `deploy.mjs.`

 !!!!! IMPORTANT !!!!!
Replace "PASTE_YOUR_PRIVATE_KEY_HERE" with your actual key.
Replace "Your Name" and description with your own details.
```javascript
import { createPublicClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { Graph, IdUtils, personalSpace, getWalletClient } from "@geoprotocol/geo-sdk";

const addressPrivateKey = "PASTE_YOUR_PRIVATE_KEY_HERE";
const SPACE_REGISTRY_ADDRESS = "0xB01683b2f0d38d43fcD4D9aAB980166988924132";

const SpaceRegistryAbi = [
  {
    type: 'function',
    name: 'addressToSpaceId',
    inputs: [{ name: 'account', type: 'address' }],
    outputs: [{ name: '', type: 'bytes16' }],
    stateMutability: 'view'
  },
  {
    type: 'function',
    name: 'registerSpaceId',
    inputs: [
        { name: '_type', type: 'bytes32' },
        { name: '_version', type: 'bytes' }
    ],
    outputs: [],
    stateMutability: 'nonpayable'
  }
];

const run = async () => {
  try {
    const walletClient = await getWalletClient({
      privateKey: addressPrivateKey,
    });
    
    const account = walletClient.account;
    console.log(account.address);

    const rpcUrl = walletClient.chain?.rpcUrls?.default?.http?.[0];
    const publicClient = createPublicClient({
      transport: http(rpcUrl),
    });

    let spaceIdHex = await publicClient.readContract({
      address: SPACE_REGISTRY_ADDRESS,
      abi: SpaceRegistryAbi,
      functionName: "addressToSpaceId",
      args: [account.address],
    });

    const ZERO_ID = "0x00000000000000000000000000000000";

    if (spaceIdHex === ZERO_ID) {
      const { to, calldata } = personalSpace.createSpace();
      const txHash = await walletClient.sendTransaction({
        account,
        to,
        data: calldata,
      });
      await publicClient.waitForTransactionReceipt({ hash: txHash });
      
      spaceIdHex = await publicClient.readContract({
        address: SPACE_REGISTRY_ADDRESS,
        abi: SpaceRegistryAbi,
        functionName: "addressToSpaceId",
        args: [account.address],
      });
    }

    const spaceId = spaceIdHex.slice(2, 34).toLowerCase();
    console.log(spaceId);

    const myEntityId = IdUtils.generate();
    const entityResult = Graph.createEntity({
      id: myEntityId,
      name: "Your Name",
      description: "Twitter: https://x.com/YourHandle | GitHub: https://github.com/YourHandle",
      types: [],
      values: [],
    });

    const { cid, editId, to, calldata } = await personalSpace.publishEdit({
      name: "User Profile",
      spaceId,
      ops: entityResult.ops,
      author: account.address,
      network: "TESTNET",
    });

    console.log(cid);

    const publishTxHash = await walletClient.sendTransaction({
      account,
      to,
      data: calldata,
    });

    console.log(publishTxHash);

  } catch (error) {
    console.log(error);
  }
};

run();
```
Phase 5:Execution
Run the final script to publish your data on chain:
```javascript
node deploy.mjs
```

Twitter : https://x.com/GunahkarCasper
