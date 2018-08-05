---
layout: post
title: "Recurrent Highway Networks -- [JG Zilly et al. (2016)]"
date: 2018-05-15
categories: [readings, papers]
---

Paper reference: [arXiv:1607.03474](https://arxiv.org/abs/1607.03474)

### 1. Introduction

Network depth is a key factor for neural network power. Deeper networks can be exponentially more efficient at representing some function classes. As a sequential model, RNN  has long credit assignment paths. 

However, simply increasing the depth of a networks gives rise to the vanishing and exploding gradient problems. Since the gradient may shrink or explode during the process of backpropagation. The widely used architecture LSTM is specifically for alleviate this problem.

Highway layer based on the LSTM cell addressed the limitation of training very deep feedforward network, even with hundreds of stacked layers. These layers have been applied in many domains like speech recognition and language modeling.

In this paper, 2 main contributions are proposed:
- a new mathematical analysis of RNNs gradient flow
- Recurrent Highway Networks which have long credit assignment paths in time and in space (per time step).

### 2. Deep recurrent transisiotns
Compared to stacking recurrent layers, increasing the recurrence
depth can add significantly higher modeling power
to an RNN.
  ![]({{site.url}}/assets/image/recurrent_depth.png){:width="300px"}_recurrent_depth v.s. stacked depth_

### 3. Gradient Flow in Recurrent Networks

Consider a standard RNN $$y^{[t]} = f(Wx^{[t]} + Ry^{[t-1]} + b$$. The derivative of the loss $$\mathcal{L}$$ with respect to parameters $$\theta$$ can be written using the chain rule:

$$\frac{d\mathcal{L}}{d\theta} = \sum_{1\leq t_2\leq T}\frac{d\mathcal{L}^{[t_2]}}{d\theta} = \sum_{1\leq t_2\leq T }\sum_{1\leq t_1\leq t_2}\frac{\partial \mathcal{L}^{[t_2]}}{\partial y^{[t_2]}}\frac{\partial y^{[t_2]}}{\partial y^{[t_1]}}\frac{\partial y^{[t_1]}}{\partial \theta}$$

The key factor of the transport, the Jacobian matrix is obtained by chaining the derivatives across all time steps:

$$\frac{\partial y^{[t_2]}}{\partial y^{[t_1]}}=\prod_{t_1 < t \leq t_2} \frac{\partial y^{[t]}}{\partial y^{[t-1]}} = \prod_{t_1 < t \leq t_2} R^T diag[f'(Ry^{[t-1]})]$$

If we denote $$A$$ as de temporal Jacobian, $$\gamma$$ as a maximal bound on $$f'(Ry^{[t-1]})$$ and $$\sigma_{max}$$ as the largest singular value of $$R^T$$. Then the norm of the Jacobian satisfies:

$$\| A \| \leq \|R^T\|  \|diag[f'(Ry^{[t-1]})]\| \leq \gamma \sigma_{max}$$

So it's clear that the conditions for vanishing gradients is $$\gamma\sigma_{max} < 1$$. We are going to see some insights about the gradient vanishing/exploding with the eigenvalues:

**Geršgorin circle theorem (GCT)** (Geršgorin, 1931): For any square matrix $$A \in R^{n\times n}$$

$$spec(A) \subset \bigcup_{i \in \{1,...,n\}}\Big\{\lambda \in \mathbb{C} \mid \|\lambda - a_{ii}\|_\mathbb{C} \leq \sum_{j=i,  j \neq i}^{n} |a_{ij}| \Big\}$$

i.e. the eignevalues of matrix $$A$$, are located within the union of the complex circles centered around the diagonal values $$a_{ii}$$ with radius equal to the sum of the absolute values of the non-diagonal entries in each row of $$A$$.

As illustrated as GTC, some models proposed to initialize $$R$$ with an identity matrix and small values on the off-diagonals.

  ![]({{site.url}}/assets/image/GCT.png){:width="300px"}



### 4. Recurrent Highway Networks (RHN)

In this section, the author porpose Recurrent Highway Networks inspired by the RNN with Highway layers.

Recall:
- Highway layer: $$y = h\cdot t + x\cdot c$$, where "." denotes element-wise multiplication, $$h = H(x, W_H)$$, $$t = T(x, W_T)$$, $$c = C(x, W_C)$$.
- A standard RNN: $$y^{[t]} = f(Wx^{[t]} + Ry^{[t-1]}) + b$$

Then an RHN layer with a recurrence depth of $$L$$ is described by:

$$s_l^{[t]} = h_l^{[t]} \cdot t_l^{[t]} + s_{l-1}^{[t]} \cdot c_l^{[t]}$$

where:

$$h_l^{[t]} = tanh(W_Hx^{[t]}\mathbb{1}_{\{l=1\}} + R_{H_l}s_{l-1}^{[t]} + b_{H_l})$$

$$t_l^{[t]} = \sigma(W_Tx^{[t]}\mathbb{1}_{\{l=1\}} + R_{T_l}s_{l-1}^{[t]} + b_{T_l})$$

$$c_l^{[t]} = \sigma(W_Cx^{[t]}\mathbb{1}_{\{l=1\}} + R_{C_l}s_{l-1}^{[t]} + b_{C_l})$$

  ![]({{site.url}}/assets/image/RHN.png){:width="600px"}
  
`Compared to the situation for standard RNN, it can be seen that an RHN layer has more flexibility in adjusting the centers and radii of the Geršgorin circles`