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

- **puzzle friendly. ** For every possible output value $$y$$, if $$k$$ chosen from a distribution with high min-entropy, then it is infeasible to find $$x$$ such that $$H(k \mid x) = y$$. (Difference with hiding property: to find one $$x$$ doesn't mean finding the input $$x$$, there many possibles $$x$$s but it's infeasible to find one.).
## 2. Intro to cryptocurrencies
