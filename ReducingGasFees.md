At STK, we’re on a mission to enable instant cryptocurrency payments at point of sale, making it as easy to use crypto as it is to use cash. Our team of developers is working hard to build a stable, scalable solution and as part of this process, we conduct regular research on the latest solutions for common network challenges. Our latest research was around reducing the amount of gas used for the execution of smart contracts. As part of our commitment to open and honest communications with our community, we asked our awesome Ethereum Expert, Natalie, to write a summary of her experience. Here’s how it went down:

# On reducing gas fees
When I was first tasked with optimization of Solidity code, reducing the amount of gas used for execution of smart contracts, I had absolutely no idea what to do. The Solidity docs mentions ways to optimize, but they are sparse and hard to find. There are many posts on Stack Overflow outlining different methods to optimize, but are still scattered across different websites.

As Dapp Developers, minimizing the gas spent on execution is more important in our field than any other. In other languages, knowing how to use red-black trees will help you land a new job; but for Ethereum, all computation on the network is subject to fees. This is what’s known as “gas.” Typically in optimization, your goal is to minimize gas for executing your contract. Thus, if your contract is needlessly large, you’ll spend significantly more gas on executing than otherwise needed.

# Step 1: Yellowpaper

First, I had to discover which operations in Solidity used the most gas. This would help me narrow down the data structures and operations that would help us reduce the most amount of gas possible.

