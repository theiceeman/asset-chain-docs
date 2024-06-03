---
description: >-
  In this section we'll get you deploying a sample contract on Asset Chain in
  less than 10 minutes.
---

# 🕔 Quick Start

Let’s see how to deploy a smart contract on Asset Chain using the [Remix IDE](https://remix.ethereum.org/) for simplicity.

### Get everything ready <a href="#get-everything-ready" id="get-everything-ready"></a>

Before getting started:

1. Follow for the step-by-step on how to add Asset Chain testnet to [Metamask](../general-info/add-asset-chain.md).
2. This guide assumes you have got Testnet RWA and connected to the Asset Chain Testnet Network. Learn how to do that in [Testnet Faucets](../tools/faucets.md).

We are ready to get started!

### Remix & Sample Code <a href="#remix-and-sample-code" id="remix-and-sample-code"></a>

[Remix](https://remix.ethereum.org/) is a no-setup tool for developing smart contracts. It’s easy to get started allowing a simple deployment process, debugging, interacting with smart contracts, and more. It’s a great tool to test quick changes and interact with deployed smart contracts.

<figure><img src="broken-reference" alt=""><figcaption><p>smart contract code sample</p></figcaption></figure>

For the sake of this tutorial, we will be deploying the ‘ERC20Token.sol’ smart contract for minting a fungible token, but you can use any of your code.

You can copy and then paste this in Remix or use your own contract code. Here's the sample code:

ERC20Token.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.3/contracts/token/ERC20/ERC20.sol";

contract ERC20Token is ERC20 {
    constructor(string memory _name, string memory _symbol)
        ERC20(_name, _symbol)
    {
        _mint(msg.sender, 1000000000 * 10**decimals());
    }
}

```



### Steps to deploy <a href="#steps-to-deploy" id="steps-to-deploy"></a>

1. Copy the sample code and paste it in one of the .sol files in Remix
2. To compile your smart contract, go to the `Solidity Compiler tab` and select the contract you want to compile
3. Click on "Compile", you can also enable "Auto Compile" for automatic compilation whenever you change the contract code.

Make sure to open the advanced configurations and setting the EVM version to London. This is to avoid an issue with the `PUSH0` opcode. You can read more on this specification with all Optimism chains [here](https://community.optimism.io/docs/developers/build/differences/#opcode-differences).

![](https://docs.mode.network/\~gitbook/image?url=https%3A%2F%2F2176895816-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FmOUA87dDndFyiETJjxpf%252Fuploads%252FgvDYYhLyR0v2evtGgZe7%252Fimage.png%3Falt%3Dmedia%26token%3D1a9fe833-6167-487f-a89d-d5cc313d53a1\&width=768\&dpr=4\&quality=100\&sign=4a0175cbb69b731dc9ab52ed73451d163a398d62c2868e34631bc8bfa145c211)

1. Once the smart contract is compiled successfully, switch to the "`Deploy & Run Transactions`" tab.
2. In the "`Environment`" dropdown menu, select "`Injected Provider - MetaMask`"; this will connect your MetaMask to Remix and will allow you to make transactions from that connected wallet.

Make sure to have Asset Chain Testnet as your selected network in Metamask before deploying.

![](https://docs.mode.network/\~gitbook/image?url=https%3A%2F%2Flh4.googleusercontent.com%2FjmsucoJ4vr4ByW3\_0Nt4gwlckzu78pvh7ugVp2nEep9z9LtpY-BuC5WmhX4k\_uKk2vA\_iIvDZg-VEn8YDzKdoSzmE327wjbLiCIpCGe9xc\_GAxBOC5-LYet-qBNPQ54W5waFpeMZak61a-rmk\_ITxog\&width=300\&dpr=4\&quality=100\&sign=8abd245423fe32a3df61b633698504982aad62c31a9c4e3c6e2336a19e4fb375) ![](https://docs.mode.network/\~gitbook/image?url=https%3A%2F%2Flh6.googleusercontent.com%2FnIYOD8FEnw-1qCtgMI\_uKK4qRwEjciveycdc3q6iLtuW7su7sOQMZHhG1dw8Rwk2ulO4JFlQU8YxQlJIB8c6uMZJ5t19PCikrkIKVsRZW68PVRz8RVs1NtQOxrQ6x7CwZXtwjlv6W4Fe9x45\_44LWSQ\&width=300\&dpr=4\&quality=100\&sign=dd7dac6bc0c696aeb1fbf00a5d4b6fd03533ea96ad855e006ca34a4ea1d00621)

1. Select the compiled contract you want to deploy and click ‘Deploy.’

Now, MetaMask should pop up and ask you to confirm the transaction with super low fees.



<div align="left">

<figure><img src="broken-reference" alt="" width="208"><figcaption></figcaption></figure>

</div>

{% hint style="success" %}
**Congrats! You just deployed your first smart contract to Asset Chain.**
{% endhint %}



To learn more about Asset Chain and how to turn your code into a business, join our [Discord](dev-onboarding-checklist.md) and say hello 👋

