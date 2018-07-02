---
layout: post
title: "WORD TRANSLATION WITHOUT PARALLEL DATA -- [Conneau et al. (2017)]"
date: 2018-05-05
categories: [readings, papers]
---

Paper reference: [arXiv:1710.04087](https://arxiv.org/abs/1710.04087)

### 1. Introduction

Most successful word embedding methods rely on the distributional hypothesis of Harris (1954), which states that words occurring in similar contexts tend to have similar
meanings:
- [Tomas Mikolov, Kai Chen, Greg Corrado, and Jeffrey Dean. Efficient estimation of word representations in vector space. Proceedings of Workshop at ICLR, 2013a.](https://arxiv.org/abs/1301.3781)
- [Jeffrey Pennington, Richard Socher, and Christopher D Manning. Glove: Global vectors for word representation. Proceedings of EMNLP, 14:1532–1543, 2014.](http://www.aclweb.org/anthology/D14-1162)
- [Piotr Bojanowski, Edouard Grave, Armand Joulin, and Tomas Mikolov. Enriching word vectors with subword information. Transactions of the Association for Computational Linguistics, 5: 135–146, 2017.](https://arxiv.org/abs/1607.04606)<br>

[Mikolov et al. (2013)](https://arxiv.org/abs/1309.4168) first noticed that continuous word embedding spaces exhibit similar structures
across languages even for distant languages. Since then many studies are conducted to exploit this similarity by learning a mapping from a source language embedding space to a target space, relying on bilingual word lexicons. 

### 2. Model
Suppose that we have $$n$$ pair of words $$\{x_i, y_i\}_{i\in\{1,n\}}$$ and want to learn a mapping from the source to the target space such that

$$W^\star = \underset{W \in M_d(\mathbb{R})}{\operatorname{argmin}} ||WX-Y||_F$$

In practice, the result improves if we enforce an orthogonal constraint on $$W$$.

$$W^\star = \underset{W \in O_d(\mathbb{R})}{\operatorname{argmin}} ||WX-Y||_F = UV^T, \text{ with } U\Sigma V^T = SVD(YX^T)$$

This equation is so-called the Procrustes problem, which advantageously offers a closed form solution obtained from the singular value decomposition(SVD).

#### 2.1 Adversarial learning setting

The main idea of the unsupervised learning procedure is done by a adversarial learning algorithm, as introduced as [Goodfellow et al. (2014)](https://papers.nips.cc/paper/5423-generative-adversarial-nets)

Suppose that we have 2 set of word embeddings $$\mathcal{X} = \{x_1, ..., x_n\}$$ and $$\mathcal{Y} = \{y_1, ..., y_m\}$$ from a source and a target language respectively. We are going to train 2 models:
- **A Discriminator** which discriminate between elements randomly sampled from $$ W\mathcal{X} = \{Wx_1, ..., Wx_n\}$$ and $$\mathcal{Y}$$. The loss can be written as:

$$\mathcal{L}_D(\theta_D|W) = -\frac{1}{n}\sum_{i=1}^n \log P_{\theta_D}(\text{source}=1|Wx_i)-\frac{1}{m}\sum_{i=1}^m \log P_{\theta_D}(\text{source}=0|y_i)$$

- **A mapping** who prevent the discriminator from making accurate predictions.

$$\sum_{1}^{n}$$

#### 2.2 Refinement

#### 2.3 CROSS-DOMAIN SIMILARITY LOCAL SCALING (CSLS)




