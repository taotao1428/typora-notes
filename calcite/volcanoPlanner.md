# VolcanoPlanner

注册节点时，会将每个RelNode封装成一个RelSubnet

## RelSubset

继承OptRelNode了，可以作为一个节点存在整个树中。

每个RelSubSet是RelSet的子集，子集之间通过TraitSet区分。







## RelMetadataQuery







## RelSet

 derived and required traitset

保存了相同的关系



RelSet中rels保存的relNode，都已经将input替换成RelSubnet了，

RelSet中parents保存的relNode是relNode而不是RelSubnet。