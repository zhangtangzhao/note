---
sort: 4
---

# zookeeper

## CAP理论
C：一致性 A：可用性  P：分区容错性
一个分布式系统不可能同时满足这三个基本需要，最多只能同时满足其中两项

## BASE理论
即使无法做到强一致性，但分布式系统可以根据自己的业务特点，采用适当的方式来使系统达到最终一致性

Basically Avaliable 基本可用：当分布式系统出现不可预见的故障时，允许损失部分可用性，保障系统的"基本可用"；体现在"时间上的损失"和"功能上的损失"；
Soft state 软状态：允许系统中的数据存在中间状态，即系统的不同节点的数据副本之间的数据同步过程存在延迟，并认为这种延时不会影响系统的可用性
Eventually consistent 最终一致性：所有的数据在经过一段时间的数据同步后，最终能够达到一个一致的状态

| What    | Follows  |
| ------- | -------- |
| A table | A header |
| A table | A header |
| A table | A header |

## zookeeper的配置参数

| 序号 | 参数名  |
| - | -------- |
| 1 | A header |
| 2 | A header |
| 3 | A header |

























