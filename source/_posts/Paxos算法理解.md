---
title: Paxos算法理解
date: 2017-04-03 15:36:18
tags: Paxos
---

## 一致性算法
> Paxos算法（Basic Paxos）用来解决分布式环境中多台机器对某个提案/值达成一致的问题。  

Paxos算法中的三个角色：proposer、acceptor、learner。一台机器可以兼任三种角色。  
注意accept和chosen的区别：  
- accept：即接受，一个acceptor可以接受任何proposal。
- chosen：即批准，只有被大多数acceptor所accept的proposal才算被批准。

### 1.1 问题描述
假设一组进程都可以提出值进行决议，一致性算法保证只有其中一个value被最终选择。如果没有value被proposed，那么就不会选择任何value。如果某个value被chosen了，所有的进程都可以learn到这个value。由此可以得出一致性的安全性（safety）需求如下：  

* 一个value只有被proposed之后才有可能被chosen，
* 再一次Paxos算法运行实例中，只有一个value最终会被chosen，并且
* learner只能获取到被chosen的value。

除了safety，还需要保证progress，后面将会讨论。  
算法的作者Lamport在论文中不断地加强上述三个约束（关键是第二个）最终提出了Paxos算法。  

### 1.2 value的决策
决策一个值最简单的方式就是只设置一个acceptor，这样一个proposer发送proposal给这个acceptor，然后acceptor总是会选择它所接收到的第一个value（proposal中会指定value）。这是最simple的方法，但是它不能满足要求，因为acceptor存在单点故障。  
因此，更进一步，我们设置多个acceptor，一个proposer提出一个value给acceptor，一个acceptor可能会接受这个value。只有当大多数acceptor都接受这个value时才算它被chosen，大多数的意思通常是超过半数的参与者，因为任何。两个大多数集合一定至少有一个公共的参与者，这样当一个acceptor最多只能接受一个value时保证大多数能够达成一致。  
这种方式可以在只有一个proposer且该proposer只提出一个value请求决策的前提下能够正常工作。  
这种情况下，一致性算法要想能够工作，暗含如下约束：  

> P1. 一个acceptor必须接受它收到的第一个value。

这个约束显然是有问题的，因为多个proposer可能在相同时刻提出多个不同的value，这会导致虽然每个acceptor都接受了一个value，但是并没有某个value被大多数acceptor所接受。例如假设有两个value分别被一半的acceptor接受，那么最终无法达成一致。  
P1的约束暗示一个acceptor必须能够接受多个proposal。我们利用不同的编号来跟踪不同的proposal，这样每个proposal包含一个唯一的编号和一个值。这样一个value被chosen也就表示其所属的proposal被大多数acceptor所接受。  
我们允许有多个proposal被依次chosen的情况发生（后面的proposal编号更高），但是必须保证这多个proposal都包含相同的value，这样需要如下约束：  

> P2. 如果一个带有值v的proposal被chosen，那么每个被chosen的具备更高编号的proposal都具有值v。

proposal的编号都是有序的，后面的会比前面的编号更高。   
P2可以保证safety约束的第二条：只有一个value会被chosen。  
要想被chosen，一个proposal必须至少被一个acceptor接受，因此，我们可以用P2a来满足P2：  

> P2a. 如果一个带有值v的proposal被chosen了，那么后续每个acceptor所accept（注意是accept，而不是chosen，单个acceptor可以accept任何proposal，被大多数acceptor所accept得proposal才叫chosen）的每个具备更高编号的proposal都必须具备值v。

我们依然需要满足P1来保证某个proposal被chosen。但是由于通信是异步的，P2a可能与P1发生冲突，考虑这种情况：一个值为v的proposal被chosen了，说明它被大多数acceptor所accept，但是依然存在少部分acceptor没有收到这个proposal，此时如果一个新的proposer苏醒并提出一个更高编号的值不是v的proposal发给这些少部分acceptor，那么根据P1的约束，这些acceptor一定会accept这个新的proposal，这就违背了P2a约束。  

要想保持P1和P2a，就需要进一步加强P2a：  
> p2b. 如果一个值为v的proposal被chosen了，后续被任何proposer提出的更高编号的proposal的值都必须是v。

由于一个proposal一定是先被某个proposer提出，然后才能被acceptor接受，最后才能被批准。P2b比P2a时序更早，从accept阶段提前到了issue阶段，因此满足P2b一定能满足P2a，从而满足P2约束。  
如何满足P2b？我们进一步提出P2c：  
> P2c. 对于任意的值v和编号n，如果一个带有值v编号为n的proposal被提出，那么一定有一个多数参与者集合S满足：（a）S中不存在acceptor已经accept任何编号小于n的proposal，或者（b）S中的acceptor所接受的proposal中编号最高的具备值v。

