## Preface

### The Study of Codes

By now, nearly everyone has heard of Bitcoin. In simple terms, Bitcoin is digital cash - a
monetary system that melds and anonymity of cash with the convenience, freedom, and
power of the internet, allowing you to send and receive funds around the world without relying on a
central authority such as a bank or a government.

Bitcoin's power comes from an invention called "blockchain." A blockchain is a distributed
ledger that uses the combined power of computers participating in it to operate, which is what
allows for the decentralization of authority in the network.

From a technical perspective, there are many interesting concepts that make up a blockchain -
distributed databases and consensus (or, decentralized governance) are both fascinating.
Fundamentally, however, the basic building block for blockchain systems is cryptography.

Cryptography is the study of codes - writing codes, solving codes, and manipulating codes. Yes, this
includes the [super secret spy decoder ring you had as a kid][decoder-ring], and even pig latin! 
Cryptography
is an ancient study that has existed for thousands of years, most often in the form of ciphers. It
is generally believed that ciphers were invented as a method for concealing the information
contained in a message from any person who didn't intentionally receive it.

### Atwhay Aboutyay Ymay Omputercay?

Cryptography and computers have had a competitive relationship since the beginning of digital
computing. During World War II, the United Kingdom invested heavily in deciphering Axis communications. With deciphering becoming too difficult to be performed by hand, a race began
to develop a machine that was capable of deciphering codes faster than any human. This eventually
led to the development of [Colossus][colossus], the first digital programmable computer.

Since then, the war of computers and cryptography has only elevated. In what many describe as an
"arms race," many of the computer systems we take for granted rely heavily on cryptography, while
the invention of more powerful computers forces previously state-of-the-art ciphers into
obsolescence.

Without cryptography, it would be impossible to encrypt data, ensure secure communications, or even confirm
that you're browsing a safe site - yes, I'm looking at you, [little-green-lock-in-my-browser][ssl].
We're going to focus on the cryptography that powers a few parts of Bitcoin, a cryptographically revolutionary system.

## Public and Private Key Pairs

### What's in a Pair?

If you have used Bitcoin at all, you have probably heard of a [private key][private-key]. Private keys are
vital to the Bitcoin system. They are the mechanism for proving ownership of bitcoin. This is what
allows a user to authorize a transaction on the network. Private keys exist in many forms outside of
Bitcoin for many purposes, and most people who are familiar with them from a previous experience
would know them as a way to send [encrypted messages][pgp].

For every private key that exists in Bitcoin, there is a 1:1 relationship with something called a
public key. As you can imagine, a private key is intended to remain private and shared with no one,
under any circumstance. A public key, in contrast, can be shared with anyone - there is no danger
in me placing my public key on my website, for example, or to e-mail it to a client to receive
payment for some activity. In this sense, you can think of public and private keys like a username
and password - one allows you to identify yourself, while the other allows you to prove you are that
person. However, unlike a password, **a private key can never be reset or recovered if lost.** Thus, a private
key is an extremely important piece of data and should be protected perhaps to the point of paranoia.

Due to the 1:1 guarantee, public and private keys share a cryptographic relationship that links them together.
In Bitcoin, private keys produce a public key via an [Elliptical Curve Digital Signature Algorithm][ecdsa],
or ECDSA. A private key that is an input for that algorithm will always produce its corresponding
public key. However, the public key can never be reverse-engineered to produce its corresponding private
key due to the one-sided nature of this algorithm.

A Bitcoin private key is usually a 256-bit number, which can be represented a number of ways.

```
Private Key:
KxeNcRw8mBfyLrnnXQymQkogLjvmn6uJCmSWLRmZ6Mt3Hzfgo1mY

Deposit Address:
1MnU3iyTeej69DKGGKo6vU3H3dKKZ9ZL6u
```

### That's a Lot of Keys!

Public and private key pair cryptography is what powers the address system in Bitcoin - the
cryptocurrency equivalent to a checking account. A new address can simply be generated
programatically. Whenever a new one is required, I can use my interface of choice (perhaps a
Bitcoin wallet) and make one.

Usually, when I introduce someone to Bitcoin, their immediate question is "What if someone guesses
my private key?", to which I reply, "Well, that is highly, HIGHLY improbable."

But how improbable?

