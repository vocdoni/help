# Census generation

Our entity is ready, now it can handle voting processes. But before we can create one, we need a census of the people who can vote.

As we said, digital identities arise from a cryptographic keypair. The ideal scenario is when the identity is generated on the user's device and only the public key is shared, but for the sake of simplicity, we will create a few identities ourselves.

```typescript
let votersPublicKeys: string[] = []

function populateCensusPublicKeys() {
    for (let i = 0; i < 10; i++) {
        const wallet = Wallet.createRandom()
        votersPublicKeys.push(wallet["signingKey"].publicKey)
    }
    console.log("Voter's public keys", votersPublicKeys)
}
```

Which results in

```text
Voter's public keys [
 '0x046f5ae433845ddc5d93a44b4aa3b9f654861372ada5f074b88bd18623dcbac5e4c3495f2f4f96e9053ce89f8befd3ad5424224948b84ba5a9292ef3f594003c46',
 '0x0460aaecd59d68dc63aa173f77d89476280d77b67c5793c5f166ff951eb4d761df8d57eab31e8064180027386ead992b74bd097461e0a44e80b371650f4c09370b',
  ...
]
```

Then we can upload the census claims to a gateway and publish it:

```typescript
import { addCensus, addClaimBulk, getRoot, publishCensus } from "dvote-js/dist/api/census"

let merkleTreeOrigin: string
let merkleRoot: string

async function publishVoteCensus() {
    // Prepare the census parameters
    const censusName = "Vilafourier all members #" + Math.random().toString().substr(2, 6)
    const adminPublicKeys = [await entityWallet["signingKey"].publicKey]
    const publicKeyClaims = votersPublicKeys.map(k => digestHexClaim(k)) // hash the keys

    // As the census does not exist yet, we create it (optional when it exists)
    let { censusId } = await addCensus(censusName, adminPublicKeys, pool, entityWallet)
    console.log(`Census added: "${censusName}" with ID ${censusId}`)

    // Add claims to the new census
    let result = await addClaimBulk(censusId, publicKeyClaims, true, pool, entityWallet)
    console.log("Added", votersPublicKeys.length, "claims to", censusId)
    if (result.invalidClaims.length > 0) console.error("Invalid claims", result.invalidClaims)

    merkleRoot = await getRoot(censusId, pool)
    console.log("Census Merkle Root", merkleRoot)

    // Make it available publicly
    merkleTreeOrigin = await publishCensus(censusId, pool, entityWallet)
    console.log("Census published on", merkleTreeOrigin)
}
```

Which results in:

```text
Census added: "Vilafourier all members #550262" with ID 0x1048d8cB20fE806389802ba48E2647dc85aB779a/0x8e0c00cda17a132e2ce6c89072cc2037b0800c280c1ddba318eb2620fae2ff16
Added 10 claims to 0x1048d8cB20fE806389802ba48E2647dc85aB779a/0x8e0c00cda17a132e2ce6c89072cc2037b0800c280c1ddba318eb2620fae2ff16
Census Merkle Root 0xbf8572159929b4e37c40e2ce18fd46156e750a4aac1f06dbda3237edccc81115
Census published on ipfs://QmcTuUJbN1tEWA3nqHFU8N8iuUN2ymJoqW4UcKBzHuYrPw
```

Cool! Our entity is ready, and our first census is now public.

