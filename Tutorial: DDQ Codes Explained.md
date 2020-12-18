# Tutorial: DDQ Codes Explained
  This tutorial explains the DDQ codes implemented in Pytorch.
  Source codes are available at https://github.com/MiuLab/DDQ

## Dataset Explained
The Microsoft Research dataset available at https://github.com/xiul-msr/e2e_dialog_challenge contains three single domains of movie, restaurant and taxi:

|Task|Intents|Slots|Dialogues|
| -----| ----- | ----- | ----- |
|Movie-Ticket Booking|11|29|2890|
|Restaurant Reservation|11|30|4103|
|Taxi Ordering|11|29|3094|

### SL Dataset for Dialog Policy
`movie_all_wState.p`, `restaurant_all_wState.p` and `taxi_all_wState.p` available at https://github.com/leishu02/EMNLP2019_gCAS/tree/master/data

* To **load data with python3**, apply `rb` in `open` and `latin1` in `pickle.load` to avoid errors.

    ```
    data = pickle.load(open('movie_all_wState.p', 'rb'), encoding='latin1')
    ```
* **Data format** 

    `data` is a dict of `{ID:[turn_dict], ...}`

    `turn_dict` is a dict with keys as: `state`, `msr_agent_act`, `agent_act`, `msr_user_act`, `user_act`, `turn_id` 
    
    * `state`
    
        ```
        {
          'agent_action': None, 
          
          'user_action': {
              'request_slots': {'ticket': 'UNK'}, 
              'turn': 0, 
              'speaker': 'user', 
              'inform_slots':  {'city': 'Seattle', ...}, 
              'diaact': 'request'
          }, 
          
          'turn': 1, 
          
          'current_slots': {
              'request_slots': {'ticket': 'UNK'}, 
              'agent_request_slots': {}, 
              'inform_slots': {'city': 'Seattle', ...}, 
              'proposed_slots': {}
          },
          
          'kb_results_dict': {'city': 463, ...}, 
          
          'history': [{
                  'request_slots': {'ticket': 'UNK'}, 
                  'turn': 0, 
                  'speaker': 'user', 
                  'inform_slots': {'city': 'Seattle', ...}, 
                  'diaact': 'request'}
          ]
       }

        ```
                    
    * `msr_agent_act`

        ```
        {
          'act_slot_response': {
              'request_slots': {}, 
              'turn': 1, 
              'diaact': 'inform', 
              'inform_slots': {
                  'city': 'Seattle', ...
              }
          }, 
          'act_slot_response_slot': None
        }
        ```

    * `agent_act`

        ```
        [('inform', {'taskcomplete': '', 'numberofpeople': '2', ...}), ...]
        ```

    * `msr_user_act` 

        ```
        {
          'request_slots': {'ticket': 'UNK'}, 
          'diaact': 'request', 
          'inform_slots': {'city': 'Seattle', ...}
        }
        ```

    * `user_act`

      ```
        [('request', {'city': 'Seattle', ...}), ...]
      ```
    
    * `turn_id` is of int type
    

### RL Environment for Dialog Policy
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
      
  * `state`
  
    `state` is a dict with keys `history_slots`, `infrom_slots`, `request_slots`, `rest_slots`, `turn`, `diaact`
    
    vectorization done in src\deep_dialog\agents\agent_dqn.py `prepare_state_representation`
  
  * `act`
    vectorization is done in src\deep_dialog\agents\agent_dqn.py `action_index`

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
 
### DialogManager
  Brief Intro on whole interaction procedure.
  1. `initialize_episode`
      1. init `state_tracker`, empty `state` 
      2. init `running_user`, empty `state`, sample `goal` (Note: goal is shared between `RuleSimulator` and `world_model`), sample first `user_action`
  2. loop `next_turn`
      1. get raw `state` and pass to `agent.state_to_action` to get raw `agent_action`
      2. `state_tracker` update
      3. get `state_user` and env return `user_action`
          1. `self.state` turn += 2
          2. vectorize `s` and encode `self.sample_goal`
          3. call `predict` and instantiate `user_action`
          4. fill first `inform_slot` with corresponding `goal` if possible, else fill `I_DO_NOT_CARE`
      4. `state_traker` update
  
 
## Algorithms 
  This section gives a detailed look at each model.

### DQN
  Copied from https://github.com/ConvLab/ConvLab/blob/master/convlab/agent/algorithm/dqn.py
  
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
  Copied from https://github.com/ConvLab/ConvLab/blob/master/convlab/agent/algorithm/ppo.py
  
    ```
    for iteration = 1, 2, 3, ... do
        for actor = 1, 2, 3, ..., N do
            run policy pi_old in env for T timesteps
            compute advantage A_1, ..., A_T
        end for
        optimize surrogate L wrt theta, with K epochs and minibatch size M <= NT
    end for
    ```
