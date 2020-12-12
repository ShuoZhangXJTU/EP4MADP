# Tutorial: DDQ Codes Explained
  This tutorial explain the DDQ codes implemented in Pytorch.
  Source codes are available at https://github.com/MiuLab/DDQ

## Dataset Explained
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
  See: ***src\run.py***
  
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
  
