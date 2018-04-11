---
layout: post
title: "Cryptocurrency course of Princeton university: Lecture 2 - How Bitcoin Achieves Decentralization"
date: 2018-04-04
categories: [courses, blockchain]
---

## 2. How Bitcoin Achieves Decentralization

### 2.1. Centralization vs. decentralization

Decentralization is not all-or-nothing. E.g. email uses decentralized protocol but dominated by centralized webmail services.

Aspects of decentralization of Bitcoin:
- Peer-to-peer network: open to every one and low barrier to entry.(High level of decentralization)
- Mining: open to anyone but inevitable concentration of power often seen as undesirable.
- Updates to software: core developers trusted by the community have great power.

### 2.2. Distributed Consensus

Key technical challenge of decentralized e-cash: `distributed consensus`.

Distributed consensus:
- The protocol should terminate and all correct nodes decide on the same value.
- This value must have been proposed by some correct node.

  ![]({{site.url}}/assets/image/BTC_peer_to_peer_sys.jpeg){:width="300px"}_Bitcoin is a peer-to-peer system_
  
Bitcoin is a peer-to-peer system. When Alice make a payment to Bob, he broadcasts this transaction (take Goofy transaction as eg) to all the Bitcoin nodes. Bon doesn't need to run a Bitcoin node to receive this transaction and once he gets into this network, he should be notified that the transaction is made so the consensus is reached in this peer-to-peer system.  

How consensus could work in Bitcoin: At any given time,
1. all nodes have a sequence of blocks of transactions they've reached consensus on
2. each node has a set of outstanding transactions it's heard about.

Why consensus is hard:
- nodes may crash
- nodes may be malicious
- Network is imperfect: not completely connected, faults, latency(notion of global time).

Bitcoin consensus works better in practice than in theory ironically. Some things Bitcoin does differently:
- Introduces incentives
	- possible only because it's a currency
- Embraces randomness
	- Does away with the notion of a specific end-point
	- Consensus happens over long time scales - about 1 hour

The Bitcoin consensus is partly a technical mechanism and partly clever incentive engineering.

### 2.3. Consensus without identity: the blockchain

Why don't Bitcoin nodes have identities:
- Identity is hard in a P2P system - Sybil attack
- Pseudonymity is a goal of Bitcoin

Key idea of Bitcoin consensus: implicit consensus:
- In each round, `random` node is picked to propose the next block in the chain.(this random process is based on `proof-of-power` that is introduced later)
- Other nodes implicitly accept/reject this block by either extending it or ignoring it and extending chain from earlier block.
- Every block contains hash of the block it extends.

![]({{site.url}}/assets/image/consensus_algorithm_bitcoin.jpeg){:width="400px"}_Simplified consensus algorithm of Bitcoin_
  
The simplified consensus algorithm of Bitcoin is described above, except the process of randomly picking a node for broadcasting new block by proof-of-power. We notice that there are 2 types of broadcasting:  1. At any time transactions are broadcasted to all other nodes and every node is constantly listening to the network to collect these new transactions. 2. Each round (10 minutes averagely, introduced later), a "randomly" picked node broadcasts it's new block containing all the outstanding transactions it's heard about. Nodes can check the validity of the transactions at anytime to decide whether accept the transaction(broadcasting 1) or the block(broadcasting 2).

