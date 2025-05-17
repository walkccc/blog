---
title: Reinforcement Learning on OpenAI Gym
author: Peng-Yu Chen
date: 2018-12-06
tags:
  - Reinforcement Learning
  - Python
---

{{< katex >}}

# Reinforcement Learning: OpenAI Gym

強化學習（Reinforcement Learning，以下簡稱 RL）有別與一般監督式學習（Supervised
Learning）與要 end-to-end 的資料訓練。

## Reinforcement Learning 介紹

> Agent 在某個 state 做了一個 action，而移到下一個 state，環境此時給了 agent 一
> 個 reward，而 agent 則根據這個 reward 調整他的腳步。

聽起來有點抽象？比喻來說：

你是 agent，而環境是家（包括：爸爸、媽媽等），action 是你的行為舉止，state 是你
所在的地點，reward 是媽媽給你的獎勵（可能是負的，即：負回饋）。

- 若你（agent）在客廳（state），打破了一個杯子（action），而媽媽斥責你（reward）
  ，你就會透過這次的不小心，去避免下次在客廳時不要再打破杯子。
- 相反地，若你（agent）在廚房（state），洗了碗筷（action），而媽媽說你很棒
  （reward），你就會知道，洗碗可以幫忙分擔家事並得到正回饋，所以以後一樣到了廚房
  看到有碗筷時，就會增加想洗碗的機率。

---

由上可知，RL 是 ML 家族中的一員，而我個人覺得他更有 AI 的感覺，他是一種目標導向
（goal-oriented）的學習方法，透過 agent 與環境間的互動獲得更種獎勵或懲罰，學會如
何做「最好的」決策（policy）。

整個決策決定過程可由以下 5 個要素組成：

1. Agent：與 environment 互動（action）。
1. Action：agent 藉由自己的 policy 所做的動作。
1. Environment：agent 的行動範圍，根據 agent 的 action 給予不同的 reward。
1. State：agent 在特定時間處的狀態。
1. Reward：environment 根據 agent 的 action 給的獎勵或懲罰。

# Algorithm Implementation

我們使用 OpenAI Gym 當中的 `CartPole-v0` 來實作 RL 演算法，OpenAL Gym 提供了各種
不同的 environment 來做 RL 的訓練。