我们可以利用数学归纳法证明P2c蕴含P2b：  
假设具备值v编号为m的proposal已经被chosen，当n=m+1时，利用反证法，如果proposal n的值不是v，而是w，根据P2c，一定存在一个多数派S满足：（1）S中的所有acceptor都没有accept过任何编号小于n的proposal，这一点显然不满足，因为S与接收过m提案的集合C一定至少有一个公共成员，显然该成员accept了提案m；（2）S中的acceptor所接受的提案中编号最高的具备值w，这一条也不满足，因为m的值是v。所以假设不成立，当n=m+1时，提案n的值与提案m相同，为v。  
推广到更大的n，对于提案m+1…n-1，根据如上假设，它们都具备值v，假设提案n的值为x，不等于v，那么根据P2c约束，一定存在一个多数派S2，满足：（1）要么它们没有接受过任何编号小于n的任何提案，这一点显然也不满足，因为提案m…n-1已经被chosen了；（2）要么它们accept的提案中编号最大的具备值x，而如果v不等于x，显然也不满足。因此，提案n的值一定是v。  
得证。  
因此P2c蕴含P2b。  

P2c这个约束可以通过消息传递程序进行实现。  
根据P2c的约束内容可以得出，proposer在正式提出一个proposal之前，一定要先和大多数acceptor进行通信，得到它们之前投票的情况，根据回复的信息来设置proposal的value，然后开始投票过程，当得到大多数acceptor接受之后，提案获得通过，然后再通知learner即可。  
由于现实环境中，多条消息可能并发到达某个acceptor，因此可能存在这种情况：某个acceptor还没有accept过任何proposal，此时它回复了编号为n的proposal自己的情况（prepare阶段），但是在对n进行表决（accept阶段）之前，它收到了编号更小为m的另一个提案的accept请求，由于此时它还没有accept提案n，因此它接受了m提案，如果提案m和n的值不一样，那么这就违背了P2c约束，因为时间原因它没有回复给n它所accept的提案m的value。  
预防这个问题发生的方法是：acceptor给proposer第一次应答的同时保证自己不会accept编号更小的提案。这也是对P1约束的加强：  
> P1a. 一个acceptor可以accept编号为n的proposal当且仅当它没有回复过编号大于n的proposal的prepare请求。

至此，P1a和P2c约束已经足够完备，是时候提出完整的Paxos算法了。


### 1.3 Paxos算法内容
Paxos算法分为两个阶段：prepare阶段和accept阶段。

#### 1. prepare阶段：
a. proposer选择一个提案编号n，并利用它构造一个prepare请求发给一个包含大多数acceptor的集合。  
b. 如果一个acceptor编号为n的prepare请求，且n大于它已经回应过的任何一个prepare请求的编号，那么它将把它所accept过的最大编号的提案作为回复，同时承诺不再accept任何编号小于n的proposal。

#### 2. accept阶段：
a. 如果proposer收到了大多数acceptor对其发出的prepare请求的回复，那么它将发送提案n的accept请求给这些acceptor，该请求中包含的值v要么是回复中的此前编号最高proposal的值，要么是该proposer自定义的值（当前者不存在时）。  
b. acceptor将接受提案n的accept请求，除非它已经回复了一个编号更高的proposal的prepare请求。  

### 1.4 算法的Progress保证
存在这样一种情况：两个proposer不断提出编号比对方高的提案，这样会进入活锁状态，永远不会有提案获得批准。算法也就不能向前推进。
为了解决这个问题，确保算法能够progress，必须选出一个主proposer，且只有它能够提出提案，这样就能保证算法的正常执行。  
如果分布式系统能够正常运行，那么主proposer的方案使得系统的liveness（响应、活跃度）也能得到保证。  

## 2 算法扩展
基本的Paxos算法称为Basic Paxos，它允许多个proposer同时提出提案。本算法的一个变种是Mutility Paxos，它通过利用单个主proposer同时向多个Paxos实例发出prepare请求，在后续的决议过程中直接跳过prepare阶段，直接执行accept阶段，提高算法的运行效率。之所以可以跳过prepare阶段，是因为采用了单一proposer，且只有该proposer负责提出议案，议案编号也由它生成，可以保证不存在prepare阶段中可能存在的冲突，不再需要通过prepare阶段来检测冲突。