Given this consensus protocol, let's imagine what a malicious node (Alice) can do. Suppose that now it is Alice's turn to propose a block:
- She can't simply steal a bitcoin of other users because she cannot forge their signatures as long as the security garantie of crypto digital signature is solid(the algorithm of secret key and public key).
- She hates Bob so she just denies all the transactions originating from Bob's address and she simply doesn't include them in the block that she proposes. This attack should be valid but it's no more than a small annoyance, because if the node who propose the next block is honest and the transaction of Bob will be included in this new block.
- `Double spending attack` is still the key issue that we need to consider. Assume that Alice is customer going to buy some products on the online store of Bob, who accepts bitcoin payment. That means when Alice buy some items, she will create a bitcoin transaction from her address to Bob's address.
	![]({{site.url}}/assets/image/bitcoin_double_spending.jpeg){:width="400px"}_double spending in bitcoin consensus_
	Suppose that a new block is extended containing this (green) transaction. Bob sees this new block and confirm this transaction. But if the next node that "randomly" chosen to propose a new block is controlled by Alice, who transfers that bitcoin C<sub>A</sub> to another of her address A' as the red transaction. Up to now these 2 blocks are actually both valid for a node that hears them later and can't not distinguish which one is a double spending attack (the receiving time doesn't help due to the network latency). 
	![]({{site.url}}/assets/image/bitcoin_double_spending_succeed.jpeg){:width="400px"}_successful double spending attack_
	Indeed if the next chosen node is still controlled by Alice or even bribed by her, it intentionally extends new block from the block with malicious(red) block. In bitcoin protocol, **honest nodes extend the longest valid branch** so new blocks will continuously extending from the lower branche which makes the upper correct block as a orphan block. This is a success attack of double spending because Bob has confirmed the transaction but the bitcoin is actually not transferred from Alice to Bob.
	Now let's see how Bob can protect himself from this attack. It's crucial for him to decide when to confirm the transaction. He can accept Alice's payment at the moment that his node hears the transaction broadcasted by Alice, but it's obviously not a safe choice because if it happens to be Alice to propose next block but she doesn't include this transaction in it, Bob is defrauded. As we can see in the graph blow, Bob can wait some time that several blocks are extended from the correct block containing the transaction from Alice to him and then confirm this transaction. In this case, malicious nodes is hardly to catch up the upper branch, because honest nodes will always extend the longest branch so that malicious nodes should be "lucky" enough to be selected with high possibility to propose their blocks to the lower branch.
	![]({{site.url}}/assets/image/bitcoin_trans_confirmation.jpeg){:width="400px"}_6 confirmations to avoid double spending_
	
Recap:
- Protection against invalid transactions is cryptographic, but enforced by consensus, i.e, assuming majority of nodes are honest and they will ignore the invalid transactions.
- Protection against double-spending is purely bu consensus.
- One is never 100% sure a transaction is in consensus chain but is guaranteed with high probability. 
	
### 2.4. Incentives and proof of work

We assumed the honesty of the majority of nodes in the previous section but obviously the assumption is not solid and can be subverted by some financial methods. **Bitcoin gives nodes incentives for behaving honestly.** We are not able to penalise malicious nodes because they have no identities. Instead, we reward honest nodes who propose blocks ending up in the long-term consensus chain.
- incentive 1: block reward. Creator of block gets to include special coin-creation transaction in the block and choose recipient address of this transaction. The reward value halves every 4 years from 25BTC. Block creator gets to "collect" the reward only if the block ends up on the long-term consensus chain.
- incentive 2: transaction fees. Creator of transaction can choose to make output value less than input value. The remainder is a transaction fee and goes to block creator.

Remember that we have a remaining problem: picking a node randomly for proposing the next block. It is made by `Proof-of-work`: select nodes in proportion to a resource that no one chan monopolize.
- in proportion to computing power: proof-of-work
- in proportion to ownership: proof-of-stake
	
The computing competition in proof-of-work in bitcoin is Hash puzzles.
![]({{site.url}}/assets/image/hash_puzzles.png){:width="400px"}_hash puzzles_
The lucky node who solve the hash puzzle problem gets to propose the next block and the chance to solve it is proportional to is computing power.

Here are some property of PoW:
1. difficult to compute. The nodes who compete for solving the hash puzzle problem is called miners.
2. parameterizable cost. Nodes automatically re-calculate the target every 2 weeks to reach a average time between blocks as 10 minutes.
3. trivial to verify, Nonce must be published as part of block. Other miners simply verify that $$H(nonce \mid prev\_hash \mid tx \mid ... \mid tx) < target$$. Here $$\mid$$ is still the string concatenation symbol.

Given that Prob(Alice wins next block) = fraction of global hash power she controls. The key security assumption comes:
> Attacks infeasible if majority of miners <ins>weighted by hash power</ins> follow the protocol.