Well, for a frame of reference, the total address space of Bitcoin is 2^160. That is this many:

```
1,461,501,637,330,902,918,203,684,832,716,283,019,655,932,542,976
```

Good luck visualizing that. For an even further head trip, consider that there are an estimated 2^63
grains of sand on Earth - this includes sand on beaches and underneath the ocean. 2^63 * 2^63 =
2^126.

This means, for every single grain of sand on Earth, you could create a new Earth, and then count
all of the grains of sand on all of those Earths - and still not even come close to the address
space of Bitcoin.

Clearly, in this case, cryptography obtains security through very big numbers. I could try to guess private
keys over and over again, using any means known to mankind (including computers, of course), for
many millions of years, and it is simply not going to happen.

This is wonderful because it allows all kinds of applications to be created using Bitcoin - for
example, Decent created a prototype platform for charity that allows donations to be made using
cryptocurrencies. There is no need to place a limit on the number of campaigns that could be
created, or limit the number of deposit addresses a user can have, because there are a practically
infinite number of addresses.

> But What If Someone Just Gets Lucky and Guesses My Key?

Seriously, you're not getting it. **Really**. **Big**. **Numbers**. Re-read the last section.

In closing, public and private key pairs are a fundamental tool in cryptography that have many uses.
In Bitcoin, the use is to confirm ownership and create a large pool of addresses available for use.

Remember, because of the large number of keys, it is safe to assume that any key I generate is
mine and only mine. Thus, the Bitcoin system requires no further proof of ownership. Otherwise,
in other systems, we'd traditionally use our identity as proof of ownership. Since Bitcoin removes this requirement,
cryptocurrencies promote anonymity just like physical cash, but perhaps to a greater degree.

## Hierarchal Deterministic Wallets

### With Great Power Comes Great Responsibility

Now that we have covered key pairs in Bitcoin, we can cover the unique way in which they are
created. Private keys give us complete control over our finances - this is the purpose
of Bitcoin. However, in many ways, having complete
control is frightening. If a private key is lost, the funds associated with it are gone, forever. If
someone steals a private key, they have complete access to the funds, and theoretically are the new
owner of said funds. And, there is no recourse in the event of private key loss, theft, or any other
issue.

That is a lot of pressure on a Bitcoin user. Of course, a user could do what they do with any other
valuable data, and create a backup private key. However, private keys should be very closely
guarded. Private keys should *never* be backed up on a cloud server or transmitted through internet
communication of any kind. Storing them on a computer or cell phone is dangerous, for if a system
is compromised by a thief or attacker, all of the funds associated with private keys on the system
would be free for the taking. A private key could be stored in a fireproof safe and buried 3 meters
underground in a remote wilderness, but since using Bitcoin generates
hundreds or thousands of private keys for a single user. How can someone possibly keep track of all
of those keys? How can something be stored without storing?

This is where the "hierarchal deterministic wallet" or "HD wallet" comes in to save
the day.

### That's a Mouthful

Let's break it down the HD wallet word by word. To be "hierarchal" is to be arranged in an order or
rank, or to be placed in sequence. Sequence is very important in the HD wallet system. "Determinism"
is a fancy way of saying "cause and effect". There will always be an outcome for any event in any
system. And a "wallet" is an interface for using Bitcoin that allows you to access, send, and
receive bitcoins. Some notable wallets include [Electrum][electrum], [Mycelium][mycelium], and
[bread][bread].

So, an HD wallet is a Bitcoin wallet that generates a sequence of private keys, where each private
key is determined by the previous or "parent" key in the sequence. This system was added to Bitcoin
[BIP-32][bip-32]. A BIP is a "Bitcoin improvement proposal" or the format for making changes to
Bitcoin.

Similar to the relationship between a private and public key, the private key sequence that results
from using an HD wallet is defined by a one-way relationship between inputs and outputs of an
algorithm. At a high level, Bitcoin private keys are generated using a "pseudorandom number
generator." Due to programs being so strictly defined and process-oriented, it is very
difficult to capture the human concept of "randomness" in a computer. So, oftentimes these random
generators require an input of entropy called a "seed". This is sometimes represented by
random mouse movements or key presses in other programs, and a greater or "more random" seed of
entropy is required for "more random" generation. A quirk of pseudorandom number generators is
the result of algorithmic thinking - the generator follows a sequence of steps to produce an output.
As such, with many algorithms, there is one and only one output for a single input. With the HD
sequence, we use the outputs (private keys) of previous number generation as inputs for new number
generation, and create a "chain" of keys.

