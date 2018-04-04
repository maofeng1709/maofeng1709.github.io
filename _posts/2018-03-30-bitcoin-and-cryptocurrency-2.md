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

### 2.3. Consensus without identity: the blockchain

Why don't Bitcoin nodes have identities:
- Identity is hard in a P2P system - Sybil attack
- Pseudonymity is a goal of Bitcoin

	




