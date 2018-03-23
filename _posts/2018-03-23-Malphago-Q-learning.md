---
layout: post
title: "Malphago, a Q-learning agent for rock-paper-scissors"
date: 2018-03-23
categories: [projects]
---

## 1. Overview
	
[Malphago](malphago-195815.appspot.com) is a Q-learning agent for the simple game rock-paper-scissors served by Flask web application. The purpose of this project is to test Q-learning's performance in an interactive environment. In the meantime, [Deep Malphago](https://malphago-195815.appspot.com/DeepMalphago) is developed with deep Q-learning algorithm with tensorflow nn models, which is inspired by Deepmind's paper [Playing atari with deep reinforcement learning](https://arxiv.org/pdf/1312.5602.pdf).
	
<iframe style="width: 100%; height: 705px; border: none;" src="https://malphago-195815.appspot.com/" width="100%" height="705"></iframe> 
	
## 2. Hypothesis

- Each human player has his own strategy/habit of playing rock-paper-scissors.
	
- This strategy/habit depends on his last observation of game result. (rock v.s paper, paper v.s scissors or etc.)
	
- This strategy/habit may change during the playing process.
	
- Different human players have different strategies/habits.
	
Based on the hypothesis above, we apply Q-learning algorithm to capture the human player's strategy and we assign one Q-function to each game/player with the help of [Flask-Session](https://pythonhosted.org/Flask-Session/). E.g., once a human player Bob connects to our application, flask server allocates a session for Bob which records Bob's playing status, history as well as the updating Q-function. When Bob disconnects from our app(e.g. closing the web browser) or click to restart an episode, the session for him is destroyed because one Q-function is applicable only for one specific player for one episode of game. A new Q-function should be initialised for a new connection of human player.

## 3. Q-function setup

As introduced in the previous section, human player's next choice depends on his current observation of game result. We call this observation as "state" and we call the choice in (rock, paper, scissors) as "action" in the rest of this article. Q-learning algorithm evaluates the long-term reward of each action given the current state. The Q-function applied by Malphago is actually a table as below:
	
|-----------------------------|------|-------|----------|
| state(previous observation) | rock | paper | scissors |
|-----------------------------|------|-------|----------|
| init state                  |      |       |          |
| (rock, rock)                |      |       |          |
| (rock, paper)               |      |       |          |
| ...                         |      |       |          |
{:.post-table}
	
We have 3x3=9 normal states and a initial state which represents the game just begins. Each empty cell in the table should contain a value which indicates the expected long-term reward that Malphago take the column action $$a$$ given the row state $$s$$, defined as $$Q(s,a)$$.
	
## 4. Q-learning algorithm for Malphago

Our goal is to find a good Q-function iteratively while playing with human player. The updating equation at line 6 in the Algorithm below can be proved to converge to the optimal Q-function.

![]({{site.url}}/assets/image/Q_learning_algo.png){:width="500px"}
	
The get_init_Q() is used to initialise our Q-function based on the total game history with all human players. The idea is that initially, $$Q(s_0, a_0)$$ equals to: <br>
$$\mathbb{E}[r\mid s_0,a_0] = \mathbb{P}(a_0 \text{ wins human's action } \mid s_0, a_0) - \mathbb{P}(\text{human's action wins } a_0 \mid s_0, a_0)$$
 <br>For example,<br>
 $$Q(s_0, rock) = \mathbb{P}(\text{human's action = scissors} \mid s_0,rock) - \mathbb{P}(\text{ wins human's action = papers} \mid s_0, rock)$$
Here we implicitly apply a reward function $$r(win, tie, loss) = (1, 0 , -1).$$
	
Each time we receive a human player's action by a request from client's side, we observe the result of the game and get the current reward, i.e. the line 5 in the above algorithm. Then we update the Q-function according to the updating equation as the code below shows at line 9, 10:

{%highlight python linenos%}
@app.route('/update_state', methods=['GET', 'POST'])
def update_state():
    Q, my_choice, curr_state = get_context()
    # update the Q function
    choice = int(request.form['choice'])
    write_db(request.cookies['session'], choice, my_choice, curr_state)
    r = get_reward(choice)
    next_state = choice + my_choice*3
    update_value = comm_vars.alpha * (r + comm_vars.gamma * max(Q[next_state]) - Q[curr_state][my_choice])
    session['Q_function'][curr_state][my_choice] += update_value
    # return new choice, eplison greedy
    if np.random.rand() < comm_vars.epsilon:
        session['my_choice'] = np.random.randint(3)
    else:
        session['my_choice'] = np.argmax(session['Q_function'][next_state])
    session['curr_state'] = next_state
    print get_context()
    return str(session['my_choice'])
{% endhighlight %}

Notice that every time we take an action by $$\epsilon$$-greedy, i.e. with probability $$\epsilon$$, $$a_t$$ is selected randomly, due to the the Exploration-Exploitation Dilemma.

## 5. deep Q-learning algorithm for deep Malphago

The main idea of deep Q-learning is to approximate the Q-function $$Q(s,a)$$ by a "deep model", i.e. replace the table like function by a neural network. First of all we need a vectorial representation of state $$\phi(s)$$, as illustrated by the table below, $$\phi(s)$$ is a 7-dim vector, where fist 6 dims are 2 hot encoders of (rock, paper, scissors) and last dim is the indicator of init_state.

|-----------------------------|-----------------|
| state $$s$$						|  $$\phi(s)$$    |
|-----------------------------|-----------------|
| init state                  |   0000001   |
| (rock, rock)                |   1001000   |
| (rock, paper)               |   1000100   |
| (rock, scissors)            |   1000010   |
| (paper, rock)               |   0101000   |
| (paper, paper)              |   0100100   |
| ...              				|     	        |
{:.post-table}

We'll have 3 $$Q_a(\phi(s))$$ for each $$a \in $$ (rock, paper, scissors), defined by a tensorflow graph of 2-layer feed-forwad neural network:

{%highlight python linenos%}
def deep_Q(X, tf_variable):
    W1, b1, W2, b2 = tf_variable
    hidden = tf.tanh(tf.matmul(X, W1) + b1)
    y_pred = tf.sigmoid(tf.matmul(hidden, W2) + b2)
    return y_pred
{%endhighlight%}

The tf.tanh() function at hidden layer makes Q non-linear and the tf.sigmoid() at output layer makes the Q-value between (-1, 1).

The deep Q-learning is referenced by Deepmind's algorithm of playing Atari games:

![]({{site.url}}/assets/image/deep_Q_learning_algo.png){:width="500px"}

As is shown in the algorithm, we set the target value $$y_j$$ for function $$Q_{a_j}(\phi(s_j))$$ as $$r_j + \gamma\, max_a \, Q_a(\phi(s_{j+1}))$$, and perform a descent gradient step on mean squared loss. We should notice that each time we train the model with only a fixed size of latest replays, which is because of our hypothesis that human player's strategy may change, so we only consider their recent behaviours to update our Q-function. 

## 6. Serve tensorflow model with Flask

The main obstacle is serving tensorflow model for each flask session. Flask-session stores session information at server side for each session id but the tensorflow objects can not be stored normally. The solution here is to only store parameters of a tensorflow graph as python list. Every time the server receive a request, it opens a tensorflow session and rebuild the graph from the stored parameters and then perform tensorflow operations.

## 7. Conclusions

As nature of the game rock-paper-scissors, there is no optimal strategy  nor strongest player. One can judge whether Malphago learns his strategy subjectively but there is no strict metric to evaluate the algorithms performance. Even we can not say whether the deep model is better than the simple version. At least we can conclude that Malpago got a not bad winning percentage. 

Hope you have fun with Malphago and thanks for reading.


