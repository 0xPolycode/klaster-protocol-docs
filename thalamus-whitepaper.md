# Thalamus • A Cross-Chain Clustering Protocol


<img style="height: 100px" src="https://github.com/0xPolycode/thalamus-protocol-whitepaper/assets/129866940/d5058d41-4401-4868-b451-5c1fdd2ef010"></img>

## Introduction

Since the advent of smart contract blockchains in 2014 - blockchain researchers and developers have been working towards solving a few fundamental issues with the design of blockchain networks. One of those key problems is scalability - the blockchain design (as it exists today) must verify and store *all* transactions ever processed on it. This makes sense for large transactions between big accounts, but - if we're going to make blockchain the default financial backbone - there is no sense to employ the same security and finality for a person buying a cup of coffee and an insurance company paying out a multi-million insurance claim.

Currently, the most popular approach for solving this is to create multiple, independent blockchain networks (called many names - L2s, rollups, appchains, parachains, ...) which process transactions with differing speed, security and finality guarantees. This approach is employed by most major blockchain projects - from Ethereum with rollups, Polkadot with parachains, Avalanche with subnets, Cosmos with their "Internet of Blockchains", etc...

While there are still technical challenges in implementing these systems on a mainstream, commercial basis - the future of blockchain development is clear - many, independed AppChains & L2s, processing their own transactions and settling periodically on a few robust, battle-tested L1s. 

## Challenges

Developing production blockchain solutions as a series of smaller networks, however, comes with its own set of challenges. A notable change is that - having multiple independent networks breaks composability and atomicity - two key aspects of blockchain networks today and the backbone of the interesting solutions in DeFi & NFT spaces. Let's define our terms:

* **Composability** - The ability of one end user account or smart contract to interact with *any* other smart contract deployed on the network(s). Additionalliy, the ability to *chain* the interactions between multiple smart contracts and use the result of one operation as the input for the next operation.
* **Atomicity** - The guarantee that, in a chain of operations, either all of them succeed *or* none of them do. Meaning - a protection from partial execution. 

A protocol which solves cross-chain composability and atomicity would truly enable the "Internet of Blockchains" or "Internet of Smart Contracts" to scale and reach mainstream adoption. 

Lately, there has been a lot of progress in cross-chain communication protocols and standards - with notable examples such as Chainlink CCIP, Connext or Layer Zero. These communication protocols enable blockchains to share messages and execute remotely execute actions across different blockchain networks. 

In this paper, we are going to present Thalamus - a middleware protocol positioned between smart contracts and cross-chain communication solutions, enabling a seamless, developer-friendly & standardized way for all Web3 apps to adapt to the cross-chain future. 

Borrowing from networking terminology, we'll define the term **Blokchain Clustering** - Making multiple, independent blockchain networks & multiple cross-chain communication solutions act as a single virtual composable and atomic blockchain network. 

## Thalamus

Thalamus is a _blockchain clustering_ protocol - defining an interaction standard for smart contracts to communicate through. With Thalamus - users can interact with smart contracts without needing to know *which* blockchain network the contract is on. Instead, the users will interact with a blockchain of their choosing and Thalamus will work to aggregate all the cross-chain communication providers to enable a seamless interaction with *any* contract on *any* chain, while preserving composability and atomicity of the requested operations.

For smart contract developers, Thalamus will abstract away cross-chain communication and drastically reduce the time-to-market for cross-chain applications. For existing DeFi & NFT projects, writing simple adapters will make them fully compatible with Thalamus and enable cross-chain calls to protocols such as Uniswap, AAVE, MakerDAO, ...

![Thalamus Architecture](https://github.com/0xPolycode/thalamus-protocol-whitepaper/assets/129866940/2773ed3f-1a77-4fee-9541-47bc0cf5aeea)

### Implementation

The Thalamus protocol is implemented as a singleton contract on every supported chain. This singleton contract is tasked with creating, sending and receiving _Remote Transaction Call_ (RTC) messages. Each RTC message contains the data needed to execute the transaction on the destination chain.

```
RTC Message Structure
---
Destination Chain ID
Remote Execution Call Data: RTCCallDataObject[]

RTCCallDataObject:
  EVM Call Data
  AdapterType (ERC20, ERC721, ...)
  AdapterParams (e.g. for ERC20 - tokens used, approval amount, apporval recipients)
```

![RTC Arch](https://github.com/0xPolycode/thalamus-protocol-whitepaper/assets/129866940/a5b3e4be-72c7-44e6-81f4-1345003fe39c)



