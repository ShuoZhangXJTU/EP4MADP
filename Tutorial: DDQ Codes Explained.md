# Tutorial: DDQ Codes Explained
  This tutorial explain the DDQ codes implemented in Pytorch.
  Source codes are available at https://github.com/MiuLab/DDQ

## Dataset Explained
  The dataset is partly drived from DSTC-8
  See: ***src\deep_dialog\data***
### Goal 
  Goal is loaded from ***user_goals_first_turn_template.part.movie.v1.new.p***
  
  ***goal_set*** has a form of ***{'train': [], 'valid': [], 'test': [], 'all': []}***, 
  with each element in list as ***{'request_slots': {}, 'diaact': 'request', 'inform_slots': {'city': 'birmingham', 'numberofpeople': '4', 'theater': 'carmike summit 16', 'state': 'al', 'starttime': 'around 6pm', 'date': 'today', 'moviename': 'deadpool'}}***
  

### Dictionary
  See 

## Major Procedure
  See: ***src\run.py***
  1. 