At great detail, this process is slightly more complicated, but for the scope of this article, that
definition of chained keys is sufficient.


![hd wallet](https://raw.githubusercontent.com/pluralsight/guides/master/images/c2327d40-3abb-490d-bb12-9d6c68fb5a0d.png)


Why is this important? Well, consider the following. It is known that in a sequence of keys, each
private key is generated based on some input, which is a preceding key. This means that by simply
having access to the very first key in a sequence, every following key in the sequence can be
generated. In theory, a "backup" of the entire collection of keys is available by remembering only a
single one!

### Improving the Improvement

BIP-32 defined a system of sequentially generating private keys for use in wallets. This way, users
of Bitcoin can have a sense of insurance from theft or data loss by storing a single "master" key.
However, there is one final problem - it is difficult to remember even a single private key. They
are long and unwieldy. This is a common problem in computer science - what is useful to a computer
system (large numbers, data sets, and lengthy strings) is often very annoying to human beings. For
the sake of convenience, software engineers often spend a lot of time creating ways to "abstract"
computer data into a more comfortable format for people to digest. This brings us to [BIP-39][bip-39],
which introduces a mnemonic seed phrase to the HD wallet protocol.

If you look at the codebase for a Bitcoin wallet, you will probably find somewhere a file called
"words.txt", "wordlist.txt", or similar. This is the list of words used to generate the mnemonic
seed for an HD wallet. It contains 2048 words and is available in many different languages.

This process is fairly simple. When an HD wallet is a created, it generates "computer generated
entropy." This entropy is converted into a sequence of numbers between 0 and 2047, and those numbers
are used as indices to take a sequence of 12 numbers from the wordlist and combine them to form a
seed phrase.

```
bamboo minor dirt drop mom october theme alpha clump filter vessel twelve
```

When this process is completed, the resulting seed phrase is used as the seed input for the HD
wallet, and all of the private keys your wallet generates can be recovered simply by remembering
that phrase.

### More Big Numbers

When I first learned of the mnemonic key process, I was worried that it was less secure than
Bitcoin itself. I was correct. The total number of seed phrases that can be generated is 2048^12,
or 2^132, which is less than the total address space of 2^160 and thus easier to brute force.
However, 2^132 is still a monumentally huge set (in fact, it is still greater than the total sand
particle count we discussed earlier) and generally secure enough to use. 

For those who are interested, 2^132 looks like this:

```
5,444,517,870,735,015,415,413,993,718,908,291,383,296
```

What is truly valuable about this process isn't so obvious. What is really going on here is the
conversion of computer data (in this case, randomness) into a
convenient, **human-friendly** format. This seed phrase is not only memorable, but communicable (good
luck telling someone a private key over the phone) and thus has a far greater usability to a person
than a private key does by itself. Once again, much of the time software engineers spend on
perfecting software is the abstraction of computer-friendly data to human-friendly data. Computers
and people have quite opposite strengths and weaknesses, and systems like this allow them to
interact in ingenious ways.

In summary, the hierarchal deterministic wallet or "HD wallet" system was added to Bitcoin via
Bitcoin Improvement Proposals (BIP) 32 and 39. The system allows users to "store" their entire
collection of private keys by remembering a single seed phrase, which can be used to recover
their private keys in the event of loss.

## Hashcash

### The Original Anti-Spam

Hashcash was invented by Adam Back, a cryptographer, in 1997. It was invented as a method of
deterring e-mail spam and preventing DDoS (distributed denial-of-service) attacks. Hashcash protects
systems from such attacks by acting as a bottleneck for computer activity, using a concept called
proof-of-work (sometimes referred to as POW in the cryptocurrency world).

In computers, certain activities are known as "cheap" or "expensive" based on the amount of
resources required to perform them. Hashcash is named accordingly as it is used to make cheap
computer processes more expensive by forcing the actor to perform a specific amount of additional
computation before being allowed to complete an action. For example, lets consider e-mail. It is
remarkably cheap for a modern computer to process and send text. This is a problem for would-be
victim e-mail servers, as even a slow and simple computer is capable of overloading the victim with
messages. By forcing a sender to perform a more expensive process, a sender cannot feasibly spam the server. As a result, the server can assume that the
sender is well-intentioned.

Hashcash achieves its bottleneck in any situation by adjusting the
"difficulty" level or, the amount of processing power required to complete a transaction successfully.

### Why So Difficult?

So how is Hashcash used in Bitcoin? In some ways, Hashcash is everything in Bitcoin! When you hear
about mining blocks, processing transactions, and sending or receiving bitcoins, you are actually hearing
about Hashcash.

For a bit of context, Bitcoin was created with the goal of a 10 minute block time. This means the
software attempts to allow for a new block (a bundle of transactions) to be added to the network
each and every 10 minutes, no more and no less. The 10 minute limit was chosen to allow adequate
time for the entire Bitcoin network to [remain stable][block-time] and in-sync.

### Battle of the Supercomputers

The 10 minute block time is protected using Hashcash and its adjusting difficulty. Bitcoin works
because network participants (miners) process transactions and bundle them into blocks. For these
blocks to be valid and added to a blockchain, the processor must complete a significant amount of
processing work (proof-of-work). This work is proven using Hashcash. 

When Bitcoin first started,
competition on the mining network was low and it was very easy to mine for rewards using simple
hardware like a laptop. As competition grew, miners started purchasing advanced hardware (first
high-end graphics cards, then ASICs, which are specialized hardware produced strictly to mine) that
would offer a competitive advantage over the rest of the network. This hardware race has resulted
in a worldwide network of miners using very, very powerful hardware to remain competitive. Luckily,
Hashcash and its adjustable difficulty allows Bitcoin to survive without being more or less
"overloaded" by the massive amount of processing power its network holds.

Bitcoin is very resilient. It calibrates itself over time as more or less power is available to the
network to prevent the block time from drifting away. For every 2016 blocks added to the blockchain,
a calibration occurs. Essentially, if the average block time is trending faster than 10 minutes,
difficulty is increased. If the average block time is trending slower than 10 minutes, difficulty
is decreased. This allows the network to scale with mining competition.

So, Hashcash is a cryptographic function used as a proof-of-work system. It was invented to protect
e-mail servers and websites but since has been used for many things, and has grown more famous as
the Bitcoin mining algorithm. With an adjustable difficulty, Hashcash allows Bitcoin to adapt over
time and remain stable with a growing network.

## In Closing

Cryptography is a fascinating art that blurs the lines between computers and reality. In most cases,
computers are expected to be perfect: all functionality can be boiled down to "true" or
"false" and the computer is always right. Cryptography introduces the concept of "good enough" computing, leveraging huge numbers and
processing power to create systems that are "good enough" to be secure and interface with the human world.

Bitcoin uses cryptography
everywhere, from its address system, to its user experience, and even mining. Hopefully now you see that cryptography is truly
the lifeblood of a blockchain.

If you're craving a more in-depth look at the actual architecture involved in the Blockchain, check out [this guide on hack.guides()](https://www.pluralsight.com/guides/software-engineering-best-practices/blockchain-architecture)!

_____


Thank you for reading this guide! I hope you found it engaging and informative. If you have questions or feedback, please let me know in the comments section below!

[bip-32]: https://en.bitcoin.it/wiki/BIP_0032
[bip-39]: https://en.bitcoin.it/wiki/BIP_0039
[block-time]: https://en.bitcoin.it/wiki/Help:FAQ#Why_do_I_have_to_wait_10_minutes_before_I_can_spend_money_I_received.3F
[bread]: https://www.breadapp.com
[colossus]: https://en.wikipedia.org/wiki/Colossus_computer
[decoder-ring]: https://upload.wikimedia.org/wikipedia/commons/thumb/5/5b/Captain-midnight-decoder.jpg/800px-Captain-midnight-decoder.jpg
[ecdsa]: https://en.bitcoin.it/wiki/Elliptic_Curve_Digital_Signature_Algorithm
[electrum]: https://www.electrum.org
[mycelium]: https://wallet.mycelium.com
[pgp]: http://openpgp.org/
[private-key]: https://en.bitcoin.it/wiki/Private_key
[ssl]: https://www.verisign.com/en_US/website-presence/website-optimization/ssl-certificates/index.xhtml
