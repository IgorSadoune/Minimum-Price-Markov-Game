# Minimum Price Markov Game (MPMG) Environment

## Overview

`mpmg` is a modular environment designed for studying the Minimum Price Markov Game (MPMG), a concept in game theory and algorithmic game theory. It provides an easy-to-use framework for conducting experiments with multiple agents using collusion and cooperation dynamics. This environment is useful for researchers and developers interested in game theory, reinforcement learning, and multi-agent systems.

The paper introducing the MPMG can be find here: https://arxiv.org/abs/2407.03521

## Features
- **Customizable Multi-Agent Environment**: Supports different numbers of agents and heterogeneous vs. homogeneous settings.

## Installation

To install the package locally, run the following command from the root directory:

```sh
pip install mpmg
```

This installs the package in "editable" mode, meaning any changes made in the source code will immediately reflect in the installed package.

## Requirements
- Python 3.6+

## MPMG Basics
The MPG involves $n$ agents, represented by a set $\mathcal{N} = \{1,\dots,n\}$, competing to win a contract valued at $v$ in a first-price procurement auction.

**Payoffs**
All players are symmetric in their strategy set. Indeed, each agent $i$ chooses a strategy $s_i \in S$ where $S = \{\textit{FP}, \textit{CP}\}$. The strategy **Fair Price (FP)** is associated with bidding the fair price $b_i$, and **Collusive Price (CP)** is associated with bidding a higher price, defined by $\alpha \cdot b_i$, where $\tau \geq \alpha > 1$, with the upper bound $\tau$ ensuring realistic bids. We denote the strategy profile for a given occurrence as the tuple $(s_1, s_2, \dots, s_n)$.

**Heterogeneity**
Each agent $i$ has a market power defined by the parameter $\beta_i \in (0,1)$, with $\sum_{i \in \mathcal{N}} \beta_i = 1$, indicating that the average market strength, $\mu(\beta)$, is always $\frac{1}{n}$. This market power might represent the size of the agent or its market share, influencing its ability to reduce costs and leverage economies of scale. All agents share the same cost function, which determines their bids:

$$
b_i = (1-\beta_i)v \quad \forall i \in \mathcal{N}
$$

The notion of strong and weak agents is relative to the distribution of $\beta$, but according to the above equation, the closer $\beta$ is to 1, the lower the agent can bid, thus the stronger it is. Market heterogeneity is quantified by $\sigma(\beta)$, the standard deviation of the distribution of power parameter values. In a perfectly homogeneous market, $\sigma(\beta) = 0$ and $\beta_i = \frac{1}{n}$ for all $i$.

**Payoffs**

Let \( u_i(\textit{FP}, k) \) represent the payoff of agent \( i \) when it bids the fair price and \( k \) opponents also bid the fair price, and let \( u_i(\textit{CP}, k) \) denote the payoff of agent \( i \) when it bids the collusive price while \( k \) opponents bid the fair price, where \( k \in [0, n-1] \). The key idea from the minimum price rule is that coordination must be unanimous to achieve collusive profits. Therefore, the individual payoff for playing **CP** when all players cooperate surpasses the individual payoff from universal defection, i.e.,

$$
u_i(\textit{CP}, k=0) > u_i(\textit{FP}, k=n-1)
$$

However, defection must always yield a higher payoff than cooperation when at least one opponent defects, leading to

$$
u_i(\textit{FP}, k>0) > u_i(\textit{CP}, k>0)
$$

In fact, we can set \( u_i(\textit{CP}, k>0) = 0 \), meaning that when some players defect (FP) while others cooperate (CP), the cooperating agents receive no payoff, while defecting agents earn a share based on their market strengths relative to the defecting opponents. Let \( \Omega \subseteq \mathcal{N} \) denote the set of agents playing **FP**, and let the total market power of the defecting agents be \( \beta_\Omega = \sum_{j \in \Omega} \beta_j \). Assuming symmetric play where \( \beta_{\Omega} = 1 \), the payoffs in the general heterogeneous case are given in the table below.

<table>
    <thead>
        <tr>
            <th>Strategy Profile</th>
            <th>$$ k = 0 $$</th>
            <th>$$ k > 0 $$</th>
            <th>$$ k = n-1 $$</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>$$ u_i(\textit{FP}, k) $$</td>
            <td>$$ b_i $$</td>
            <td>$$ \frac{\beta_i}{\beta_{\Omega}} \cdot b_i $$</td>
            <td>$$ \beta_i \cdot b_i $$</td>
        </tr>
        <tr>
            <td>$$ u_i(\textit{CP}, k) $$</td>
            <td>$$ \alpha \cdot \beta_i \cdot b_i $$</td>
            <td>$$ 0 $$</td>
            <td>$$ 0 $$</td>
        </tr>
    </tbody>
</table>




## Usage

### Input Parameters
```
num_agents (int): Number of agents. Must be a positive integer, default value is 2.

sigma_beta (float): Heterogeneity level, standard deviation of the power parameters' distribution. Must be in [0,1], default value is 0.

alpha (float): Collusive bid multiplier. Must be > 1.
```

### Methods And Attributes
The `MPMGEnv` class provides methods for resetting the environment, taking steps, and observing the state, rewards, and dynamics of multi-agent interactions.

```
Methods
-------

  reset():
    reset the environment, and returns the initial state
    input: no input
    output: np.ndarray
  
  step(actions):
    returns rewards, next_state and  the done use in episodic task.
    input: List[int]  
    output: (np.ndarray, np.ndarray, bool)

Attributes
----------
num_agents (int): Number of agents.

sigma_beta (float): Heterogeneity level.

alpha (float): Collusive bid multiplier. 

action_size (int): action space size, which is always 2.

joint_action_size (int): action_size ** num_agents, joint action space size.

beta_size (int): num_agents, the size of the beta parameters array.

state_size (int): num_agents + joint_action_size + beta_size. Size of the observation space. May change upon customization of the state space.

state_space: The observation space is composed of 'action_frequencies', 'joint_action_frequencies', and 'beta_parameters', and is of size state_size.

action_frequencies (np.ndarray(num_agents)): action frequencies of action 1 for each player.         

joint_action_frequencies (np.ndarray(joint_action_size)): joint action frequencies for each joint action.
```

Example use:

```python

# import the environment
from mpmg import MPMGEnv

# Create an instance of the environment
env = MPMGEnv(n_agents=2, sigma_beta=0.0, alpha=1.3)

# Reset the environment
state = env.reset() 

# Probably a loop here
for i in range(...):

  # Sample actions
  actions = [1, 0]  # Example of actions array for 2-players

  # Take a step in the environment
  rewards, next_state, done = env.step(actions)
  
  # Do what you need
  ...
  
  # Update state
  state = next_state
```

## Scenarios
`MPMGEnv` is a social dilemma based on the Prisoner's Dilemma. 

- **Full Defection**: All agents choose to defect (action 0), Nash Equilibrium.
- **Full Cooperation**: All agents cooperate (action 1), Pareto Optimal.
- **Asymmetric play**: actions taken can be separated into two sets, other suboptimal outcome.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## Contributing

Contributions are welcome! Feel free to open an issue or submit a pull request for improvements, bug fixes, or new features.

## Author

Igor Sadoune - igor.sadoune@polymtl.ca

