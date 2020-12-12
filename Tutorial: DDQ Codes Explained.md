# Tutorial: DDQ Codes Explained
  This tutorial explain the DDQ codes implemented in Pytorch.
  Source codes are available at https://github.com/MiuLab/DDQ

## Dataset Explained
  The dataset is partly drived from DSTC-8
  See: ***src\deep_dialog\data***
  * Goal 
      Goal is loaded from ***user_goals_first_turn_template.part.movie.v1.new.p***

      ***goal_set*** has a form of ***{'train': [], 'valid': [], 'test': [], 'all': []}***, 
      with element as ***{'request_slots': {slot:val, ...}, 'diaact': 'request', 'inform_slots': {slot:val, ...}}***

  * Knowledge Base (KB) 
      KB is loaded from ***movie_kb.1k.new.p***
      ***movie_kb*** has a form of ***{ID:{slot:val, ...}}***
  
  * Knowledge Base (KB) 
      KB is loaded from ***movie_kb.1k.new.p***
      ***movie_kb*** has a form of ***{ID:{slot:val, ...}}***

  * Dictionary
      See 

## Major Procedure
  See: ***src\run.py***
  1. 
