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
\label{eq1}
\end{equation}
$$

With $$J$$ the final loss function, we need calculate the derivative of $$J$$ with respect to each parameters $$(W_c, W_x, b)$$ and $$c_{t-1}$$ to back-propagate the gradient flow and update those parameters. Let's take $$W_x$$ as an example, it's derivative can be calculated by the chain rule:

$$
\begin{equation}
\frac{\partial J}{\partial W_x} = \frac{\partial J}{\partial c_t}\frac{\partial c_t}{\partial W_x}
\label{eq2}
\end{equation}
$$

Moreover, if we note $$u_t = W_cc_{t-1} + W_xx_t + b$$, the chain rule can be further written as 

$$
\begin{equation}
\frac{\partial J}{\partial W_x} = \frac{\partial J}{\partial c_t}\frac{\partial c_t}{\partial u_t}\frac{\partial u_t}{\partial W_x}\label{eq3}
\end{equation}
$$

For each cell at timestamp $$t-1$$ , it receives $$\frac{\partial J}{\partial c_t}$$ and use its own parameters to calculate $$\frac{\partial c_t}{\partial W_x}$$. Some materials expand the formula $$\eqref{eq3}$$ without any extra specification as the following:

$$
\begin{equation}
\frac{\partial c_t}{\partial u_t} = \frac{\partial tanh(u_t)}{\partial u_t} = 1 - tanh^2(u_t) = 1 - c_t^2 \label{eq4}
\end{equation}
$$

$$
\begin{equation}
\frac{\partial u_t}{\partial W_x} = x_t^T \label{eq5}
\end{equation}
$$

$$
\begin{equation}
\frac{\partial J}{\partial W_x} = \frac{\partial J}{\partial c_t}(1 - c_t^2)x_t^T \label{eq6}
\end{equation}
$$

**Which is obviously not correct in matrix case !**

if we apply a mini-batch gradient descent for the backpropagation, let's say batch size is $$m$$, all the relative dimensions for the chain rule calculation are: $$J() \in \mathbb{R}, c_t \in \mathbb{R}^{n_c \times m}, W_x \in \mathbb{R}^{n_c \times n_x}, x_t \in \mathbb{R}^{n_x \times m}$$.

We can see that the expected dimension of $$\frac{\partial J}{\partial W_x}$$ is the same shape of $$W_x$$, i.e. $$\mathbb{R}^{n_c \times n_x}$$. However, we can not produce this shape simply by tensor multiplication operations on the right side of equation $$\eqref{eq6}$$. Furthermore, the equations $$\eqref{eq4}$$ and $$\eqref{eq5}$$ actually hold with the matrix variables.

**So how can we derive a closed-form equation for the chain rule where matrix-valued functions are involved, so that the backpropogation can be coded efficiently in a programming language?**

### Vector in, Vector out: Jacobian

Now suppose that $$c_t$$ is a vector in $$\mathbb{R}^{n_c}$$, so the input $$u_t$$ of $$tanh$$ in $$\eqref{eq1}$$ is a vector of same dimension. Then the derivative of $$c_t$$ with respect to $$u_t$$, also called the Jacobian:

$$
\frac{\partial c_t}{\partial u_t} = 
\begin{pmatrix}
\frac{\partial c_t^{(1)}}{\partial u_t^{(1)}} & \cdots & \frac{\partial c_t^{(1)}}{\partial u_t^{(n_c)}} \\ 
\vdots & \ddots & \vdots \\ 
\frac{\partial c_t^{(n_c)}}{\partial u_t^{(1)}} & \cdots & \frac{\partial c_t^{(n_c)}}{\partial u_t^{(n_c)}} 
\end{pmatrix}
$$

Since $$tanh$$ is a element-wise activation function, only the $$i$$th element in the input as an impact on the output, so:

