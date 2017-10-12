## Preface

A blockchain is a mesh network of computers linked not to a central server but rather to each 
other. Computers in this network define and agree upon a shared state of data and adhere to certain constraints imposed upon this data.

This shared state is simply a distributed state machine, with each "block" making a change to the
current, known, shared state.

### A word about Bitcoin

The Bitcoin project implemented the first large-scale blockchain.

Bitcoin’s blockchain is "simple" in comparison to most of the other blockchains that exist today. For a beginner's look at Bitcoin, see this [guide](https://www.weforum.org/agenda/2016/06/blockchain-explained-simply/).  

For the purposes of this article, we will mainly look to Bitcoin's blockchain when discussing aspects of blockchain architecture in general. However, the
architectural components of **transactions**, **blocks**, **mining**, and **consensus** can be
generalized and implemented in many different ways, leading to various possible blockchain projects. These projects usually involve creating other cryptocurrencies (or [_alt-coins_][coin-market-cap]), but the diversity in blockchain-related project ideas is growing quickly.

## Transactions

[Transactions][transactions] are the things that give a blockchain purpose. They are the smallest
building blocks of a blockchain system.

Transactions generally consist of a recipient address, a sender address, and a value. This is not too
different from a standard transaction that you would find on a credit card statement.

A Bitcoin transaction moves the value of some bitcoin from one address to another address.

A transaction _changes the state_ of the agreed-correct blockchain. A blockchain is a shared, decentralized, distributed state machine. This means that all [nodes][nodes] (users of the blockchain system) independently hold their own copy of the blockchain, and the current known "state" is calculated by processing each transaction in order as it appears in the blockchain.

Transactions are bundled and delivered to each node in the form of a block.
As new transactions are distributed throughout the network, they are [independently verified and
"processed"][protocol-rules-transaction] by each node.

This constant [movement of coin][coin-story] is what constitutes the data within any blockchain architecture, while the ways in which transactions are handled and verified varies by implementation.

### Format of a Transaction

Transactions contain one-or-more inputs and one-or-more outputs.

An [**input**][transactions-input] is a reference to an _output from a previous transaction_.

An [**output**][transactions-output] specifies an amount and a [address][addresses].

An input always references a previous transaction's output. This continual pointer of inputs to previous transactions outputs allows for an uninterrupted, verifiable stream of value (represented by the bitcoin currency) amongst addresses.

![transaction](https://raw.githubusercontent.com/pluralsight/guides/master/images/87988388-722d-4728-83eb-317ae281c3ff.png)

Also included in the input data structure is a _scriptSig_.
This is a cryptographic signature that proves that the creator of this transaction is allowed to
create it given its inputs.

Remember than an input is a reference to an output on a previous transaction.

The _scriptSig_ contains the address _in the referenced transaction's output_, and an [ECDSA
signature][ecdsa] of this current transaction.

This proves that the current transaction was created by the owner of the output of the referenced
transaction that this is spending from.

### How did it start?

In the beginning, there was one transaction.

It had no inputs, it just had an output crediting an address (owned by Bob) some value.
There were no inputs, because, well, an input references the output of a previous transaction.
This is the first transaction. So no inputs.

Eventually, Bob will want to spend that value by transferring it to someone else (Alice).
*So Bob creates a transaction.*

That transaction has one input, which is a reference to the output of that very first transaction -
remember: Bob controls the keys to the address that the original output credits.

That transaction has one output, which is some public address to which Alice holds the private key,
and an amount of currency to be sent. 

Now Alice controls this value.

By virtue of her private key, she is able to sign off on another transaction that sends up to the amount she received to a different address, controlled by some other entity. 

It is this simple value-transfer mechanism using "transactions" over time that create a blockchain. As each transaction occurs, it gets added to the blockchain ledger, leading us to the next topic: blocks, miners, and verification.

## Blocks

Blocks are data structures whose purpose is to bundle sets of transactions and be distributed to all nodes in the network. Blocks are created by [miners][miners] (discussed in more detail below).

Blocks contain a **block header**, which is the metadata that helps verify the **validity** of a block.

Typical block metadata contains:

* version - the current version of the block structure

* previous block header hash - the reference this block's _parent block_ 

* merkle root hash - a cryptographic hash of all of the transactions included in this block

* time - the time that this block was created

* nBits - the current _difficulty_ that was used to create this block

* nonce ("number used once") - a random value that the creator of a block is allowed to manipulate however they so choose

These 6 fields constitute the block header. The rest of a block contains transactions that the miner has chosen to include in the block that they created.

Users create transactions and submit them to the network, where they sit in a pool waiting to be included in a block.

