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


## Modification: Data
  - convlab2\util\dataloader\dataset_dataloader.py
  1. class MultiWOZDataloader, apply DomainFiltingByGoal in function load_data, right after "for sess_id, sess in data.items():"
      ```
      def DomainFiltingByGoal(id, goal):
        """
            return True if need to skip
        """
        banned_id = [436, 3993, 4676, 5107, 5928, 6199, 7107, 7411, 7690, 9257,
                     9458, 2495, 2888, 3048, 4685, 4907, 5149, 6722, 8861]
        required_domains = {'restaurant', 'hotel'}
        domains = ['taxi', 'police', 'hospital', 'hotel', 'attraction', 'train', 'restaurant']
        if id in banned_id:
            return True
        else:
            for dom_ in domains:
                if dom_ not in required_domains and len(goal[dom_]) != 0:
                    return True
            return False
      ```
  2. Rebuild Datasets to update our domain filtering and the changes on vectors
