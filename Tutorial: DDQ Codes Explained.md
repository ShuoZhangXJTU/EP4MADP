# Tutorial: DDQ Codes Explained
  This tutorial explains the DDQ codes implemented in Pytorch.
  Source codes are available at https://github.com/MiuLab/DDQ

## Dataset Explained
  This section reports the brief usage of the dataset to build env.
  The dataset is partly driven by DSTC-8
  See: `src\deep_dialog\data`
  
  * `Goal` 
  
      `goal_set` is `{'train': [], 'valid': [], 'test': [], 'all': []}`
      
      with elements as `{'request_slots': {slot:val, ...}, 'diaact': 'request', 'inform_slots': {slot:val, ...}}`

  * `Knowledge Base (KB)` 
  
      `movie_kb` is `{ID:{slot:val, ...}}`
  
  * `Act / Slot Set` 
  
      `act_set` is `{intent: ID, ...}`
      
      `slot_set` is `{slot: ID, ...}`

  * `Dictionary`
  
      `movie_dictionary` is `{slot: val_list}`

## Major Procedure
  This section gives an overview of `src\run.py`, note every time, we train in single domain, not joint space.
  
  1. Load Data: goal, kb, act/slot set, dictionary
  2. Set `AgentDQN` as agent, `RuleSimulator` as env  
  3. Set planning for agent with world model `ModelBasedSimulator`
  4. Set `DialogManager` with agent, env, world, act_set, slot_set, kb
  5. Start `run_episodes`
      1. `warm_start_simulation` if possible, train world
      2. train loop episodes
          1. test current DQN 
          2. `simulation_epoch_for_training`, Plannning
          3. `simulation_epoch`, Regular Interaction 
          4. if simulation success > current best and threshold, then `simulation_epoch_for_training`
          5. update best if needed
          6. `agent.train` and `reset_dqn_target`
          7. train `world_model` if set to train
  
## Detailed Procedures
  This section gives a detailed look at each procedure function.
  
  * `warm_start_simulation` 
  
      This is used to boost `world_model`
      1. set to use `RuleSimulator` as env
      2. `record_training_data` and `record_training_data_for_user` 
      3. train `world_model` if `param: boosted`
  
  * `simulation_epoch_for_training` 
  
      This is used perform `Planning` through the interaction with chosen env
      1. If not `param: grounded` set to use `RuleSimulator` as env at first episode, then all set to `world_model`, 
      else always use `RuleSimulator` as env
      2. `record_training_data` and `record_training_data_for_user` 
  
  * `simulation_epoch` 
  
      This is used perform regular interaction with rule env
      1. set to use `RuleSimulator` as env
      2. `record_training_data` only


## Modules 
  This section gives a detailed look at each Module.

 
## Algorithms 
  This section gives a detailed look at each model.

### DQN
  Copied from [https://github.com/ConvLab/ConvLab/blob/master/convlab/agent/algorithm/dqn.py]
  
    ```
    1. Collect some examples by acting in the environment and store them in a replay memory
    2. Every K steps sample N examples from replay memory
    3. For each example calculate the target (bootstrapped estimate of the discounted value of the state and action taken), y, using a neural network to approximate the Q function. s' is the next state following the action actually taken.
            y_t = r_t + gamma * argmax_a Q(s_t', a)
    4. For each example calculate the current estimate of the discounted value of the state and action taken
            x_t = Q(s_t, a_t)
    5. Calculate L(x, y) where L is a regression loss (eg. mse)
    6. Calculate the gradient of L with respect to all the parameters in the network and update the network parameters using the gradient
    7. Repeat steps 3 - 6 M times
    8. Repeat steps 2 - 7 Z times
    9. Repeat steps 1 - 8
    ```
    
### PPO
  Copied from [https://github.com/ConvLab/ConvLab/blob/master/convlab/agent/algorithm/ppo.py]
  
    ```
    for iteration = 1, 2, 3, ... do
        for actor = 1, 2, 3, ..., N do
            run policy pi_old in env for T timesteps
            compute advantage A_1, ..., A_T
        end for
        optimize surrogate L wrt theta, with K epochs and minibatch size M <= NT
    end for
    ```
