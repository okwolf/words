---
layout: post
title: Chain pulling
description: Basic block blunders
image: /images/blockchain.jpg
---

> "Just put it on the blockchain, and it will be secure."

Rachel discretely rolled her eyes: "You should know that won't solve the problem, and in fact, will create more problems for us."

George persisted:

> "Nonsense! My nephew keeps all his money in Bitcoin, and he hasn't lost a single byte!"

"Satoshi," Rachel corrected him.

> "Huh? What does sushi have to do with it?"

Rachel was depleted, "It's the unit of... never mind, George."

It was not her first argument over the blockchain as a panacea, and it certainly won't be her last. She had hoped the hype would die down after the previous market crash, but the allure was as strong as ever. Nowhere is the siren song of blockchain stronger and more dangerous than within high tech companies' walls. It's a cruel twist of fate that organizations so knowledgeable in one technology would be so naive and ignorant toward another.

Chasing the new and shiny object won't always lead you in the right direction. Occasionally, you'll be greeted by oncoming traffic, if you're not aware of your surroundings. I'm not saying you should be a Luddite who shuns anything new and feels morally superior. Instead, one should deploy new technology only when adequately understood, and it furthers your business goals, not for bragging rights.

## blockchain isn't Bitcoin

Before getting to when one should (and shouldn't) use blockchain technology, some background knowledge is needed. First of all, while the Bitcoin protocol may have introduced the concept of blockchain to many (indeed is the most popular form of one), they are each distinct with their own identities, and we ought to keep it that way. Some circles of excessive hype assume that because blockchain powers Bitcoin, and Bitcoin has seen massive growth since its introduction, using blockchain must be some secret to exponential growth. This correlation does _not_ imply causation. There are significant conceptual and technological differences between the two, which I will explain.

Bitcoin is an implementation. It was created to serve as a peer-to-peer payment network. Because of this, efforts to build on top of Bitcoin for other purposes are usually suboptimal. While there is support for primitive scripting, it is intentionally limited to what is needed for performing transactions and isn't even [Turing-complete](https://en.wikipedia.org/wiki/Turing_completeness). Even if you "fork" the Bitcoin code, you'll still start from a specialized codebase built and adapted to solve _money_ problems, instead of _your_ problems. Unless you're starting a competing cryptocurrency, this path is best eschewed.

## what blockchain really is

Blockchain, on the other hand, is more of an abstraction. As the name implies - blocks are involved - and they form a chain:

![](/images/chain-of-blocks.png)

Each block in the chain stores some amount of data (transactions in the case of Bitcoin), a link to the block that came before it, and some manner of verifying integrity for the entire chain of blocks back to the first block (called the genesis block). Thanks to modern cryptography, the verification process is much cheaper and easier than adding new blocks to the chain. This arduous process is known as mining, although it doesn't involve removing anything from the ground to add to the blocks' chain.

Rewards are given to incentivize doing the strenuous work of adding blocks. This _literally_ compensates for the difficult work that provides the security inherent in a blockchain, leading to a very competitive race to be the first finding a valid new block, including as much additional data requested by users as possible, and reaping the spoils of that victory. Additions to the chain are broadcast far and wide, and thanks to a built-in bias, the longest valid chain always wins, protecting the whole system's integrity.

The blockchain can behave like a database, a filesystem, or any other persistent storage. The data in each block uses whatever format makes sense for your use case. The key differentiators are the strong guarantees made using cryptography that protect the integrity of both existing data and newly added data. This database operates in an append-only fashion, similar to how application logs or version control systems work. For anyone familiar with Git, this is like all users sharing one branch rebased continuously to and from many remotes, including an automatic way of resolving conflicts by choosing which commits to trust.

[![](/images/do-i-need-blockchain.jpg)](https://twitter.com/vgcerf/status/1019987651301081089)

## what blockchain shouldn't be used for

Blockchain isn't a product feature one adds to get market buzz or investment. Or at least, it shouldn't be. It's a much more strategic weapon, not deployed for mere sex appeal. Suppose your use case involves creating a blockchain within a private network. In that case, you will need to evaluate if your network will contain enough nodes to provide the same security benefits as a large public one like Bitcoin. Below a certain threshold network size, the benefits aren't worth the hassle, and your data is only as secure as your network's weakest link. Attacks on a network of this size can easily overwhelm the minimally distributed nodes, once the network is penetrated. Likewise, if the system you are building relies on a central point of authority (and failure), you will have a bad time attempting to use blockchain.

## what blockchain can be used for

I do not intend to rain on every single blockchain parade here. There are success stories of large financial institutions using blockchain, but most private networks aren't a good fit for the reasons outlined previously. However, there are several public marketplaces with distributed system characteristics that are ripe for chaining their blocks. Two of them are already used every time you visit a secure webpage. First, there's DNS, which resolves domains like [`okwolf.com`](https://words.okwolf.com) to which server on the internet to send your request. When establishing a connection with that server, it responds with an attempt to certify it's authenticity using a certificate. In turn, this certificate is verified using other certificates already trusted because they come from certificate authorities. Distributed records of trusted ownership like these are desirable to protect against tampering and are proper candidates for using a blockchain for storing those records.

We need not limit our applications for blockchain to marketplaces that involve exchanging ownership, however. Some of the more intriguing applications involve modernizing how our government operates. Consensus protocols built on top of blockchains could be used for electronic voting, an enticing proposition in the current [pandemic environment](/working-remotely-well). Smart contracts could enforce fairness where corruption has taken hold. Unfortunately, this system won't be perfect when run by humans, flawed as we are. Likewise, Bitcoin wallets are only as secure as their private key, the protection of which is entirely up to the user. Still, using blockchain can be a massive improvement over the way things currently operate in many areas of our life.

### The future is bright for all sorts of technologies, and that's not just pulling your chain.
