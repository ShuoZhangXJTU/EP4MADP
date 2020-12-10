# Tutorial: Domain Reduction 
This tutorial gives a detailed procedure of changing the domains on MultiWOZ agenda-based user simulator in Convlab-2.


## Modification: Goal 
  - convlab2\task\multiwoz\goal_generator.py
    1. line 16, modify domains 
    2. line 172, in GoalGenerator, modify self.domain_ordering_dist
       ```
       #-------- Reformulate the Rest Domains ----------
       #-- filter using ones
       target_domains = {'restaurant', 'hotel'}
       target_dod = deepcopy(self.domain_ordering_dist)
       for key_, prob_ in self.domain_ordering_dist.items():
           if not set(key_).issubset(target_domains):
               del target_dod[key_]
       #-- recalculate prob
       sum_prob = sum(list(target_dod.values()))
       for key_, ori_prob_ in target_dod.items():
           target_dod[key_] = ori_prob_ / sum_prob
       self.domain_ordering_dist = target_dod
       #-------------------------------------------------
       ```
       
  - convlab2\util\multiwoz\multiwoz_slot_trans.py
    1. modify REF_USR_DA & REF_SYS_DA


## Modification: Vector
  - convlab2\policy\vector\vector_multiwoz.py
    1. modify self.belief_domains & self.db_domains, note this filter out belief state to build smaller state vectors,
    belief state is modeled as a 0/1 vector representing all the slots, where 1 reprensents value is not None.
    2. filter out self.da_voc & self.da_voc_opp, opp means opposite role (e.g., user to agent), 
    this will result in different action space (vector) and state space (because state contains actions)

## Modification: DST
  - convlab2\dst\rule\multiwoz\dst.py
    1. modify default_state


## Modification: Evaluator
  - convlab2\evaluator\multiwoz_eval.py
    1. modify requestable, mapping
    
