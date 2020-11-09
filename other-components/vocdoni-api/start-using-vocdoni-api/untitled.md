# Entity creation



Make sure you have [Node.js](https://nodejs.org/) 12 installed on your computer.

In an empty folder, create a `package.json` file by running

```javascript
$ npm init -y
Wrote to /home/user/dev/governance-as-a-service/package.json:

{
  "name": "governance-as-a-service",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \\\\"Error: no test specified\\\\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Next, install the DVoteJS dependency and two dev dependencies to support TypeScript:

```text
$ npm i dvote-js
$ npm i -D typescript ts-node
```

Edit the `scripts` section of `package.json` and leave it like this:

```javascript
  "scripts": {
    "main": "ts-node index.ts"
  },
```

Then, let's create a file named `index.ts` and create a function to generate the wallet of our new entity:

```typescript
import { Wallet } from "ethers"

function makeEntityWallet() {
    console.log("Creating entity wallet")
    const entityWallet = Wallet.createRandom()

    console.log("Entity address:", entityWallet.address)
    console.log("Entity private key:", entityWallet.privateKey)
}

makeEntityWallet()
```

To check that it works, let's run `npm run main`:

```bash
$ npm run main
Creating entity wallet
Entity address: 0x1048d8cB20fE806389802ba48E2647dc85aB779a
Entity private key: 0x6d88d78a4a76e42144b6867fdff89477f0a3b805d85b97cd46387a2f770f91f1
```

So here's our wallet, what's next?

In this example, we will be using an Ethereum testnet called Sokol. Write down your private key and and use the address to request some test coins from [the faucet](https://faucet-sokol.herokuapp.com/). You will need them to send some transactions.

![Sokol faucet](./assets/image-1.png)

Now, instead of creating a random wallet, we should use the one that received the test ether. In the lines above, replace this:

```typescript
const entityWallet = Wallet.createRandom()
```

Into this:

```typescript
// Use your private key here
const entityWallet = new Wallet("0x6d88d78a4a76e42144b6867fdff89477f0a3b805d85b97cd46387a2f770f91f1")
```

Obviously, make sure to store any real private key nowhere within the source code or Git in general.

Now we need to get a pool of gateways and connect to one of them:

```typescript
import { GatewayPool } from "dvote-js/dist/net/gateway-pool"

let pool: GatewayPool

async function connect() {
    // Get a pool of gateways to connect to the network
    pool = await GatewayPool.discover({ networkId: NETWORK_ID, bootnodesContentUri: GATEWAY_BOOTNODE_URI })
    await pool.connect()
}

const disconnect = () => pool.disconnect()
```

Then, let's define the metadata of our Entity and make it available on IPFS and Ethereum. Add the following code to `index.ts`:

```typescript
import { EntityMetadata } from "dvote-js"
import { updateEntity } from "dvote-js/dist/api/entity"

async function registerEntity() {
    // Make a copy of the metadata template and customize it
    const entityMetadata: EntityMetadata = Object.assign({}, Models.Entity.EntityMetadataTemplate)

    entityMetadata.name.default = "Vilafourier"
    entityMetadata.description.default = "Official communication and participation channel of the city council"
    entityMetadata.media = {
        avatar: 'https://my-organization.org/logo.png',
        header: 'https://my-organization.org/header.jpeg'
    }
    entityMetadata.actions = []

    const contentUri = await updateEntity(entityWallet.address, entityMetadata, entityWallet, pool)

    // Show stored values
    console.log("The entity has been defined")
    console.log(contentUri)
}
```

And then:

```bash
$ npm run main
Setting the entity metadata
WARNING: Multiple definitions for addr
WARNING: Multiple definitions for setAddr
The entity has been defined
ipfs://QmdK5TnHDXPt4xozkuboyKP94RTrUxFr1z9Pkv5qhfahFG
```

Done!

This is what we just did:

* We created JSON metadata containing the custom details of our entity
* We pinned the JSON content on IPFS using a gateway from the pool
* The hash of our metadata is `QmdK5TnHDXPt4xozkuboyKP94RTrUxFr1z9Pkv5qhfahFG` and should be available [from any IPFS peer](https://ipfs.io/ipfs/QmdK5TnHDXPt4xozkuboyKP94RTrUxFr1z9Pkv5qhfahFG)
* An Ethereum transaction was sent to the entities contract, defining the pointer to our new metadata.

The value on the smart contract can only be updated by our wallet, the blockchain ensures the integrity of our data and IPFS ensures its global availability.

**Visualizer**

To check that our entity is properly declared, we can check it on the visualizer: `https://app.dev.vocdoni.net/entities/#/<entity-id>`

This is the link [in our case](https://app.dev.vocdoni.net/entities/#/0xdeea56f124dd3e73bcbc210fc382154a11f3bab227af55927139c887e15573e4).

> Note: Keep in mind that we're using a testnet and some of the data might be eventually disposed.

