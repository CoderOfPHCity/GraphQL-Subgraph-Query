# Indexing Data On KaiaChain With Subgraph

One of the important characteristics of Web3 is the necessity for rapid and consistent access to on-chain data. 

Ethereum, a prominent smart contract platform, enables access to its blockchain data through [JSON-RPC](https://www.jsonrpc.org/), a remote procedure call (RPC) protocol that encodes data in JSON format.

While JSON-RPC is a useful approach to communicate with Ethereum data, it may not always be the most efficient or user-friendly option for developers looking to create scalable dApps. 

That is where [The Graph](https://thegraph.com/) comes in. The Graph is a decentralized protocol designed to index and query blockchain data that is difficult to access directly.

When your dapp has to display information about the current status or history of something in your smart contract, using events rather than state variables will conserve gas and improve overall efficiency. 

Things like tracking an [ERC721](https://docs.openzeppelin.com/contracts/3.x/erc721) tokenId, transaction history, approval activity, and minting. You simply need to read this info.

So, if you design your frontend to read from logs, your smart contract could potentially use less storage variables. Your frontend can do queries from an external source, with all data prepared in a clean style.

The problem is that you'll need some infrastructure to collect all of the important events, clean them up, and then host them elsewhere. The Graph solves these problems.


## Prerequisites
1. Fistly you need a basic understanding of Ethereum development and programming fundamentals.
2. A web3 wallet (e.g., MetaMask, Rabbi wallet) with ETH on Ethereum mainnet.
3. Ensure to have [Docker](https://docs.docker.com/engine/install/) and [Docker compose](https://docs.docker.com/compose/install/) installed . 
4. The[ Graph CLI](https://thegraph.com/docs/en/developing/creating-a-subgraph/) and [Node.js ](https://nodejs.org/en/download/package-manager)installed

## Getting Started
Go to thegraph's studio: https://thegraph.com/studio/subgraph. You'll need to connect your metamask wallet to log in. Enter a project name and click the button "Create Subgraph"
![kaia111](https://hackmd.io/_uploads/rJglHOJGkg.png)

Next, Install the [Graph CLI⁠](https://github.com/graphprotocol/graph-tooling/tree/main/packages/cli) to this section
To build and deploy a subgraph, you will need the Graph CLI.
```
npm install -g @graphprotocol/graph-cli@latest
```
The graph init command can be used to set up a new subgraph project, either from an existing contract or from an example subgraph.

Now that you have got graphcli installed, run the following commands simultaneously to set up the graph initialisation environment of your terminal.
![start1](https://hackmd.io/_uploads/BJO6y5yzJl.png)


You'll be prompted to enter a bunch of info. Here's an example on my local terminal environment.

First select the blockchain, for the purpose of this tutorial we are working with [KaiaChain](https://www.kaia.io/) which is an EVM compatible chain so we go with Ethereum.
![eth](https://hackmd.io/_uploads/B1H5FOJMJl.png)

Subgraph supports over 40+ blockchains inclkuding kaiachain.
![kaia](https://hackmd.io/_uploads/Hyx9adkMJl.png)


The Graph CLI retrieves the contract ABI from Etherscan via a public RPC endpoint. Failures are expected, but retries usually fix them. If failures persist, consider utilizing a local ABI.

For this tutorial guide, i use the `klaytn DAI` stablecoin contract address to index the transfer and approval data on the kaia blockchain.
```
0x5c74070fdea071359b86082bd9f9b3deaafbe32b
```
Next is to set a path for the smart contract ABI locally or using the public RPC endpoint. Then set the start-block number and the contract name.
![key](https://hackmd.io/_uploads/BJ87hOkGkl.png)

## Design a Schema

Go to your subgraphs folder. In my case, it would be `kaiasubgraph`. Open your code editor from this folder.

The default schema created by graph init is named `schema.graphql`. It generates an entity (data table) for every smart contract occurrence. 

![codeimg-facebook-cover-photo (1)](https://hackmd.io/_uploads/SJgcxtkzkg.jpg)

The schema defines 3 entities: `SetOwner`, `Transfer`, and `Approval`, each tracking key events related to token ownership, transfers, and approvals on Kaiachain. 

The entities store relevant metadata like addresses, amounts, block details, and transaction hashes to provide a tamper-resistant history of token-related activities for developers to build applications upon.

![codeimg-facebook-highlighted-image](https://hackmd.io/_uploads/HySROY1zyx.jpg)
The src folder will have a ts file named after your contract.

Feel free to update the event handler functions (e.g., handleSetOwner, handleTransfer, handleApproval) to process event data according to your specifications, such as adding custom logic, converting data, or communicating with external services.

The.ts file is generated automatically using the subgraph structure and the smart contract's ABI (Application Binary Interface). 

It converts event data from the Ethereum Virtual Machine (EVM) into entities defined in the schema, allowing you to persist and query the data via a GraphQL-based API.

The event handler functions are called when the smart contract emits the corresponding events, and they build or update the entities in the subgraph's data store.

## Methods to map incoming events to schema attributes 
The corresponding event handler functions in the .ts file are:

* handleSetOwner
* handleTransfer
* handleApproval

The key parts of this file are the event handler functions. These functions tell The Graph what to do when certain events happen on the blockchain, like when token ownership changes, tokens are transferred, or approvals are made.

For example, let's say your contract has a `SetOwner` event. The Graph will generate a handleSetOwner function in the .ts file. This function's job is to take the data from the `SetOwner` event and create a new database record to store that information.

The function does this by taking the event parameters, like the new owner's address, the block number, timestamp, and transaction hash, and using them to create a new `SetOwner` entity. It then saves this entity to the subgraph's database.

The same pattern applies for other events, like `Transfer` and `Approval`. The Graph generates handleTransfer and handleApproval functions that create new entities based on the event data.

The key thing to remember is that these event handler functions are the main way you interact with the data coming in from the blockchain. You can customize them to do more than just create new records

Finally, before attempting to deploy your subgraph, ensure that your subgraph.yaml file provides the appropriate handler functions. The initial auto-generated parameters are probably incorrect.

Under datasources, you'll find eventHandlers. We should only keep those that we need.
![codeimg-facebook-highlighted-image (1)](https://hackmd.io/_uploads/SkhjaY1fyg.jpg)

## Deploying The Subgraph
The Graph currently runs a Hosted Service, which is centralized and was built initially for more adoption (it will eventually be sunset). We'll be using this Hosted Service so we don't need to worry about staking GRT to signal our Subgraph.

We'll need the deployer key shown in your Subgraph's draft (e.g., https://thegraph.com/studio/subgraph/kaiasubgraph; `kaiasubgraph` is the name of the project to deploy it.

Once you have the key, run the following commands in your terminal:

Authenticate with your access token:
![start](https://hackmd.io/_uploads/SkwMlc1MJl.png)
Just follow the cli instructions you see in The Graph's studio page and you should be good to go.

After successful deployment, you'll notice a query url that you may use from a frontend app.
![deplooy](https://hackmd.io/_uploads/r1T0ZqJfyg.png)
Congratulations! You can now query your subgraph on the decentralized network!

For any subgraph on the decentralized network, you can start querying it by passing a GraphQL query into the subgraph’s query URL which can be found at the Explorer page above.

After deploying your Subgraph, it will begin indexing the data and become available for querying via The Graph's hosted service or a decentralized network of Graph Node operators. 

Once synced, go to the Playground tab on your Subgraph and click the Play button to perform a query. 
Here are some sample queries on the playground interface.

Now lets query the following scehma directly from the playground.
![play](https://hackmd.io/_uploads/BJCGX9yGkl.png)

```
{
  setOwners(first: 5) {
    id
    owner
    blockNumber
    blockTimestamp
  }
  transfers(first: 5) {
    id
    from
    blockNumber
    transactionHash
    to
    amount
  }
}
```

The output in the subgraph explorer reflects the fields defined in the subgraph schema, which match the event data captured by the handler functions. 

The GraphQL query selects the first 5 "setOwners" and "transfers" entities, returning the corresponding fields like "id", "owner", "from", "to", "amount" etc.

And below you can see the same value attributes from `kaiachain explorer`.
![b8](https://hackmd.io/_uploads/rydJS6kzyl.png)


## Publish a Subgraph
After you've deployed and thoroughly tested your subgraph in Subgraph Studio, you may put it into production by publishing it to The Graph's decentralized network. This operation makes the subgraph accessible, allowing Indexers to begin the indexing process.

To do so, click the Publish button, pick the network (in this case, Kaia Mainnet), and submit the transaction using your Web3 wallet.
![publish](https://hackmd.io/_uploads/rJZkSqkGye.png)
Hurray!you now understand how to build, deploy and publish your own unique Subgraph on The Graph.

The GitHub repo associated with this tutorial can be found [Here](https://github.com/CoderOfPHCity/GraphQL-Subgraph-Query).



