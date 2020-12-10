# EP4MADP
Essential Details 

## Domain Reduction on MultiWOZ agenda-based user simulator in Convlab-2
```
**convlab2\task\multiwoz\goal_generator.py**
1. line 16, modify domains 
2. line 172, in GoalGenerator, modify self.domain_ordering_dist by:
        # ======== Reformulate the Rest Domains ==========
        # -- filter using ones
        target_domains = {'restaurant', 'hotel'}
        target_dod = deepcopy(self.domain_ordering_dist)
        for key_, prob_ in self.domain_ordering_dist.items():
            if not set(key_).issubset(target_domains):
                del target_dod[key_]
        # -- recalculate prob
        sum_prob = sum(list(target_dod.values()))
        for key_, ori_prob_ in target_dod.items():
            target_dod[key_] = ori_prob_ / sum_prob
        self.domain_ordering_dist = target_dod
        # =================================================

**convlab2\util\multiwoz\multiwoz_slot_trans.py**
1. modify REF_USR_DA & REF_SYS_DA

**convlab2\policy\rule\multiwoz\policy_agenda_multiwoz.py**
1. 
```
