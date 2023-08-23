# Thalamus • A Cross-Chain Clustering Protocol

<img style="height:100px" src="https://github.com/0xPolycode/thalamus-protocol-whitepaper/assets/129866940/e8ab9dae-a633-4d3b-8467-c04f15b651e9"></img>

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

Borrowing from networking terminology, we'll define the term **Blokchain Clustering** - Making multiple, independent blockchain networks & multiple cross-chain communication solutions act as a single virtual blockchain network with preserved atomicity and composability. 

## Thalamus

Thalamus is _not_ a cross-chain communication protocol. Thalamus is a standardized interface which abstracts away cross-chain communication protocols from smart contract developers. It enables easy conversion of exsiting DeFi, NFT & payment apps into cross-chain native apps _and_ drastically reduces the time needed to develop new cross-chain applications. The V1 of Thalamus will have all of its cross-chain communication handled by ChainLink CCIP. Thalamus is a set of smart contracts, deployed on all compatible blockchain networks, which drastically reduce integration time of new and existing DeFi apps into cross-chain protocols, such as CCIP.

It achieves this through the creation of several standards:

- **Two Way Handshake RTC** - Remote Transaction Calls - a standardized, composable way to call a function on an EVM smart contract from one blockchain network to another. Thalamus defines a two-way handshake standard so that Web3 apps can keep composability & atomicity while communicating though cross-chain solutions.
  
- **Native Multichain Assets** - Extensions of ERC20 and ERC721 standards, which make them cross-chain compatible.
    - **Native Multichain ERC20** - An ERC20 token which has the same address on all deployed blockchain networks and has its supply _shared_ between multiple blockchain networks through secure cross-chain communication solution.
      
    - **Native Multichain ERC721** - An NFT which has the same address on all deployed blockchain networks and has its items _shared_ between multiple blockchain networks through secure cross-chain communication solution.
      
- **dApp Adapters** - Smart contracts which handle the additional logic required by `Thalamus RTC` to perform cross-chain blockchain transactions. They are generally lightweight wrappers around existing dApps _or_ built-into solutions which are cross-chain since inception. 

<img style="height:550px" src="https://github.com/0xPolycode/thalamus-protocol-tech/assets/129866940/01f7c9e5-db62-4124-bf43-f42cb78a9e2e"></img>

## Implementation

The Thalamus protocol is implemented as a singleton contract on every supported chain. This singleton contract is tasked with creating, sending and receiving _Remote Transaction Call_ (RTC) messages. Each RTC message contains the data needed to execute the transaction on the destination chain. The RTC messages are forwarded to the singleton contract on the destination chain, which then executes the action.

Thalamus system works best when paired with MultiChain Asset standard contracts. So far, we have developed the standards for multichain ERC20 and ERC721 assets, but a `MultiChainAssetAdapter` can be built for any other type of asset.

`ERC20MultiChainAssetAdapter` and `ERC721MultiChainAssetAdapter` both support wrapping of regular `ERC20` and `ERC721` assets, making the Thalamus system immediately adaptable for DeFi, without changing the implementation of blockchain apps (e.g. regular Uniswap or regular AAVE would work just fine).

