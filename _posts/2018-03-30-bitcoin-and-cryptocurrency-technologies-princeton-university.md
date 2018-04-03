---
layout: post
title: "Cryptocurrency course of Princeton university: Lecture 1 - Intro to crypto and cryptocurrencies"
date: 2018-03-30
categories: [courses, blockchain]
---

## 1. Crypto background

### 1.1. cryptographic hash functions
takes any string as input, fixed-size output, efficiently computable.

Security properties:
- **collision-free.** No body can find $$x$$ and $$y$$ such that $$x \neq y$$ and $$H(x) = H(y)$$. While collisions do exist but it's not findable by regular people with regular machines. No hash function is proved to be collision-free but people choose to believe some are collision-free because they tried hard to find collisions but didn't succeeded. Application: Hash as message digest. If we know $$H(x) = H(y)$$ it's safe to assume that $$x = y$$.

  ![]({{site.url}}/assets/image/Collisions_do_exist.png){:width="300px"}_collisions do exist_

- **hiding.** We want: Given $$H(x)$$, it is infeasible to find $$x$$. Hiding property: if $$r$$ is chosen from a probability distribution that has *high min-entropy*, then given $$H(r \mid x)$$ (r concatenated with x), it is infeasible to find x. *High min-entropy* means no particular value is chosen with more than negligible probability. Application: seal a message an envelope: `commit(msg) = (com, key) = (H(key|msg), H(key)) ` where key is a random 256-bit value, then publish key and msg, only people who hold the correct com can open the massage.

- **puzzle friendly.** For every possible output value $$y$$, if $$k$$ chosen from a distribution with high min-entropy, then it is infeasible to find $$x$$ such that $$H(k \mid x) = y$$. (Difference with hiding property: to find one $$x$$ doesn't mean finding the input $$x$$, there many possibles $$x$$s but it's infeasible to find one.). Application: Search puzzle. Given a "puzzle ID" _id_ (from high min-entropy distribution) and a target set $$Y$$, try to find a "solution" $$x$$ such that $$H(id \mid x) \in Y$$. Puzzle friendly property implies that no solving strategy is much better than trying random values of x.

Bitcoin use SHA-256 hash function:

![]({{site.url}}/assets/image/SHA-256.png){:width="300px"}_SHA-256 hash function_

> Theorem: if $$c$$ is collision-free, the SHA-256 is collision-free.

### 1.2. Hash Pointers and Data Structures
A hash pointer tells where some info is stored and what is the (cryptographic) hash of the info. We can ask to get the info back and verify that it hasn't changed with the hash pointer.

key idea: build data structures with hash pointers.

![]({{site.url}}/assets/image/hash_pointer.png){:width="300px"}_hash pointer for detecting tempering_

If any one wants to tamper one block, he has to temper all the precedent hash pointers all the way to the beginning, because every block is checked by it's previous block's hash pointer. Suppose we remember the head hash pointer so that we can easily detect the tempering.

![]({{site.url}}/assets/image/merkle_tree.png){:width="300px"}_hash pointer for merkle tree_

Another example is the binary tree (`merkle tree`) with hash pointer

![]({{site.url}}/assets/image/merkle_tree_search.png){:width="300px"}_merkle tree search_

For proving a data block is in a `merkle tree`, we only need show $$O(log(n))$$ hash pointers.

More generally, we can use hash pointers in any pointer-based data structure that has no cycles.

### 1.3. Digital signatures

Only you can sign, but anyone can verify. Signature is tied to a particular document and it can't be cut-and-pasted to another doc.

API for digital signatures:

```
(sk, pk) = generateKeys(keysize)
sk: secret signing key
pk: public verification key

sig = sign(sk, message)

isValid = verify(pk, message, sig)
```


## 2. Intro to cryptocurrencies
