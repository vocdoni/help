# Voting



The previous code typically runs on the backend of your organization -you will simply choose to generate a census based on your specific organization's parameters.

This next portion, however, is meant to be executed on the voter's device. The specific method used to create and manage user keys is an important decision which presents a trade off between usability and security.

* Users create the keys on their end
  * This allows for a self sovereign identity and provides higher privacy
  * However, users have to register the public key on your backend and you need to validate it
* The organization creates one-time keys
  * The keys are not in the user's control
  * They don't need to register
  * They can simply receive a link and use it to vote, and then the key expires

Whatever your case is, we will illustrate a web browser environment where the voter fetches the process metadata, signs his/her choices and submits them to a gateway.

```typescript
import { Wallet } from "ethers"
import {
    getVoteMetadata,
    estimateDateAtBlock,
    getBlockHeight,
    getEnvelopeHeight,
    packagePollEnvelope,
    submitEnvelope
} from "dvote-js/dist/api/vote"
import { getCensusSize, digestHexClaim, generateProof } from "dvote-js/dist/api/census"
import { waitUntilVochainBlock } from "dvote-js/dist/util/waiters"

async function submitSingleVote() {
    // Get the user private key from the appropriate place
    const wallet = new Wallet("voter-privkey")  // TODO: Adapt to your case

    // Fetch the metadata
    const processMeta = await getVoteMetadata(processId, pool)

    console.log("- Starting:", await estimateDateAtBlock(processMeta.startBlock, pool))
    console.log("- Ending:", await estimateDateAtBlock(processMeta.startBlock + processMeta.numberOfBlocks, pool))
    console.log("- Census size:", await getCensusSize(processMeta.census.merkleRoot, pool))
    console.log("- Current block:", await getBlockHeight(pool))
    console.log("- Current votes:", await getEnvelopeHeight(processId, pool))

    await waitUntilVochainBlock(processMeta.startBlock, pool, { verbose: true })

    console.log("Submitting vote envelopes")

    // Hash the voter's public key
    const publicKeyHash = digestHexClaim(wallet["signingKey"].publicKey)

    // Generate the census proof
    const merkleProof = await generateProof(processMeta.census.merkleRoot, publicKeyHash, true, pool)

    // Sign the vote envelope with our choices
    const choices = [1, 2]
    const voteEnvelope = await packagePollEnvelope({ votes: choices, merkleProof, processId, walletOrSigner: wallet })

    // If the process had encrypted votes:
    // const voteEnvelope = await packagePollEnvelope({ votes, merkleProof, processId, walletOrSigner: wallet, encryptionPubKeys: ["..."] })

    await submitEnvelope(voteEnvelope, pool)
    console.log("Envelope submitted")
}
```

That's quite a lot so far!

To check for the status of a vote, you can append these extra calls at the end:

```typescript
import { getPollNullifier, getEnvelopeStatus } from "dvote-js/dist/api/vote"

{
    // ...

    // wait 10+ seconds
    await new Promise(resolve => setTimeout(resolve, 1000 * 10))

    // Compute our deterministic nullifier to check the status of our vote
    const nullifier = await getPollNullifier(wallet.address, processId)
    const status = await getEnvelopeStatus(processId, nullifier, pool)

    console.log("- Registered: ", status.registered)
    console.log("- Block: ", status.block)
    console.log("- Date: ", status.date)
}
```

The output:

```text
- Starting: 2020-09-17T15:50:44.000Z
- Ending: 2020-09-18T15:50:44.000Z
- Census size: 10
- Current block: 5
- Current votes: 0
Waiting for Vochain block 35
Now at Vochain block 6
Now at Vochain block 7
[...]
Now at Vochain block 35`

Submitting vote envelopes
Envelope submitted

- Registered:  true
- Block:  36
- Date:  2020-09-17T15:51:06.000Z
```

This is what we just did:

* Initialize the wallet with the voter's private key
* Fetch the vote metadata
* Wait until `startBlock`
* Request a census merkle proof to a random Gateway, proving that the voter's public key matches the census Merkle root
* Package our choices into an envelope signed with the voter private key \(and optionally encrypt it\)
* Submit the envelope using a random Gateway
* Retrieve the status of the vote

