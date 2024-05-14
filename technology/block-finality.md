---
description: Time it takes to finalize a block.
---

# 🏆 Block Finality

Finality is the point at which a transaction on a blockchain becomes irreversible and permanently recorded. But before we dive in further, it’s important to have a basic understanding of how a blockchain works.

At its core, a blockchain is simply a database that contains transactions. When a transaction is submitted to a blockchain, it’s not added directly to it. Rather, it’s bundled into a “block” alongside other transactions, which is then incorporated into the database. As such, a blockchain is a sequential chain of blocks that contain transactions, hence its name.

<figure><img src="https://docs.fantom.foundation/~gitbook/image?url=https%3A%2F%2F214665317-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MKjpUMrhoyibSIWfRrl%252Fuploads%252FGSdFgvqTGJIfUC5YLSVE%252FTouring%2520Sonic%2520-%2520Blockchain.png%3Falt%3Dmedia%26token%3D02b298e0-613e-4ab9-a4fb-f1771e782eb0&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=23f7b0ec9a348d257151966505a682c74c56f0a22503fe6909e3f0c42d51a966" alt=""><figcaption></figcaption></figure>

Most importantly, a blockchain is decentralized, which means it doesn’t run on a single server. Instead, the blockchain exists simultaneously on a massive amount of computers around the world that run software to communicate with each other and agree on the content of the database. (On AssetChain, these computers are called **validators**.)

It’s these computers that bundle transactions into blocks and add them to the blockchain at frequent intervals. Every \~10 minutes for Bitcoin, every \~12 seconds for Ethereum, and every \~0.4 seconds for the Sonic closed testnet.

<figure><img src="https://docs.fantom.foundation/~gitbook/image?url=https%3A%2F%2F214665317-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MKjpUMrhoyibSIWfRrl%252Fuploads%252FuipZA069hoVAjm6X2o3q%252FChain%2520Comparison.jpg%3Falt%3Dmedia%26token%3De0f51eb8-e007-4da2-87d2-4ab57b69b4c7&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=2ba10ce5f5831aa071e7f1a53534990429f7f66af6472e8bb381687574c5ce94" alt=""><figcaption></figcaption></figure>

However, once a block is added to the blockchain, it doesn’t mean the transactions contained within are instantly final and permanent. In the majority of blockchains, a single computer is usually picked to create the next block and add it to the blockchain. Other computers in the system then receive this block and validate the transactions within it to make sure they’re valid.

However, due to the design of most blockchains, there can be occasions when two computers are simultaneously chosen to create a block. In such a scenario, how does the rest of the system know which block to follow? Well, they must wait until the next computer is chosen to create the next block, and the previous block used by this computer is accepted as the legitimate one, while the alternative block is disregarded.

This is called the **longest chain rule** as every computer in the system follows the version of the blockchain with the most blocks, which naturally would be the version on the most recent computer chosen to create a block. Both Bitcoin and Ethereum implement this mechanism.

<figure><img src="https://docs.fantom.foundation/~gitbook/image?url=https%3A%2F%2F214665317-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MKjpUMrhoyibSIWfRrl%252Fuploads%252FKtc6gf2IC53FYC0zVJKX%252FTouring%2520Sonic%2520-%2520Longest%2520Chain%2520Rule.png%3Falt%3Dmedia%26token%3Da2ee13ce-e82d-4641-8750-94ada6e0f97b&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=b4294d398ac49f8d303adf389920a0ae6d55a172c2b8c640bfe01bdbca71bea0" alt=""><figcaption></figcaption></figure>

After a computer is selected to create the next block, bringing the blockchain back on track, we still have to wait for several more blocks to be created. Due to potential communication delays between the computers in the system, it’s technically possible for some of them to still have the old invalid block in their version of the blockchain.

After a certain amount of blocks have been created, this is no longer a risk as we can assume those outdated computers have undergone **chain reorganization** to change their database to the current and correct one. Only then can we consider the transactions in the initial block as final and irreversible.

And now we’re ready to define **time to finality**, which simply means how long it takes for a transaction to be considered irreversible after it’s submitted. On Bitcoin, it takes around an hour (6 blocks created). On Ethereum, it takes around two minutes (12 blocks created). On the AssetChain mainnet, it takes **only a few seconds**.

That’s because the design of AssetChain allows for a block and its transactions to be considered final and irreversible the moment it’s added to the blockchain. The longest chain rule and chain reorganizations do not exist on AssetChain. Once a transaction has been added to the blockchain, it’s **instantly final**.

There are a few things that work in conjunction to make this possible. On AssetChain, no single computer is chosen to create the next block. Instead, the block is added to the blockchain once the majority of the computers in the system have received it. As such, there can never be a situation on AssetChain in which the computers disagree on the state of the database.
