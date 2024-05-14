# 2️⃣ Lachesis aBFT

Lachesis is a breakthrough **aBFT consensus algorithm**.

Below are the key properties of the Lachesis algorithm:

* **Asynchronous**: validators process transactions independently and are not forced to work on the current block being produced.
* **Leaderless**: no participant plays a special role, such as being the leader in producing a block.
*   **Byzantine fault tolerant**: functional even with up to one-third of faulty or

    malicious nodes.
* **Final**: transactions are confirmed within 1-2 seconds.

Lachesis allows scaling transaction throughput while keeping **instant finality** without an increased risk of centralization.

### How does Lachesis work? <a href="#how-does-lachesis-work" id="how-does-lachesis-work"></a>

AssetChain's consensus mechanism, Lachesis, combines asynchronous Byzantine fault tolerance (aBFT) with directed acyclic graphs (DAGs). aBFT allows nodes to process transactions independently without requiring the sequential exchange of blocks, resulting in faster transaction times. DAGs, which represent blocks of transactions connected in a non-linear manner, facilitate this.

In Lachesis, each validator has its own local DAG and creates blocks from incoming transactions, which are added to its DAG. Validators asynchronously exchange these blocks, spreading information through the network. Once a majority of validators agree on a block, it is added to a blockchain containing all final consensus transactions, which is the AssetChain mainnet.

By combining the advantages of aBFT and DAGs, Lachesis achieves fast and efficient transaction processing, with the whole process of submitting a transaction and adding it to the mainnet taking approximately **1-2 seconds**.

### What are epochs in Lachesis? <a href="#what-are-epochs-in-lachesis" id="what-are-epochs-in-lachesis"></a>

Lachesis’s structure is a DAG of events. To optimize storage and retrieval, the DAG is separated into sub-DAGs, each of which is called an epoch. Each epoch comprises many finalized blocks.

Each epoch is sealed when one of these conditions is satisfied:

* The epoch reaches a defined number of blocks
* The epoch lasts for a specified time
* At least one cheater is found in this block

When an epoch gets sealed, its inner epoch indexes get pruned, and new events of the sealed epochs are ignored. Each epoch forms a separate DAG, and thus parents from other epochs are not allowed.

For a sanity check, each event includes the hash of the previous epoch.

### Comparison of Consensus Protocols <a href="#comparison-of-consensus-protocols" id="comparison-of-consensus-protocols"></a>

| Consensus Protocols  | Nakamoto | pBFT | Lachesis aBFT |
| -------------------- | -------- | ---- | ------------- |
| Scalable             | +        | +    | +             |
| Low Latency          | -        | -    | +             |
| Highly Decentralized | +        | -    | +             |
| Secure               | +        | +    | +             |
| Green                | -        | +    | +             |
