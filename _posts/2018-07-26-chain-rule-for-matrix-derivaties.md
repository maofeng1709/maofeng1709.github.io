---
layout: post
title: "Chain rule for derivatives of matrix-valued functions in deep learning"
author: "Mao FENG"
categories: others
---

I have encountered several times the problem of shape matching while constructing a deep neural network, especially for the derivatives of matrix-valued function during backpropagation.

Let's consider the a standard basic RNN:

$$
\begin{equation}
c_t = tanh(W_cc_{t-1} + W_xx_t + b)
\label{eq0}
\end{equation}
$$

With $$J$$ the final loss function, we need calculate the derivative of $$J$$ with respect to each parameters $$(W_c, W_x, b)$$ and $$c_{t-1}$$ to back-propagate the gradient flow and update those parameters. Let's take $$W_x$$ as an example, it's derivative can be calculated by the chain rule:

$$
\begin{equation}
\frac{\partial J}{\partial W_x} = \frac{\partial J}{\partial c_t}\frac{\partial c_t}{\partial W_x}
\label{eq1}
\end{equation}
$$

For each cell at timestamp $$t-1$$ , it receives $$\frac{\partial J}{\partial c_t}$$ and use its own parameters to calculate $$\frac{\partial c_t}{\partial W_x}$$. Some materials expand formula $$\eqref{eq1}$$ without any extra specification as the following:

$$

\frac{\partial tanh(x)}{\partial x} = 1 - tanh^2(x) \\
\frac{\partial c_t}{\partial W_x} = (1 - tanh^2(W_cc_{t-1} + W_xx_t + b)) = (1 - c_t^2)x_t^T
$$

$$
\begin{equation}
\frac{\partial J}{\partial W_x} = \frac{\partial J}{\partial c_t}(1 - c_t^2)x_t^T
\label{eq2}
\end{equation}
$$

if we apply a mini-batch gradient descent for the backpropagation, let's say batch size is $$m$$, all the relative dimensions for the chain rule calculation are: $$J() \in \mathbb{R}, c_t \in \mathbb{R}^{n_c \times m}, W_x \in \mathbb{R}^{n_c \times n_x}, x_t \in \mathbb{R}^{n_x \times m}$$.

**We can see that the expected dimension of $$\frac{\partial J}{\partial W_x}$$ is the same shape of $$W_x$$, i.e. $$\mathbb{R}^{n_c \times n_x}$$. However, we can not produce this shape simply by tensor multiplication operations on the right side of equation $$\eqref{eq2}$$, because the shape of $$(1-c_t^2)$$ is not well defined . It is the derivative of the activation function $$tanh$$ which takes a tensor of shape $$[n_c, m]$$ as input and outputs a tensor of the same shape.**

**What is exactly the derivative of $$tanh$$, i.e. $$(1-c_t^2)$$ ? Let's start from the vector case:**

### Vector in, Vector out: Jacobian

Now suppose that $$c_t$$ is a vector in $$\mathbb{R}^{n_c}$$, so the input $$u$$ of $$tanh$$ in $$\eqref{eq0}$$ is a vector of same dimension. Then the derivative of $$c_t$$ with respect to $$u$$, also called the Jacobian:

$$
\frac{\partial c_t}{\partial u} = 
\begin{pmatrix}
\frac{\partial c_t^{(1)}}{\partial u^{(1)}} & \cdots & \frac{\partial c_t^{(1)}}{\partial u^{(n_c)}} \\ 
\vdots & \ddots & \vdots \\ 
\frac{\partial c_t^{(n_c)}}{\partial u_{(1)}} & \cdots & \frac{\partial c_t^{(n_c)}}{\partial u_{(n_c)}} 
\end{pmatrix}
$$


### Tensor in, Tensor out: Generalized Jacobian

### Backpropagation with Tensors


### references
- [Derivatives, Backpropagation, and Vectorization](http://cs231n.stanford.edu/handouts/derivatives.pdf)
- [机器学习中的矩阵、向量求导](https://www.zhihu.com/question/52399883)

