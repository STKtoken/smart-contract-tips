Safety has always been a major concern in the cryptocurrency industry. Once in a while, we’ll see cryptocurrency exchanges get hacked for millions of dollars. The concern of security is among one of the top reasons why people are reluctant to get into the cryptocurrency space. At STK, the security and the safety of your assets is always our top priority. Instead of a traditional wallet, we made a choice to use the smart contract and payment channels for our app. Due to the security nature of these contracts, we have not pushed our contracts up to mainnet yet. They will be pushed on mainnet soon after auditing.

# Storing Crypto in a Wallet vs a Smart Contract

A crypto wallet is always generated using a private/public key pair. In other words, when you sign up for a new address with something like MyEtherWallet, you will get a “safety package” — containing your private key that you should be keeping safe. With MetaMask, you would have the mnemonic with the 12 seed words. If you forget these, you lose access entirely to your account. If you accidentally make your private key accessible to someone else, this person has full control over your funds.

A smart contract, on the other hand, is different. There is no explicit private/public key differentiation — the address of the contract is the contract’s public key, but the private key is never made available. Unlike a crypto wallet, a smart contract does what its assigned to do, through code. For instance, our STK-smart-contracts is set to interact with only STK tokens. If other ERC20 tokens are sent to the smart contract, they are lost. This contract does not accept ETH transfers — so, if a customer accidentally transfers Ether to the smart contract, our contract will explicitly refuse it and refund it back. The only thing a user would lose is the gas fees for execution of said operation. Our smart contract also specifies that only STACK and User’s address can settle a channel. The settle function controls the movement of STK Tokens through a payment channel. Due to the restriction of changing the state of the channel, once an IOU has been submitted to change the state of a channel, no one can control the refund or transfers of tokens except yourself. The cryptocurrency you own is completely within your control.

#  Transactions are safer with state channels

In the initialization of a state channel, there are a few addresses that we make use of:

* user address: the address of the wallet in which your funds are stored. When a channel is settled and the funds are returned back to the user, the tokens/ETH will be returned to this address.
* signer address: the private and public key pair generated on your device for signing purposes `*`
* recipient address: the address of the STACK’s wallet. When a channel is settled, the IOU amount will be transferred to STACK.
* channel address: the address of your personal state channel. When you deposit funds, this is where your funds are stored.

`*` For you to sign transactions, the app automatically generates private and public key pairs. All transactions (IOU’s) are signed with this private key. These are only ever used to sign transactions that are sent to the smart contract. STACK never interacts with your signer’s private key. Notice your signer’s key never holds funds. Its only purpose is to sign off-chain transactions.

With the following setup, transactions are safer with state channels, because there is no single point of failure. No matter what tokens there are in our payment channel, the funds inside it are always safe. Even if STK’s backend or database fails or goes down, the user is in complete control of the funds within the state channel.

# Network Congestion

Back in December of last year, when Cryptokitties was released officially, the sheer number of transactions on the Ethereum Network skyrocketed and caused complete congestion. A few people in the Reddit AMA asked about the effects of network congestion on our state channels.

We sign the IOU’s off chain with the signer’s public and private key pair. These do not need to interact with the network. When you decide to settle a channel, these transactions will be pushed on-chain. During the settle period, you have the option to transfer any unused tokens back to your address. Your transaction amount will also be transferred to STACK. In other words, the end user is not affected by congestion on the network. This is ultimately safer for customers, because they are not affected by the volatility and the ever-changing block times on the network. To read more about how the IOU and signing off-chain transactions work, check this out.

If you’re intrigued, feel free to check out our crypto beta demo located here: https://www.youtube.com/watch?v=Yhmjks3eFIA, where we go through a live purchase at a local coffee shop.