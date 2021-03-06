# tetris-ai

A bot that plays [tetris](https://en.wikipedia.org/wiki/Tetris) using deep reinforcement learning.

## Installation

```bash
sudo apt install libgtk2.0-dev pkg-config
# https://stackoverflow.com/a/14656610/5066426

conda create --name py37tf python=3.7
conda install -c conda-forge opencv=4.1.0
conda install tensorflow-gpu keras pillow tqdm tensorboard

conda activate py37tf
```
OR
```bash
conda create --name py37tf --file requirements.txt
conda activate py37tf
```


## How to use

This can be run in three modes: interactive, training and evaluating.

### Interactive mode

Obviously requires no training, just allows to play Tetris yourself as usual controlling pieces with the keyboard 
and the code itself shows how to use the model as well.
```bash
python run_interactive.py
```

### Train and evaluate

To train the model do this
```bash
python run_train.py
```
After the training process finished a subdir in `./logs` will be created. There will be 2 things inside: logs for the
[`Tensorboard`](https://www.tensorflow.org/guide/summaries_and_tensorboard) and `model.hdf` - the serialized form of
the model trained. 

After that you can run the agent against with the model

```bash
python run_eval.py
```
This takes the most recent subdirectory in `./logs`, loads the model from there and runs visual simulation of the agent.
The results usually vary from run to run, so some patience required.

## Demo

First 10000 points, after training.

`![Demo - First 10000 points](_img/demo.gif)`

## How does it work

#### Reinforcement Learning

At first, the agent will play random moves, saving the states and the given reward in a limited queue (replay memory). At the end of each episode (game), the agent will train itself (using a neural network) with a random sample of the replay memory. As more and more games are played, the agent becomes smarter, achieving higher and higher scores.

Since in reinforcement learning once an agent discovers a good 'path' it will stick with it, it was also considered an exploration variable (that decreases over time), so that the agent picks sometimes a random action instead of the one it considers the best. This way, it can discover new 'paths' to achieve higher scores.


#### Training

The training is based on the [Q Learning algorithm](https://en.wikipedia.org/wiki/Q-learning). Instead of using just the current state and reward obtained to train the network, it is used Q Learning (that considers the transition from the current state to the future one) to find out what is the best possible score of all the given states **considering the future rewards**, i.e., the algorithm is not greedy. This allows for the agent to take some moves that might not give an immediate reward, so it can get a bigger one later on (e.g. waiting to clear multiple lines instead of a single one).

The neural network will be updated with the given data (considering a play with reward *reward* that moves from *state* to *next_state*, the latter having an expected value of *Q_next_state*, found using the prediction from the neural network):

if not terminal state (last round): *Q_state* = *reward* + *discount* × *Q_next_state*
else: *Q_state* = *reward*


#### Best Action

Most of the deep Q Learning strategies used output a vector of values for a certain state. Each position of the vector maps to some action (ex: left, right, ...), and the position with the higher value is selected.

However, the strategy implemented was slightly different. For some round of Tetris, the states for all the possible moves will be collected. Each state will be inserted in the neural network, to predict the score obtained. The action whose state outputs the biggest value will be played.


#### Game State

It was considered several attributes to train the network. Since there were many, after several tests, a conclusion was reached that only the first four present were necessary to train:

- **Number of lines cleared**
- **Number of holes**
- **Bumpiness** (sum of the difference between heights of adjacent pairs of columns)
- **Total Height**
- Max height
- Min height
- Max bumpiness
- Next piece
- Current piece


#### Game Score

Each block placed yields 1 point. When clearing lines, the given score is *number_lines_cleared*^2 × *board_width*. Losing a game subtracts 1 point.


## Implementation

All the code was implemented using `Python`. For the neural network, it was used the framework `Keras` with `Tensorflow` as backend.

#### Internal Structure

The agent is formed by a deep neural network, with variable number of layers, neurons per layer, activation functions, loss function, optimizer, etc. By default, it was chosen a neural network with 2 hidden layers (32 neurons each); the activations `ReLu` for the inner layers and the `Linear` for the last one; `Mean Squared Error` as the loss function; `Adam` as the optimizer; `Epsilon` (exploration) starting at 1 and ending at 0, when the number of episodes reaches 75%; `Discount` at 0.95 (significance given to the future rewards, instead of the immediate ones).

#### Training

For the training, the replay queue had size 20000, with a random sample of 512 selected for training each episode, using 1 epoch.


#### Requirements

- Tensorflow (`tensorflow-gpu==1.14.0`, CPU version can be used too)
- Tensorboard (`tensorboard==1.14.0`)
- Keras (`Keras==2.2.4`)
- Opencv-python (`opencv-python==4.1.0.25`)
- Numpy (`numpy==1.16.4`)
- Pillow (`Pillow==5.4.1`)
- Tqdm (`tqdm==4.31.1`)

## Results

For 2000 episodes, with epsilon ending at 1500, the agent kept going for too long around episode 1460, so it had to be terminated. Here is a chart with the maximum score every 50 episodes, until episode 1450:

![results](_img/results.svg)

Note: Decreasing the `epsilon_end_episode` could make the agent achieve better results in a smaller number of episodes.


## Useful Links

#### Deep Q Learning
- PythonProgramming - https://pythonprogramming.net/q-learning-reinforcement-learning-python-tutorial/
- Keon - https://keon.io/deep-q-learning/
- Towards Data Science - https://towardsdatascience.com/self-learning-ai-agents-part-ii-deep-q-learning-b5ac60c3f47

#### Tetris
- Code My Road - https://codemyroad.wordpress.com/2013/04/14/tetris-ai-the-near-perfect-player/ (uses evolutionary strategies)

### Debug notes

```
    (x, rotation) -> State
    State = [lines, holes, total_bumpiness, sum_height]


`env.get_next_states()` gives
    ( 0,   0) = {list} <class 'list'>: [0, 0, 5, 4]
    ( 1,   0) = {list} <class 'list'>: [0, 0, 6, 4]
    ( 2,   0) = {list} <class 'list'>: [0, 0, 6, 4]
    ( 3,   0) = {list} <class 'list'>: [0, 0, 6, 4]
    ( 4,   0) = {list} <class 'list'>: [0, 0, 6, 4]
    ( 5,   0) = {list} <class 'list'>: [0, 0, 6, 4]
    ( 6,   0) = {list} <class 'list'>: [0, 0, 6, 4]
    ( 7,   0) = {list} <class 'list'>: [0, 0, 6, 4]
    ( 8,   0) = {list} <class 'list'>: [0, 0, 3, 4]
    ( 0,  90) = {list} <class 'list'>: [0, 2, 2, 6]
    ( 1,  90) = {list} <class 'list'>: [0, 2, 4, 6]
    ( 2,  90) = {list} <class 'list'>: [0, 2, 4, 6]
    ( 3,  90) = {list} <class 'list'>: [0, 2, 4, 6]
    ( 4,  90) = {list} <class 'list'>: [0, 2, 4, 6]
    ( 5,  90) = {list} <class 'list'>: [0, 2, 4, 6]
    ( 6,  90) = {list} <class 'list'>: [0, 2, 4, 6]
    ( 7,  90) = {list} <class 'list'>: [0, 2, 2, 6]
    (-1, 180) = {list} <class 'list'>: [0, 2, 3, 6]
    ( 0, 180) = {list} <class 'list'>: [0, 2, 6, 6]
    ( 1, 180) = {list} <class 'list'>: [0, 2, 6, 6]
    ( 2, 180) = {list} <class 'list'>: [0, 2, 6, 6]
    ( 3, 180) = {list} <class 'list'>: [0, 2, 6, 6]
    ( 4, 180) = {list} <class 'list'>: [0, 2, 6, 6]
    ( 5, 180) = {list} <class 'list'>: [0, 2, 6, 6]
    ( 6, 180) = {list} <class 'list'>: [0, 2, 6, 6]
    ( 7, 180) = {list} <class 'list'>: [0, 2, 3, 6]
    ( 0, 270) = {list} <class 'list'>: [0, 0, 2, 4]
    ( 1, 270) = {list} <class 'list'>: [0, 0, 4, 4]
    ( 2, 270) = {list} <class 'list'>: [0, 0, 4, 4]
    ( 3, 270) = {list} <class 'list'>: [0, 0, 4, 4]
    ( 4, 270) = {list} <class 'list'>: [0, 0, 4, 4]
    ( 5, 270) = {list} <class 'list'>: [0, 0, 4, 4]
    ( 6, 270) = {list} <class 'list'>: [0, 0, 4, 4]
    ( 7, 270) = {list} <class 'list'>: [0, 0, 3, 4]

`next_states.values`
dict_values([
    [0, 0, 5, 4],
    [0, 0, 6, 4],
    [0, 0, 6, 4],
    [0, 0, 6, 4],
    [0, 0, 6, 4],
    [0, 0, 6, 4],
    [0, 0, 6, 4],
    [0, 0, 6, 4],
    [0, 0, 3, 4],
    [0, 2, 2, 6],
    [0, 2, 4, 6],
    [0, 2, 4, 6],
    [0, 2, 4, 6],
    [0, 2, 4, 6],
    [0, 2, 4, 6],
    [0, 2, 4, 6],
    [0, 2, 2, 6],
    [0, 2, 3, 6],
    [0, 2, 6, 6],
    [0, 2, 6, 6],
    [0, 2, 6, 6],
    [0, 2, 6, 6],
    [0, 2, 6, 6],
    [0, 2, 6, 6],
    [0, 2, 6, 6],
    [0, 2, 3, 6],
    [0, 0, 2, 4],
    [0, 0, 4, 4],
    [0, 0, 4, 4],
    [0, 0, 4, 4],
    [0, 0, 4, 4],
    [0, 0, 4, 4],
    [0, 0, 4, 4],
    [0, 0, 3, 4]
])
```
