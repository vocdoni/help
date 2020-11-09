# Process results

If we look at the visualizer or the block explorer we should see votes already submitted.

If the process type was set to `encrypted-poll` then, results would remain unavailable until the process ends. As our process is an open poll, however, we can ask for the results in real time:

```typescript
import { getResultsDigest } from "dvote-js/dist/api/vote"

async function fetchResults() {
    const { questions } = await getResultsDigest(processId, pool)

    console.log("Process results", questions)
}
```

After all users have voted, the visualizer shows:

![Process results](./assets/image-2.png)

However, if you can't wait for an encrypted process to end later or you simply want to stop vote submissions, there's a trick you can use. This will change in the near future, but for now you can `cancel` the rest of the lifespan of a process, if needed. The scrutiny will begin a few seconds later.

```typescript
import { isCanceled, cancelProcess } from "dvote-js/dist/api/vote"

async function forceEndingProcess() {
    // Already canceled?
    const canceled = await isCanceled(processId, pool)
    if (canceled) return console.log("Process already canceled")

    // Already ended?
    const processMeta = await getVoteMetadata(processId, pool)
    const currentBlock = await getBlockHeight(pool)
    if (currentBlock >= (processMeta.startBlock + processMeta.numberOfBlocks)) return console.log("Process already ended")

    console.log("Canceling process", processId)
    await cancelProcess(processId, entityWallet, pool)
    console.log("Done")
}
```

Congratulations! You just conducted an election on your own 🎉🎊

A recap of the examples featured In the article are available on this repository:

[https://github.com/vocdoni/tutorials/tree/master/governance-as-a-service](https://github.com/vocdoni/tutorials/tree/master/governance-as-a-service)

