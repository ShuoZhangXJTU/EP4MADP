# Tutorial: DDQ Codes Explained
  This tutorial explain the DDQ codes implemented in Pytorch.
  Source codes are available at https://github.com/MiuLab/DDQ

## Dataset Explained
  This section reports the brief usage of dataset to build env.
  The dataset is partly drived from DSTC-8
  See: ***src\deep_dialog\data***
  
  * **Goal** 
  
      ***goal_set*** is ***{'train': [], 'valid': [], 'test': [], 'all': []}***
      
      element as ***{'request_slots': {slot:val, ...}, 'diaact': 'request', 'inform_slots': {slot:val, ...}}***

  * **Knowledge Base (KB)**  
  
      ***movie_kb*** is ***{ID:{slot:val, ...}}***
  
  * **Act / Slot Set** 
  
      ***act_set*** is ***{intent: ID, ...}***
      
      ***slot_set*** is ***{slot: ID, ...}***

  * **Dictionary**
  
      ***movie_dictionary*** is ***{slot: val_list}***

## Major Procedure
  This section gives a overview of ***src\run.py***
  
  1. Load Data: goal, kb, act/slot set, dictionary
  2. Set ***AgentDQN*** as agent, ***RuleSimulator*** as env  
  3. Set planning for agent with world model ***ModelBasedSimulator***
  4. Set ***DialogManager*** with agent, env, world, act_set, slot_set, kb
  5. Start ***run_episodes***
      1. ***warm_start_simulation*** if possible
      2. train loop episodes
          1. test current DQN 
          2. ***simulation_epoch_for_training***
          3. ***simulation_epoch***
          4. if simulation success > current best and threshold, then ***simulation_epoch_for_training***
          5. update best if needed
          6. agent ***train*** and ***reset_dqn_target***
          7. train ***world_model*** if set to
  
## Detailed Procedures
  This section gives a detailed look at each procedure function.
  
  * **warm_start_simulation** 
  
      This is used to boost ***world_model***
      1. ***record_training_data*** and ***record_training_data_for_user***
      2. train ***world_model*** if para: boosted
  
  
 
## Models
  This section gives a detailed look at each model.