Reading the [Ethereum Yellowpaper](https://ethereum.github.io/yellowpaper/paper.pdf), [Overview of Gas Costs](https://docs.google.com/spreadsheets/d/1m89CVujrQe5LAFJ8-YAUCcNK950dUzMQPMJBxRtGCqs/edit#gid=0), and [Ethereum Beigepaper](https://github.com/chronaeon/beigepaper/blob/master/beigepaper.pdf) helped determine the most costly operations in Solidity. The Yellowpaper is the official technical specification of the language, identifying which operations are the most gas-consuming. The Beigepaper is essentially the Yellowpaper, with the technical specifications removed, and expressed in english. These resources were extremely important in helping determining the most gas costly operations, and giving us a starting point of where to begin with optimization.

# Overview of STK-smart-contracts

[GitHub Source](https://github.com/STKtoken/STK-smart-contracts)

STK lets you convert cryptocurrency into local currency in real-time, so you can make instant payments at point of sale, through the STACK app.Now you can add your cryptocurrency wallet alongside your other currency wallets in STACK. When you tap to pay through the STACK app, you can make purchases at any retail location that supports credit or debit cards. STK opens a bridge between the Ethereum blockchain and traditional credit card payment rails.

Our contracts implementing the above are open-sourced at the GitHub repository listed above. To initiate a payment channel:

* Deploy `STKChannelLibrary` to initialize the required structs for STKChannel
* Deploy `STKChannel`, which will handle all functionality in the payment channel

For a higher level understanding of how our payment rails work, check [this](https://mycardboarddreams.wordpress.com/2017/10/02/how-stk-works/) out.

# Testing on Remix

While executing on the Javascript VM with Remix, transaction receipts make a distinction between transaction and execution costs.

**Execution Costs:** processing costs that are used on the virtual machine. These do not include deployment costs or the costs of calling a function on a smart contract.

**Transaction Costs:** execution cost + cost of sending data and contracts to the blockchain. Transaction costs includes the execution costs. Here are some examples of typical transaction costs:

* Calling constructor: 21000 units of gas
* Deploying new contract: 32000 units of gas

# Gas Used by Transaction vs Gas Price vs Value (on Etherscan)

If you are executing on Remix through a testnet such as Rinkeby, Ropsten, or Kovan, then you’ll see an Etherscan of the transaction on the respective testnet. This can also be useful in terms of minimizing the gas cost.

However, there seems to be a bit of confusion on what the different gas-related fields on Etherscan mean.

[This](https://cdn-images-1.medium.com/max/1600/0*pp1rlE7wboL-7PQo) is the Etherscan of the closing of a channel that was created using the code. Here is a link to the Etherscan of payment channel initialization, closing, and settling.

* **Gas Limit:** The maximum amount of gas (in units) that can be used in the execution of this function
* **Gas Used By Txn:** The amount of gas (in units) that was actually used by the transaction
* **Gas Price:** The price per unit of gas
* **Actual Tx Cost/Fee:** The amount of “funds” spent on execution in ETH and Fiat.

For a precise measurement of gas that you have used or gas you have saved, don’t use actual tx cost/fee. Use the ETH Gas Station, take the Gas Used by Txn and Gas Price; and it will give you a more precise transaction fee in ETH and in Fiat.

Remember that the precise transaction fee will change, depending on the Gas Price. The Gas Price standard is set by miners, depending on recent prices to get mined within a certain number of blocks. However, many wallets allow you to set it yourself. One can always use a lower than average gas price, but you run the risk of the [transaction not being picked up by miners](https://medium.com/@jgm.orinoco/releasing-stuck-ethereum-transactions-1390149f297d). Ultimately, this out of your control as a developer, thus I’ll be focusing on minimizing Gas Used by Txn instead. This is directly related to the efficiency and computation of your code. 

# Determining structures that can be moved off-chain 

As the Ethereum network charges for execution costs of code, it also charges for storage costs. Each storage variable costs around 20000 units. Naturally, this is one of the places you will want to look if you are looking to minimize the amount of gas your code is using.

In our implementation, the payment channel can be in a number of states:

* open: allowing transactions to occur and IOU’s to come in
* closed: transactions can still occur if the user has agreed to keep their tokens in the channel (if they have not agreed they cannot continue transacting), contest period begins for a period of `timeout_` blocks
* settled: contest period is over, and the channel state is reset to open

After analyzing the flow creating and closing a payment channel, I started to realize that closingAddress was not mandatory. The variable was not required for modifier checks. It was set when a channel was closed, and reset to the default zero address once the contest period was over. As the close channel already requires that only channel participants can close it, it would not be of use for us to keep track of who closed it. Thus, it was removed. This saved us about 5¢ USD in gas costs, assuming gas price of 2 gwei.

Another thing that stood out was `openedBlock_;`, as it was a similar structure. It was being set at the opening (or initialization) of a channel, and was reset after settling. It became a question of whether this was necessary to be stored on the blockchain. If we want to check whether or not the channel was closed or not, it could be inferred by another variable, the closedBlock. This variable was initialized to 0 upon creation of a channel, and set to the block number at the state which the channel closed. In other words:

* `if closedBlock==0:` channel is still open
* `if closedBlock>0:` channel is closed

Always look for opportunities in your code to infer things about the state of your code, without having a variable to explicitly define and spell it out.

# Global variable constants

[Source](http://solidity.readthedocs.io/en/v0.4.21/contracts.html#constant-state-variables) 

If a global variable does not need to be changed throughout the life of the contract, think about using a constant instead. A constant variable does not increase gas costs, as it is not stored in the storage. It’s stored in the bytecode, which is a much smaller number. In terms of gas amounts, storage costs would be around 20000; whereas storage in the bytecode would be around 200.

Unfortunately, this tactic was not useful in our contracts. Originally, we were thinking timeout_ could be implemented as a constant variable. However, getting rid of the timeout variable would come with very detrimental effects. We’ll go into that a bit more in the next section.

# The cost of hash functions

Source: [Yellow Paper](https://github.com/ethereum/yellowpaper) and https://ethereum.stackexchange.com/questions/3184/what-is-the-cheapest-hash-function-available-in-solidity/3200

The hash function that you decide to use does make a difference on the gas costs. Our hash function uses keccak256, which is the cheapest you can get.

| Hash Function | Amount of Gas  | 
|---------------|----------------|
keccak256 (also known as sha3*, in Appendix G) | 30 gas + 6 gas/word | 
sha256 (see Appendix E, line 209) | 60 gas + 12 gas/word | 
ripemd (see Appendix E, line 212) | 600 gas + 120 gas/word | 

`*`: The sha3 function mentioned here is the Ethereum flavour of sha3 and not the universally standardized version. This is the reason for the new name adoption of keccak256 instead. In web3 documentation, this is explicit in the [web3.utils.soliditySha3](https://web3js.readthedocs.io/en/1.0/web3-utils.html?highlight=sha3%20#soliditysha3) library, where it specifies the same hashing algorithm as Solidity would use.

# `bytes32` vs `string`

If you are using smaller string types, you can use bytes32 to store your data instead. While this does not affect our contract (we were already using byte types), using byte over string can help you save gas spent on storage.

# `uint8` *sometimes* uses more gas than `uint256`

[Source.](https://ethereum.stackexchange.com/questions/3067/why-does-uint8-cost-more-gas-than-uint256)

In solidity, 256bit/32 byte words are considered the “default” base unit. All operations are considered in these units. In many cases, `uint8` is cheaper than `uint256`: 

* If it is within a struct, reads and writes will be batched, as they are in tightly packed arguments. 
* If said `uint8` variable is not modified often, it is much less expensive to store in 1 memory slot as opposed to multiple. 

However, in some cases, `uint256` is cheaper than `uint8`. To make use of this, it would have to be tightly packed.***

# Tightly Packing Arguments

[Source.](http://solidity.readthedocs.io/en/latest/miscellaneous.html)

If you are accessing storage variables, the compiler may combine multiple reads and writes in a single operation. To make use of this technique, declare all the variables of the same type consecutively. In our STKChannel constructor, we have the following setup:

```solidity
channelData_.userAddress_ = _from;
channelData_.signerAddress_ = _addressOfSigner;
channelData_.recipientAddress_ = msg.sender;
channelData_.timeout_ = _expiryNumberOfBlocks;
channelData_.token_ = STKToken(_addressOfToken);
```

In this example, the changes to `userAddress`, `signerAddress`,and `recipientAddress` occur consecutively, followed by an initialization for timeout and initialization of the token. The consecutive assigning of the three addresses allows the Solidity Compiler to optimize the reads and writes.

Whenever our code need to access storage variables through resetting a value or altering a value in storage, we make use of this structure. The precise amount of gas it saves would depend on your contract.

# Optimization at the cost of readability

Optimization done is often at the expense of readability. In other words, you pay less for execution, but your code becomes slightly less readable.

Our `timeout` variable was used to determine whether the channel was in the contest period. It would be set when the channel is initialized, and determine the number of blocks between our closing and settling period.

To change the length of the contest period, this would require changing the `timeout` variable. It would have been possible to deploy a new library for the different timeouts we wanted to use. For instance, for testing purposes, the timeout is often set to 2, whereas the app implementation is set to 10. Removing this variable would have saved about 20000 units of gas. However, this would be incredibly confusing if the core channel library code had changed. We would have to push multiple versions of the code onto the chain and distinguish which contract maps to which timeout length. This would lead to more confusion and increase the chance of errors, so we opted to not make this change.

# Use of Libraries

If your code requires instantiation of data objects or structures that are identical, you may want to use Libraries to help you more efficiently instantiate. In our code examples, we make use of `STKChannelLibrary` to instantiate `STKChannel`.

For each user, instead of declaring and initializing new data structures for `STKChannel`, we implement the `STKChannelLibrary` to take care of the instantiation of the storage required for operation of the channel.

# In conclusion:

We’ve talked about a lot of different things that helped us (or helped others) with minimizing the gas cost, so to sum it up:

* Transaction costs include execution costs
* Use the ETH Gas Station to get a more precise ETH/Fiat amount spent on gas costs
* Analyze your code and try to move data off-chain
* Use global variable constants where applicable
* `keccak256` over any other hash function
* `bytes32` over `string`
* `uint8` sometimes uses more gas than `uint256`
* Use tightly packed arguments
* If some optimization makes your code unreadable, think twice
* Use solidity libraries for code reuse