![block](https://raw.githubusercontent.com/pluralsight/guides/master/images/f874e6a3-d3b0-49b2-86e4-83ed29ef5d09.png)

It's important to realize that each miner (and more generally, each user of a blockchain) is allowed to act however they want within this blockchain system. Consensus rules dictate that only valid changes to the blockchain will be accepted by everyone else. This results in a system that economically guarantees that only valid blocks will be worked on, submitted to the network, and accepted by the greater community.

Gaming the system is exceptionally expensive in terms of computing power and offers little reward.

Blockchains are probabilistic systems, by design. Nodes, or the computers in the network, independently decide and concur upon which "chain of blocks" is the longest and most valid. As a block is created and set around the network, each node processes the block and decides where it fits into the current overarching blockchain ledger.

![blockchain](https://raw.githubusercontent.com/pluralsight/guides/master/images/8cd8b94f-d05f-41e8-a0f1-70853f390094.png)

Within the context of a blockchain, there are a few different [types of blocks][protocol-rules-blocks]. 

- Most blocks simply extend the current main blockchain. These are called "main branch blocks". 

- Some blocks reference a parent block that is not at the current blockchain tip. These blocks are called "side branch blocks". 

- Some blocks reference a parent block that is not known to the node processing the block. These are called "orphan blocks."

Side branch blocks are particularly interesting. They might not currently exist in the main branch, but if more work is done on them (meaning other blocks are mined that reference them as a parent), there is the possibility that that a particular side branch will be [reorganized][chain-reorg] into the main branch.

This reorganization happens because the "main" branch of the blockchain is the one that has had the most work done on it.

As new blocks are appended to the blockchain, it becomes increasingly difficult to "overwrite" existing blocks because the most valid chain is the one that has had the most work done on it.

## Mining

So far, we've danced around the details of "creating a block", or [mining][miners]. Mining is the process of putting in real-world work (in the form of electricity) to create a valid block that will be accepted by the rest of the network.

Miners are the equivalent to the processing network of a credit card company. They take pending transactions, verify that they are cryptographically accurate, and package them into blocks to be stored on the blockchain.

At a high level, the process of mining involves [hashing][hashing] a potential block, checking to see if the hash fits the current [difficulty rules](https://en.bitcoin.it/wiki/Difficulty), and if not, changing the **nonce** in the block header and hashing the block again.

### Hashing 

Hashing functions have a few properties that make them desirable for creating [proof of work][proof-of-work], a key concept in the Bitcoin network specifically:

* Hash functions turn an arbitrarily-large piece of data into a fixed-length hash output

* They are one-to-one: the same input will always provide the same hash output

* They are one-way functions: it's impossible to "work backwards", and reconstruct the input given a hash output.

Some common hashing schemes can be found [here](https://en.wikipedia.org/wiki/List_of_hash_functions). For reference, Bitcoin uses the SHA-256 hashing algorithm for proof of work.

Go [here](https://www.coindesk.com/short-guide-blockchain-consensus-protocols/) for a closer look at alternatives to the proof of work architecture.

Due to hashing, editing even a single bit of the block header will result in a different hash. Therefore, changing the nonce will create a new hash value to cross-check with the current difficulty rules. This process must be done over and over for each new potential block, until a valid hash is found.

When a miner DOES find the right configuration of data within a block template that results in a valid hash, it's trivially easy for all other nodes to perform the same hashing function on that block, and verify that it does, in fact, result in a valid hash.

### Mining Incentive

Miners must have some incentive to put in the work required to create a valid block. In Bitcoin, this incentive is the creation of new coins via the [coinbase transaction][coinbase]. This is a special transaction that exists in every block, which has no inputs, and has a single output pointing to an address that (presumably) the miner controls. If a miner's block is accepted by the network, their address is credited with these new bitcoin.

Mining "difficulty" is typically adjusted over time as the network grows. Difficulty is a consensus rule that defines how much work must be done to create a valid block. Blocks are valid when their header is hashed to produce a result that proves that enough work has been done.

How is this proved? The hash must fit a certain format. In Bitcoin, at a high level, this format is "the block hash must start with a certain number of zeros."

[Bitcoin block 488485][block-488485], for example, has a hash of `0000000000000000008c8aa76452c5b0b422be963b1f9813538ec374178a6826`.
This is a hash that begins with 18 zeros. Considering the properties of hashing functions defined above, it is extremely difficult to find a resultant hash from an arbitrary block structure that begins with the current difficulty level of 18 zeros.

Any other actor in the network can trust that if they receive this block, it must have taken lots of computating power to find the right combination of data to generate it. To compensate the miner(s) for their time and energy, the overall network agrees that the miner is allowed to include a coinbase transaction minting new coin as their reward.

## Consensus

All of these previous concepts of independent nodes checking and verifying the validity of transactions and blocks is called [consensus][consensus].

The consensus of a blockchain is realized in a codified set of rules that everyone is playing by.
These rules are entirely self-enforced. As the blockchain network grows larger, and more nodes and miners participate in it, the overall consensus grows stronger due to the sheer number of actors that are enforcing their own rules (that everyone else is enforcing, as well).

For example, take one of Bitcoin's consensus rules: the amount of the coinbase transaction.
The rule that everyone follows is based on code that dictates that the value of the transaction should be cut in half every 200,000 blocks.

The first 200,000 blocks produced a reward of 50 BTC (Bitcoin). Blocks 200,001 through 400,000 produced a reward of 25 BTC. At the time of writing, Bitcoin's blockchain is around 488,000 blocks long, which means that each block these days produces 12.5 BTC.

> If a miner decided to produce a block today that minted 15 BTC as their reward, instead of 12.5 BTC (remember, each miner can create whatever kind of block they want), the rest of the network would receive their block, run it through their own consensus rules, and ignore that block because it doesn't fit the mold.

The block would simply be discarded by everyone else because it's not valid, and no one would accept the attempt by this malicious miner to receive a reward larger than the current rate.

As an aside: this "reward halving" is what gives Bitcoin its upper limit of about 21 million coins.
A bitcoin is divisible to eight decimal places, so eventually the reward will halve its way to 0.

The _entire network independently agrees_ on what the reward for finding a new block should be.
Trying to play "outside of the rules" is not _illegal_ or even _wrong_ (remember, anyone can do whatever they want). However, if you want your transaction or block to be accepted by the network, you must follow the same rules as everyone in the network, otherwise you run the risk of not matching consensus standards and getting your verification ignored. Thus consensus adds another layer of security to blockchain transactions.

_Side note: the [51-attack](https://medium.com/@fhansmann/demystifying-the-51-consensus-attack-942252090b33) is a common point made against proof of work protocols._

## In Closing

A blockchain is simply a distributed data structure, that is built linearly, over time, and is independently verified and audited by all actors in the network.

In general, blockchains contain transactions packaged into blocks that are mined using significant resources, and new "tokens" are created as a result of this mining.

The network-at-large cryptographically verifies that all transactions are legitimate, and uses consensus rules to determine what the valid blockchain contains.

As a consequence, blockchains introduce a revolutionary new way to create systems that are free from reliance on any centralized trusted entity to dictate truth.

______

I hope you found this guide informative and engaging. Please leave me your comments and feedback in the discussion section below. Thanks for reading!

[addresses]: https://en.bitcoin.it/wiki/Technical_background_of_version_1_Bitcoin_addresses "Bitcoin Addresses"
[chain-reorg]: https://en.bitcoin.it/wiki/Chain_Reorganization "Chain Reorganization"
[coin-market-cap]: https://coinmarketcap.com/ "Coin Market Cap"
[coin-story]: https://en.bitcoin.it/wiki/Coin_analogy "The Coin Analogy"
[coinbase]: https://en.bitcoin.it/wiki/Coinbase "Bitcoin Coinbase"
[consensus]: https://bitcoin.org/en/glossary/consensus "Consensus"
[block-488485]: https://blockchain.info/block/0000000000000000008c8aa76452c5b0b422be963b1f9813538ec374178a6826 "A Bitcoin Block"
[blocks]: https://en.bitcoin.it/wiki/Block "Bitcoin Blocks"
[ecdsa]: https://en.bitcoin.it/wiki/Elliptic_Curve_Digital_Signature_Algorithm "Elliptic Curve Digital Signature Algorithm"
[hashing]: https://en.bitcoin.it/wiki/Hash "Bitcoin Hashing"
[miners]: https://en.bitcoin.it/wiki/Mining "Bitcoin Miners"
[nodes]: https://en.bitcoin.it/wiki/Full_node "Bitcoin Nodes"
[proof-of-work]: https://en.bitcoin.it/wiki/Proof_of_work "Proof of Work"
[protocol-rules-transaction]: https://en.bitcoin.it/wiki/Protocol_rules#.22tx.22_messages "Transaction protocol rules"
[protocol-rules-blocks]: https://en.bitcoin.it/wiki/Protocol_rules#Blocks "Block protocol rules"
[transactions]: https://en.bitcoin.it/wiki/Transaction "Bitcoin Transactions"
[transactions-input]: https://en.bitcoin.it/wiki/Transaction#Input "Transaction input"
[transactions-output]: https://en.bitcoin.it/wiki/Transaction#Output "Transaction output"

[block]: block.png
[blockchain]: blockchain.png
[transaction]: transaction.png