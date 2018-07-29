---
layout: post
title: "Sequence Models by Andrew Ng: Week 1 - Recurrent Neural Networks"
date: 2018-04-24
categories: [courses, deep learning, sequence models]
---

Here I only note some of the points in the course that are new to me.

### Forward propagation and backpropagation

For a simple RNN structure:
- forward:
	- inputs: sequential values $$x_1, x_2, ... x_n$$ and the initial state $$s_0$$
	- internal states: $$s_{t+1} = g(W_s\cdot[x_t, s_t] + b_s)$$ where $$g$$ is an activation function e.g. `tanh` or `ReLU`
	- outputs: $$\hat{y}_t = f(W_y\cdot s_t + b_y) $$ where $$f$$ is another activation function, which could be `softmax` or `sigmoid`
- backward
	- labels: sequential values $$y_1, y_2, ... y_n$$
	- loss fucntion: cross-entropy ...

### Different types of RNNs

- many to many: language model
- many ro one: move rating
- on to many: music generation
- many to many with different input/output length: machine translation

  ![]({{site.url}}/assets/image/machine_translation.png){:width="300px"}_machine translation: many to many model_
  
### Language model and sequence generation

### Vanishing gradients with RNNs
RNN with large number of layers can hardly capture the long-term dependency due to vanishing gradients. -- gradients could decrease exponetially during the backpropagation. 

- exploding gradients can be solved robustly by gradient clipping, i.e. if gradient is larger than a threshold, rescale it.
- vanishing gradients have not simple solution, but architectures like GRU/LSTM are used to mitigate it.

### Gated Recurrent Unit.

Simplified version:

$$\tilde{c}^{<t>} = tanh(W_c[c^{<t-1>}, x^{<t>}] + b_c)$$

$$\Gamma_u = \sigma(W_u[c^{<t-1>}, x^{<t>}] + b_u)$$

$$c^{<t>} = \Gamma_u * \tilde{c}^{<t>} + (1-\Gamma_u)*c^{<t-1>}$$

$$\Gamma$$ is the gate to determine how much and which dimensions of the current candidate vector is updated to the memory cell $$c^{<t>}$$. Ideally, if the long-term dependent info is stored in the cell and irrelative inputs are ignored, this dependency may be captured even if it happens before many inputs.

The Full GRU:

$$\tilde{c}^{<t>} = tanh(W_c[\Gamma_r * c^{<t-1>}, x^{<t>}] + b_c)$$

$$\Gamma_u = \sigma(W_u[c^{<t-1>}, x^{<t>}] + b_u)$$

$$\Gamma_r = \sigma(W_r[c^{<t-1>}, x^{<t>}] + b_r)$$

$$c^{<t>} = \Gamma_u * \tilde{c}^{<t>} + (1-\Gamma_u)*c^{<t-1>}$$

$$a^{<t>} = c^{<t>}$$

where $$a^{<t>}$$ is the output value.

The Full version is the result of many researches and experiments, adding a relavance gate $$\Gamma_r$$ to tell whether thepreviouscell is relavant to calculate the candidate cell.

### LSTM

Indeed LSTM is a more general version of GRU, adding a forget gate $$\Gamma_f$$ to replace $$(1-\Gamma_u)$$ and another output gate $$\Gamma_o$$ to filter cell value to output value.

$$\tilde{c}^{<t>} = tanh(W_c[c^{<t-1>}, x^{<t>}] + b_c)$$

$$\Gamma_u = \sigma(W_u[a^{<t-1>}, x^{<t>}] + b_u)$$

$$\Gamma_f = \sigma(W_f[a^{<t-1>}, x^{<t>}] + b_f)$$

$$\Gamma_o = \sigma(W_o[a^{<t-1>}, x^{<t>}] + b_o)$$

$$c^{<t>} = \Gamma_u * \tilde{c}^{<t>} + \Gamma_f*c^{<t-1>}$$

$$a^{<t>} = \Gamma_o * c^{<t>}$$

### Bidirectional RNN

At each timestamp, calculate the output value from 2 directions. $$y^{<t>}$$ depends on both $$\overrightarrow{a}^{<t>}$$ and $$\overleftarrow{a}^{<t>}$$ :

$$$y^{<t>} = g(W_y[\overrightarrow{a} ^{<t>}, \overleftarrow{a} ^{<t>}] + b_y)$$

### Deep RNN