```
RTC Message Structure
---
- Destination Chain ID: Number
- Remote Execution Call Data: RTCCallDataObject[]

RTCCallDataObject:
  Destination Contract Address: string
  EVM Call Data
  MultiChainAssetAdapter Type: string (e.g. 'ERC20')
  MultiChainAsset Address: string
  Adapter Params: Object (e.g. for ERC20 - tokens used, approval amount, apporval recipients)

```
![Untitled Diagram drawio(35)](https://github.com/0xPolycode/thalamus-protocol-tech/assets/129866940/82e35fa6-dc8f-4e6a-bbfe-cbf341af301b)


Let's explore the flow of a Remote Transaction Call as outlined in the diagram above. We will be swapping on Uniswap. Our assets are on Avalanche, but we want to swap on Optimism.

#### 1. **Create RTC**
A user interacts with a frontend and calls the `rtc` function on the source chain singleton contract with the RTC message as the paramters. Our RTC message would look like this:
```json
{
   "destChainID":10,
   "rtcCallData":{
      "contract":"0xUniPoolAddressOnOptimsim",
      "callData":"Encoded call data. Call function swap",
      "mcAssetAdapterType":"ERC20",
      "mcAssetAdapterParams":{
         "tokensUsed":2000000000000000000,
         "approvals":[
            {
               "amount":2000000000000000000,
               "beneficiary":"0xUniPoolAddressOnOptimism"
            }
         ]
      }
   }
}
```
We encode this and pass it on into the `rtc` function on the Thalamus singleton contract.

#### 2. Commit RTC
Thalamus singleton contract will receive the message, fetch the logic for the `MultiChainAssetAdapterType` (if specified) and choose the cross-chain communication solution to be used for the sending of the message.

The selection of the cross-chain communication solution is a work-in-progress. The V1 of the Thalamus protocol will have Chainlink CCIP as it's only provider, so for V1 - this step means selecting CCIP and encoding the message. 

For each way of communication, there is an `expiry` period which is attached to the message. After the `expiry` period has passed, the destination chain singleton will reject the message (if it comes through the communication channel). 

After commiting the message, the source chain singleton will wait for the `ACK` or `NACK` signals from the destination chain. `ACK` signal means the action was sucessfully completed on the destination chain, while the `NACK` signal means that the message was reverted on the destination chain. 

If a period of `3 * expiry` has passed and no `ACK` or `NACK` message has been received, the source chain will consider the action a failure. Since the destination chain contract must check the `expiry` variable before performing the action, the source chain can safely revert any changes made (e.g. tokens reserved) when `3 * expiry` (one `expiry` for message from source to destination, one `expiry` for message from destination back to source and one `expiry` of buffer) has passed.

The expiry times for each chain should be considered at average `finality` time, with the extra expiry holding period serving as a buffer for deviations in finality time. If abberant finality tiems are observed on source or destination chains, the system will automatically `revert` all actions, since the `ACK` singnal will take too long to propagate. 

The singleton will assingn each sent message a `UUID` and a status! The status of the message can be `PENDING | ACK | NACK | EXPIRED` - depending on the situations explained above.

#### 3. Request RTC Execution
Request RTC execution. The cross-chain communication provider will call the `rtc` function on the destination chain singleton contract. The contract will then execute the desired function on the desination chain and (if the function was successfull) it will call the `rtcAck` function back to the source chain. 

#### 4. Execute Function on Adapter
The singleton contract will usually interact with `MultichainAdapter` contracts which wrap around existing DeFi and NFT apps and perform some additional logic. An example would be a `UniswapMultichainAdapter` contract, which would be able to wrap the LP tokens into `MultiChainERC20` tokens and send them back to the source chain. 

#### 5. & 6. Success / Fail - ACK/NACK
If the function was a sucess, the `MultichainAdapter` contract will perform all the necessary actions and send the `ACK` signal back to the source chain through the cross-chain communication provider (e.g. CCIP). The `ACK` signal can have additional actions appended to it! E.g. -  the `ACK` for the Uniswap `swap` function will come with wrapped LP tokens. If the function reverts - the singleton will send the `NACK` back to the source chain. 

#### 7. Resolve RTC status
The source chain singleton will receive the `ACK` or `NACK` and set the status of the RTC message accordingly. In the case of `NACK`, it will revert any changes made in the first step. 

#### 8. Notify user
The singleton will emit an event that the frontend can pick up, to notify the user that the process was a success.

## ThalamusDAO

The Thalamus Protocol will be deployed in two phases - V1 & V2. V1 will be a single-provider phase, where only Chainlink CCIP will be used to send messages back and forth between chains and where we will test out the infrastructure and stress-test the system. V2 will introduce ThalamusDAO - a governance system where delegated members will be able to add new cross-chain communication providers.

The DAO will be governed by a governance token `$THLD`. 

Beyond serving as a DAO governance token, the `$THLD` token will accrue value from fees which Thalamus will charge for executing cross-chain actions. This fee will be charged in the native gas token and charged above the fee charged by the cross-chain communication provider. The fee will be used to buy and burn $THLD tokens. A part of the fee will be routed to the treasury.

We are discussing the possibility of using the $THLD token to incentivise certain behaviours, such as providing redemptions for wrapped ERC20 tokens on all deployed chains.