$$
\begin{equation}
\frac{\partial c_t}{\partial u_t} = 
\begin{pmatrix}
\frac{\partial c_t^{(1)}}{\partial u_t^{(1)}} & \cdots & 0 \\ 
\vdots & \ddots & \vdots \\ 
0 & \cdots & \frac{\partial c_t^{(n_c)}}{\partial u_t^{(n_c)}} 
\end{pmatrix} = 
\begin{pmatrix}
1 - tanh^2(u_t^{(1)}) & \cdots & 0 \\ 
\vdots & \ddots & \vdots \\ 
0 & \cdots & 1 - tanh^2(u_t^{(n_c)})
\end{pmatrix}
\label{eq7}
\end{equation}
$$

But what is $$\frac{\partial u_t}{\partial W_x}$$? What is the shape of this derivative of a vector with respect to a matrix? Only with the Jacobian, it's not enough to unclose our chain rule.

### Tensor in, Tensor out: Generalized Jacobian

Recall that $$J() \in \mathbb{R}, c_t \in \mathbb{R}^{n_c \times m}, W_x \in \mathbb{R}^{n_c \times n_x}, x_t \in \mathbb{R}^{n_x \times m}$$.  We can see that $$\frac{\partial c_t}{\partial u_t}$$ and $$\frac{\partial c_t}{\partial u_t}$$ are 2 derivatives of matrix with respect to matrix, which is called *generalized Jacobian*, e.g. $$\frac{\partial c_t}{\partial u_t}$$ is an object with shape:

$$(n_c \times m) \times (n_c \times m)$$

if we let $$i \in \mathbb{Z}^{n_c \times m}$$ and $$j \in \mathbb{Z}^{n_c \times m}$$ be vectors of indices, then we have:

$$
\left(\frac{\partial c_t}{\partial u_t}\right)^{(i, j)} = \frac{\partial c_t^{(i)}}{\partial u_t^{(j)}}
$$

Similarly, we have $$\frac{\partial u_t}{\partial W_x}$$ produce a generalized Jacobian of shape: $$(n_c \times m) \times (n_x \times m)$$. Therefore, the chain rule of equation $$\eqref{eq3}$$ is a sequence of generalized matrix product, of shape $$(n_c \times m), (n_c \times m) \times (n_c \times m), (n_c \times m) \times (n_x \times m)$$, which results a matrix of shape $$n_x \times m$$ as expected. In fact, with the notion of generalized Jacobian, the shape of each element in the chain rule naturally become compatible.

However, calculating the chain rule this way is extremely inefficient as the generalized Jacobian is too space-consuming, e.g. let $$n_c = 1024$$ and $$m = 64$$, then $$\frac{\partial c_t}{\partial u_t}$$ consists of $$1024 \times 64 \times 1024 \times 64$$ scalar values, that will 16GB of memory to store, using 32-bit float point.

In the next section, we are going to derive expressions that compute the chain rule without explicitly forming the generalized Jacobian.  

### Backpropagation with Tensors

We can simplify the equation $$\eqref{eq3}$$ as:

$$
\begin{eqnarray*}
\frac{\partial J}{\partial W_x} 
&=& \frac{\partial J}{\partial c_t}\frac{\partial c_t}{\partial u_t}\frac{\partial u_t}{\partial W_x} \\
&=& \sum_{i, j} \frac{\partial J}{\partial c_t^{(i, j)}}\frac{\partial c_t^{(i, j)}}{\partial u_t}\frac{\partial u_t}{\partial W_x} \\
&=& \sum_{i, j} \frac{\partial J}{\partial c_t^{(i, j)}}\left(\sum_{p, q}\frac{\partial c_t^{(i, j)}}{\partial u_t^{(p,q)}}\frac{\partial u_t^{(p,q)}}{\partial W_x}\right) 
\end{eqnarray*}
$$

Note that $$\frac{\partial c_t^{(i, j)}}{\partial u_t^{(p,q)}} = 0$$ when $$(i, j) \neq (p,q)$$ since $$tanh$$ is the element-wise activation function, the above equation can be further expaned as:

$$
\begin{eqnarray*}
\frac{\partial J}{\partial W_x} 
&=& \sum_{i, j} \frac{\partial J}{\partial c_t^{(i, j)}}\left(\sum_{p, q}\frac{\partial c_t^{(i, j)}}{\partial u_t^{(p,q)}}\frac{\partial u_t^{(p,q)}}{\partial W_x}\right) \\
&=& \sum_{i, j} \frac{\partial J}{\partial c_t^{(i, j)}}\frac{\partial c_t^{(i, j)}}{\partial u_t^{(i,j)}}\frac{\partial u_t^{(i,j)}}{\partial W_x}
\end{eqnarray*}
$$

Let's then look at one element $$\left(\frac{\partial J}{\partial W_x}\right)^{(k,l)}$$ in the final matrix:

$$
\begin{eqnarray*}
\left(\frac{\partial J}{\partial W_x}\right)^{(k,l)} 
&=& \frac{\partial J}{\partial W_x^{(k,l)}} \\
&=& \sum_{i, j} \frac{\partial J}{\partial c_t^{(i, j)}}\frac{\partial c_t^{(i, j)}}{\partial u_t^{(i,j)}}\frac{\partial u_t^{(i,j)}}{\partial W_x^{(k,l)}} \\
&=& \sum_{i, j} \frac{\partial J}{\partial c_t^{(i, j)}}\frac{\partial c_t^{(i, j)}}{\partial u_t^{(i,j)}}\frac{\partial \sum_s W_x^{(k, s)}x_t^{(s, l)}}{\partial W_x^{(k,l)}} \\
&=& \sum_{i, j} \frac{\partial J}{\partial c_t^{(i, j)}}\frac{\partial c_t^{(i, j)}}{\partial u_t^{(i,j)}}\frac{\partial W_x^{(i, l)}x_t^{(l, j)}}{\partial W_x^{(k,l)}} 
\end{eqnarray*}
$$

We see that $$\frac{\partial W_x^{(i, l)}x_t^{(l, j)}}{\partial W_x^{(k,l)}} = 0$$ if $$i \neq k$$: 

$$
\begin{eqnarray*}
\left(\frac{\partial J}{\partial W_x}\right)^{(k,l)} 
&=& \sum_{j} \frac{\partial J}{\partial c_t^{(k, j)}}\frac{\partial c_t^{(k, j)}}{\partial u_t^{(k,j)}}\frac{\partial W_x^{(k, l)}x_t^{(l, j)}}{\partial W_x^{(k,l)}}\\
&=& \sum_{j} \frac{\partial J}{\partial c_t^{(k, j)}}\frac{\partial c_t^{(k, j)}}{\partial u_t^{(k,j)}}x_t^{(l, j)}
\end{eqnarray*}
$$

We note $$dtanh$$ the matrix with each entry $$dtanh^{(i,j)} = \frac{\partial c_t^{(i,j)}}{\partial u_t^{(i,j)}} = 1 - (c_t^{(i,j)})^2$$, so the above result is actually the vector product of $$i$$th row of $$\frac{\partial J}{\partial c_t} \odot dtanh$$ and $$j$$th column of $$x_t^T$$ ,so we finally have:

$$
\begin{equation}
\frac{\partial J}{\partial W_x} = \left(\frac{\partial J}{\partial c_t} \odot dtanh\right)x_t^T
\end{equation}
\label{eq8}
$$

where $$\odot$$ is the element-wise matrix product operation. With this result, the backpropagation can be coded by simply because at a timestamp, a cell only need to calculate the element-wise product of received $$\frac{\partial J}{\partial W_x}$$ and $$dtanh$$ and then perform a matrix multitplication with $$x_t^T$$.

As a conclusion, derivatives with respect to other parameters can also be simplified this way. Furthermore, all types of matrix-valued mapping like $$x \overset{\text{linear}}{\longmapsto} u \overset{\text{activation}}{\longmapsto} c \longmapsto y$$ can be formulated as $$\eqref{eq8}$$, where we only need to adjust the position of $$x^T$$ to make the matrix product shape compatible.



### references
- [Derivatives, Backpropagation, and Vectorization](http://cs231n.stanford.edu/handouts/derivatives.pdf)
- [机器学习中的矩阵、向量求导](https://www.zhihu.com/question/52399883)

