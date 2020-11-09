# Process creation

Now we are ready to create our fist process. To do so, we will use the values we generated previously and define the topic, the questions, the vote options, etc.

```typescript
import { getEntityId } from "dvote-js/dist/api/entity"
import { estimateBlockAtDateTime, createVotingProcess, getVoteMetadata } from "dvote-js/dist/api/vote"

async function createVotingProcess() {
    const myEntityAddress = await entityWallet.getAddress()
    const myEntityId = getEntityId(myEntityAddress)

    const startBlock = await estimateBlockAtDateTime(new Date(Date.now() + 1000 * 60 * 5), pool)
    const numberOfBlocks = 6 * 60 * 24 // 1 day (10s block time)

    const processMetadata: ProcessMetadata = {
        "version": "1.0",
        "type": "poll-vote",
        "startBlock": startBlock,
        "numberOfBlocks": numberOfBlocks,
        "census": {
            "merkleRoot": merkleRoot,
            "merkleTree": merkleTreeOrigin
        },
        "details": {
            "entityId": myEntityId,
            "title": { "default": "Vilafourier public poll" },
            "description": {
                "default": "This is our test poll using a decentralized blockchain to register votes"
            },
            "headerImage": "https://images.unsplash.com/photo-1600190184658-4c4b088ec92c?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80",
            "streamUrl": "",
            "questions": [
                {
                    "type": "single-choice",
                    "question": { "default": "CEO" },
                    "description": { "default": "Chief Executive Officer" },
                    "voteOptions": [
                        { "title": { "default": "Yellow candidate" }, "value": 0 },
                        { "title": { "default": "Pink candidate" }, "value": 1 },
                        { "title": { "default": "Abstention" }, "value": 2 },
                        { "title": { "default": "White vote" }, "value": 3 }
                    ]
                },
                {
                    "type": "single-choice",
                    "question": { "default": "CFO" },
                    "description": { "default": "Chief Financial Officer" },
                    "voteOptions": [
                        { "title": { "default": "Yellow candidate" }, "value": 0 },
                        { "title": { "default": "Pink candidate" }, "value": 1 },
                        { "title": { "default": "Abstention" }, "value": 2 },
                        { "title": { "default": "White vote" }, "value": 3 }
                    ]
                },
            ]
        }
    }
    const processId = await createVotingProcess(processMetadata, entityWallet, pool)
    console.log("Process created:", processId)

    // Reading the process metadata back
    const metadata = await getVoteMetadata(processId, pool)
    console.log("The metadata is", metadata)
}
```

Which looks like this:

```text
Process created: 0x82f42dc48bfbfa0bb1018bcb0f3a31f77d12ced1fda9566990d64d07be9a48a6
The metadata is {
  version: '1.0',
  type: 'poll-vote',
  startBlock: 2203,
  numberOfBlocks: 8640,
  census: {
    merkleRoot: '0x2044a23e328d75276a7e9e6bc1774eb67707597beba1cfdfde5e7fa174197101',
    merkleTree: 'ipfs://QmTFGgoDnMWhMU3BXxm4zTxeFZLPnRV1jiQgBqukzJPmo9'
  },
  details: {
    entityId: '0xdeea56f124dd3e73bcbc210fc382154a11f3bab227af55927139c887e15573e4',
    title: { default: 'Vilafourier public poll' },
    description: {
      default: 'This is our test poll using a decentralized blockchain to register votes'
    },
    headerImage: 'https://images.unsplash.com/photo-1600190184658-4c4b088ec92c?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80',
    streamUrl: '',
    questions: [ [Object], [Object] ]
  }
}
```

So, we just...

* Defined the metadata of our process
* Defined the  block at which we would like the process to start
* Declare the metadata of the process

As it happened before, the JSON metadata is pinned on IPFS and pointed to from the process smart contract. In addition, some metadata fields are also stored on the smart contract so they can be accessed on-chain.

> The contract flags define **how** the process will behave, whereas the metadata is used to store the **human readable** content.

In about 2-3 minutes, the Ethereum transaction will be relayed to the voting blockchain as well. When the block number reaches `startBlock`, the process will become open to those who are part of the census. The `startBlock` value should be **at least 25 blocks ahead** of the current value.

**Visualizer**

To check the new process, there are two web sites we can use:

* `https://app.dev.vocdoni.net/processes/#/<entity-id>/<process-id>`
* `https://explorer.dev.vocdoni.net/process/<process-id>`

In our case:

* `https://app.dev.vocdoni.net/processes/#/0xdeea56f124dd3e73bcbc210fc382154a11f3bab227af55927139c887e15573e4/0x82f42dc48bfbfa0bb1018bcb0f3a31f77d12ced1fda9566990d64d07be9a48a6`
* `https://explorer.dev.vocdoni.net/process/0x82f42dc48bfbfa0bb1018bcb0f3a31f77d12ced1fda9566990d64d07be9a48a6`

Again, this is a testnet and some of this data may have been cleaned up at some point.