為了避免跨平台上安裝的問題，我使用 Google Colab 的雲端平台來
[demo](https://colab.research.google.com/drive/1nellAKykrJOwySQZqZ8eagTmkR0PSXEE)。

[CartPole source code](https://github.com/openai/gym/blob/master/gym/envs/classic_control/cartpole.py)

## Random Action

先從最簡單的例子來了解相關變數，無論 environment 如何，隨機挑選一個可行的
action，即：隨機決定左移或右移。

```python
"""
Random Action
"""

env = gym.make('CartPole-v0')
env.reset()

# try 30 episodes
for i in range(30):
  obs = env.reset()
  rewards = 0

  # try 50 actions for each episode
  for t in range(50):
    action = env.action_space.sample()  # randomly choose 0 (left) or 1 (right)
    obs, reward, done, info = env.step(action)
    rewards += reward

    if done:
      print('episode {}: Episode finished after {} timesteps, total rewards {}'.format(
          i, t + 1, rewards))
      break

ipythondisplay.clear_output(wait=True)
env.close()
```

因為 agent 在這個 random choosing action 時，並沒有任何學習，所以 reward 普遍不
高。

## Hand-Made Policy

再來我們引進一個簡單的 policy：

- 如果柱子向左傾（角度 \\(< 0\\)），則小車左移以維持平衡。
- 如果柱子向左傾（角度 \\(\\ge 0\\)），則小車右移以維持平衡。

```python
"""
Hand-Made Policy
"""


# Define policy
def choose_action(obs):
  pos, v, ang, rot = obs
  return 0 if ang < 0 else 1    # 0: left, 1: right


env = gym.make('CartPole-v0')
env.reset()

for i in range(50):
  obs = env.reset()
  rewards = 0

  for t in range(250):
    action = choose_action(obs)
    obs, reward, done, info = env.step(action)
    rewards += reward

    if done:
      print('episode {}: Episode finished after {} timesteps, total rewards {}'.format(
          i, t + 1, rewards))
      break

env.close()
```

## Q-Learning with Q Table

$$Q ^ { \* } ( s , a ) = \sum _ { s ^ { \prime } } T \left( s , a , s ^ { \prime } \right) \left( R \left( s , a , s ^ { \prime } \right) + \gamma \max _ { a ^ { \prime } } Q ^ { * } \left( s ^ { \prime } , a ^ { \prime } \right) \right)$$

其中：

- \\(T\\)：transition function，\\(0 \le T \le 1\\) 表發生機率
- \\(R\\)：reward function
- \\(\gamma\\)：discount factor，通常會是一個 \\(< 1\\) 的值，可能是
  \\(0.9\\)、\\(0.8\\) 之類，代表的是對未來 reward 的重視程度。

<!-- $$Q(s, a) = r + \gamma\max_{a'} Q(s', a').$$ -->

<!-- 我們定義 Q function $Q(s, a)$，根據 agent 所處的 state $s$ 進行 action $a$ 所預期未來得到的 reward。 -->

所以我們的目的是：

$$\arg\max_a Q^*(s, a).$$

agent 透過一次次跟 environment 互動（a）獲得的 reward 來學習 Q function：

$$
Q(s_t, a_t) = (1 - \alpha) \cdot Q(s_{t - 1}, a_{t - 1}) + \alpha \cdot (r_t + \gamma \max Q(s_{t + 1}, a))
$$

其中，

- \\(t\\)：不同的時間點
- \\(\alpha\\)：learning rate

pseudo code 如下：

```
Initialize Q(s, a) randomly
for each episode
  Initialize s

  for each step of episode
      Choose a from s using policy derived from Q (e.g., ε-greedy)
      Take action a, observe r, s'
      Q(s, a) ← Q(s, a) + α[r + γ max_a' Q(s', a') - Q(s, a)]
      s ← s'
  until s is terminal
```

\\(\epsilon\\)-greedy 是一種在 exploration 和 exploitation 間取得平衡的方法。

- exploration 嘗試不同 action
- exploitation 沿用現有 policy

方法很簡單：

- \\(\epsilon\\) 時間，agent 嘗試新 action
- \\((1 - \epsilon)\\) 時間，agent 沿用現有 policy

```python
"""
Q-Learning
"""

import math


def choose_action(state, QTable, action_space, epsilon):
  if np.random.random_sample() < epsilon:    # P(randomly choose action) = ε
    return action_space.sample()
  # choose the action that maximize QTable[state]
  return np.argmax(QTable[state])


def get_state(obs, n_buckets, bds):
  state = [0] * len(obs)             # state = [0, 0, 0, 0]

  for i, s in enumerate(obs):        # each feature has different distribution
    l, u = bds[i][0], bds[i][1]    # each feature's lowerbound & upperbound
    if s <= bds[i][0]:             # lower than lowerbound
      state[i] = 0
    elif s >= bds[i][1]:           # higher than upperbound
      state[i] = n_buckets[i] - 1
    else:
      state[i] = int(((s - l) / (u - l)) * n_buckets[i])

  return tuple(state)


env = gym.make('CartPole-v0')

# Prepare Q table
# Each feature's n_bucket
# '1' represents that any value belongs to the same state, i.e., the feature is not important
n_buckets = (1, 1, 6, 3)
n_actions = env.action_space.n

# bounds of state
bds = list(zip(env.observation_space.low, env.observation_space.high))
bds[1] = [-0.5, 0.5]
bds[3] = [-math.radians(50), math.radians(50)]

# Q Table, each (s, a) pair stores one value
QTable = np.zeros(n_buckets + (n_actions,))


# epsilon-greedy, decrease by time
def epsilons(i):
  return max(0.01, min(1, 1.0 - math.log10((i + 1) / 25)))


# learning rate, decrease by time
def lrs(i):
  return max(0.01, min(0.5, 1.0 - math.log10((i + 1) / 25)))

gamma = 0.99  # reward discount factor

# Q-Learning
for i in range(200):
  obs = env.reset()
  rewards = 0
  # convert continuous value -> discrete value
  state = get_state(obs, n_buckets, bds)

  epsilon = epsilons(i)
  lr = lrs(i)

  for t in range(250):
    action = choose_action(state, QTable, env.action_space, epsilon)
    obs, reward, done, info = env.step(action)
    rewards += reward

    next_state = get_state(obs, n_buckets, bds)

    # update Q Table
    q_next_max = np.amax(QTable[next_state])
    QTable[state + (action,)] += lr * (reward + gamma *
                                       q_next_max - QTable[state + (action,)])  # Formula
    print(QTable)

    # step to next state
    state = next_state

    if done:
      print('Episode finished after {} timesteps, total rewards {}'.format(t+1, rewards))
      break

env.close()
```

## Deep Q-Learning

在 CartPole 的 task 中，state 只有 4 個 features，action 只有 0 和 1 兩個值。

但若 state 今天是來自整個環境（例如：整個螢幕、Alpha Go 圍棋棋盤），因為 states
過多，這時用 table 來表示就不太妥當。

Deep Q-Leanring（以下簡稱 DQN）：我們可以用 deep neural network 幫我們提取
features 並逼近 Q function。

NN 藉由大量的 input-output end-to-end 訓練，找出：

$$f(input) = output.$$

把它轉成 policy：

$$\pi(state) = action.$$

### Deep Q-Learning Implementation

在
[Human-level control through deep reinforcement learning](https://web.stanford.edu/class/psych209/Readings/MnihEtAlHassibis15NatureControlDeepRL.pdf)
中，提供了三種訓練 DQN 的 tips：

1. Use experience replay：把 experience 存在 memory，訓練時隨機抽樣。優點是可以
   打亂不同 experience 之間不存在的時間關係。
1. Freeze target Q-network：即建立兩種 Q-network，
   - 實際進行訓練的 evaluation network
   - 訓練目標 target network 若只訓練一個 nn，每更新一次時，不但正在訓練的
     $Q(s, a)$ 在變，目標 $Q(s', a')$ 也在變，這樣是無法收斂的！
1. Clip rewards：限縮 reward 的值，以利 backpropagation 中能穩定地計算
   gradient。

#### Step 1: 建立 Network

先建立一層 hidden layer，把 state 傳入後，得出每個 action 的分數，分數越高的
action 越容易被挑選。

目標：對未來越有利的 action 分數越高。

```python
class Net(nn.Module):
  def __init__(self, n_states, n_actions, n_hidden):
    super(Net, self).__init__()
    self.fc1 = nn.Linear(n_states, n_hidden)
    self.out = nn.Linear(n_hidden, n_actions)

  def forward(self, x):
    x = self.fc1(x)
    x = F.relu(x)
    return self.out(x)
```

#### Step 2: 建立 Deep Q-Network

Tips 中提到，總共需要兩個 network：

- evaluation network（`eval_net`）
- target network（`targ_net`）

和

- memory 儲存 experience

```python
# Deep Q-Network, composed of one eval network, one target network
class DQN(object):
  def __init__(self, n_states, n_actions, n_hidden, batch_size, lr, epsilon, gamma, target_replace_iter, memory_capacity):
    self.eval_net = Net(n_states, n_actions, n_hidden)
    self.target_net = Net(n_states, n_actions, n_hidden)

    # initialize memory, each memory slot is of size (state + next state + reward + action)
    self.memory = np.zeros((memory_capacity, n_states * 2 + 2))
    self.optimizer = torch.optim.Adam(self.eval_net.parameters(), lr=lr)
    self.loss_func = nn.MSELoss()
    self.memory_counter = 0
    self.learn_step_counter = 0  # for target network update

    self.n_states = n_states
    self.n_actions = n_actions
    self.n_hidden = n_hidden
    self.batch_size = batch_size
    self.lr = lr
    self.epsilon = epsilon
    self.gamma = gamma
    self.target_replace_iter = target_replace_iter
    self.memory_capacity = memory_capacity

  def choose_action(self, state):
    x = torch.unsqueeze(torch.FloatTensor(state), 0)

    # epsilon-greedy
    if np.random.uniform() < self.epsilon:  # random
      action = np.random.randint(0, self.n_actions)
    # greedy
    else:
      # feed into eval net, get scores for each action
      actions_value = self.eval_net(x)
      # choose the one with the largest score
      action = torch.max(actions_value, 1)[1].data.numpy()[0]

    return action

  def store_transition(self, state, action, reward, next_state):
    # Pack the experience
    transition = np.hstack((state, [action, reward], next_state))

    # Replace the old memory with new memory
    index = self.memory_counter % self.memory_capacity
    self.memory[index, :] = transition
    self.memory_counter += 1

  def learn(self):
    # Randomly select a batch of memory to learn from
    sample_index = np.random.choice(self.memory_capacity, self.batch_size)
    b_memory = self.memory[sample_index, :]
    b_state = torch.FloatTensor(b_memory[:, :self.n_states])
    b_action = torch.LongTensor(
        b_memory[:, self.n_states:self.n_states+1].astype(int))
    b_reward = torch.FloatTensor(b_memory[:, self.n_states+1:self.n_states+2])
    b_next_state = torch.FloatTensor(b_memory[:, -self.n_states:])

    # Compute loss between Q values of eval net & target net
    # evaluate the Q values of the experiences, given the states & actions taken at that time
    q_eval = self.eval_net(b_state).gather(1, b_action)
    # detach from graph, don't backpropagate
    q_next = self.target_net(b_next_state).detach()
    q_target = b_reward + self.gamma * \
        q_next.max(1)[0].view(self.batch_size,
                              1)  # compute the target Q values
    loss = self.loss_func(q_eval, q_target)

    # Backpropagation
    self.optimizer.zero_grad()
    loss.backward()
    self.optimizer.step()

    # Update target network every few iterations (target_replace_iter),
    # i.e. replace target net with eval net
    self.learn_step_counter += 1
    if self.learn_step_counter % self.target_replace_iter == 0:
      self.target_net.load_state_dict(self.eval_net.state_dict())
```

#### Step 3: 訓練

步驟如下：

1. 選擇 action
1. 儲存 experience
1. train

```python
if __name__ == '__main__':
  env = gym.make('CartPole-v0')
  env = env.unwrapped  # For cheating mode to access values hidden in the environment

  # Environment parameters
  n_actions = env.action_space.n
  n_states = env.observation_space.shape[0]

  # Hyper parameters
  n_hidden = 50
  batch_size = 32
  lr = 0.01                 # learning rate
  epsilon = 0.1             # epsilon-greedy, factor to explore randomly
  gamma = 0.9               # reward discount factor
  target_replace_iter = 100  # target network update frequency
  memory_capacity = 2000
  n_episodes = 400 if CHEAT else 4000

  # Create DQN
  dqn = DQN(n_states, n_actions, n_hidden, batch_size, lr,
            epsilon, gamma, target_replace_iter, memory_capacity)

  # Collect experience
  for i_episode in range(n_episodes):
    t = 0  # timestep
    rewards = 0  # accumulate rewards for each episode
    state = env.reset()  # reset environment to initial state for each episode
    while True:

      # Agent takes action
      action = dqn.choose_action(state)  # choose an action based on DQN
      next_state, reward, done, info = env.step(
          action)  # do the action, get the reward

      # Cheating part: modify the reward to speed up training process
      if CHEAT:
        x, v, theta, omega = next_state
        # reward 1: the closer the cart is to the center, the better
        r1 = (env.x_threshold - abs(x)) / env.x_threshold - 0.8
        # reward 2: the closer the pole is to the center, the better
        r2 = (env.theta_threshold_radians - abs(theta)) / \
            env.theta_threshold_radians - 0.5
        reward = r1 + r2

      # Keep the experience in memory
      dqn.store_transition(state, action, reward, next_state)

      # Accumulate reward
      rewards += reward

      # If enough memory stored, agent learns from them via Q-learning
      if dqn.memory_counter > memory_capacity:
        dqn.learn()

      # Transition to next state
      state = next_state

      if done:
        print('Episode finished after {} timesteps, total rewards {}'.format(t+1, rewards))
        break

      t += 1

  env.close()
```
