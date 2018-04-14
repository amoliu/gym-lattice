# gym-lattice

[![Build Status](https://travis-ci.org/ljvmiranda921/gym-lattice.svg?branch=master)](https://travis-ci.org/ljvmiranda921/gym-lattice)
![python 3.4+](https://img.shields.io/badge/python-3.4+-blue.svg)
[![DOI](https://zenodo.org/badge/127895338.svg)](https://zenodo.org/badge/latestdoi/127895338)

An HP 2D Lattice Environment with a Gym-like API for the protein folding
problem

This is a Python library that formulates Lau and Dill's (1989)
hydrophobic-polar two-dimensional lattice model as a reinforcement learning
problem. It follows OpenAI Gym's API, easing integration for reinforcement
learning solutions. This library only implements a two-dimensional square
lattice, but different lattice structures will be done in the future.

Screenshots from `render()`:

![](/assets/demo1.png)
![](/assets/demo2.png)
![](/assets/demo3.png)

## Dependencies

- numpy==1.14.2
- gym==0.10.4
- six==1.11.0
- pytest==3.2.1 *(dev)*
- setuptools==39.0.1 *(dev)*

## Installation

This package is only compatible for Python 3.4 and above. To install this
package, please follow the instructions below:

1. Install [OpenAI Gym](https://gym.openai.com/docs/#installation) and its dependencies.
2. Install the package itself:

```
git clone https://github.com/ljvmiranda921/gym-lattice.git
cd gym-lattice
pip install -e .
```

## Environment

The objective of this environment is to find a configuration with the highest
number of adjacent **H-H** pairs given a sequence consisting of *H* and *P*
molecules.

<img src="/assets/pfolding_problem.svg" width="700">

You can only put a molecule around (left, down, up, right) the one you've
previously placed. Assigning a molecule to an occupied space incurs a
penalty, while trapping yourself and running out of moves will incur a heavy
deduction.

<img src="/assets/pfolding_penalty.svg" width="700">

## Basic Usage

Initializing the environment will require the protein sequence of type `str`.
In addition, you can also set the amount of penalty incurred during a
collision or whenever the agent is trapped.

Let's try this out with a random agent on the protein sequence **HHPHH**!

```python
from gym_lattice.envs import Lattice2DEnv
from gym import spaces
import numpy as np

np.random.seed(42)

seq = 'HHPHH' # Our input sequence
action_space = spaces.Discrete(4) # Choose among [0, 1, 2 ,3]
env = Lattice2DEnv(seq)

for i_episodes in range(5):
    env.reset()
    while True:
        # Random agent samples from action space
        action = action_space.sample()
        obs, reward, done, info = env.step(action)
        env.render()
        if done:
            print("Episode finished! Reward: {} | Collisions: {} | Actions: {}".format(reward, info['collisions'], info['actions']))
            break
```

Sample output:

```
Episode finished! Reward: -2 | Collisions: 1 | Actions: ['U', 'L', 'U', 'U']
Episode finished! Reward: -2 | Collisions: 0 | Actions: ['D', 'L', 'L', 'U']
Episode finished! Reward: -2 | Collisions: 0 | Actions: ['R', 'U', 'U', 'U']
Episode finished! Reward: -3 | Collisions: 1 | Actions: ['U', 'L', 'D', 'D']
Episode finished! Reward: -2 | Collisions: 2 | Actions: ['D', 'R', 'R', 'D']
```

### Actions

Your agent can perform four possible actions: `0` (left), `1` (down), `2`
(up), and `3` (right). The number choices may seem funky at first but just
remember that it maps to the standard vim keybindings.

### Observations

Observations are represented as a 2-dimensional array with `1` representing
hydrophobic molecules (H), `-1` for polar molecules (P), and `0` for free
spaces. If you wish to obtain the chain itself, you can do so by accessing
`info['state_chain']`.

An episode ends when all polymers are added to the lattice OR if the sequence
of actions traps the polymer chain (no more valid moves because surrounding
space is fully-occupied). Whenever a collision is detected, the agent should
enter another action.

### Rewards

This environment computes the reward in the following manner:

```python
# Reward at timestep t
reward_t = state_reward + collision_penalty + trap_penalty
```

- The `state_reward` is the number of adjacent H-H molecules in the **final** state. In protein folding, the state_reward is synonymous to computing the Gibbs free energy, i.e., thermodynamic assumption of a stable molecule. Its value is 0 in all timesteps and is only computed at the end of the episode.  
- The `collision_penalty` at timestep t accounts for collision events whenever the agent chooses to put a molecule at an already-occupied space. Its default value is -2, but this can be adjusted by setting the `collision_penalty` at initialization.
- The `trap_penalty` is only computed whenever the agent has no more moves left and is unable to finish the task. The episode ends, thus computing the `state_reward`, but subtracts a deduction dependent on the length of the actual sequence.

## Cite us!

```
@misc{miranda2018gymlattice,
  author       = {Lester James V. Miranda},
  title        = {ljvmiranda921/gym-lattice: Initial Release},
  month        = apr,
  year         = 2018,
  doi          = {10.5281/zenodo.1214968},
  url          = {https://doi.org/10.5281/zenodo.1214968}
}
```
