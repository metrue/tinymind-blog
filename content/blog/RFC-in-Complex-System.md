---
title: RFC in Complex System
date: 2023-06-12T05:35:07.322Z
---

相信工程师们对 RFC 都非常的熟悉,所以本文我不想赘述,RFC 从一个单纯的 *Request for Comments*, 成为了如今技术领域引入改变之前的行为准则, 我们都从 RFC 的撰写和审核中收益很多, 本文我想分享我对在复杂系统中进行 RFC 撰写的一些体会.

### 复杂系统

当我们谈论系统的时候,看我看来,我们实际上在谈论两个东西: 代码和工程师.

```
System = f(Codes, Engineers)
```

* 代码(`Codes`): 我们常常会提到的,组件,API, 服务,配置,数据结构,通信方式,交互模式,设计风格等等,这些在各种不同系统的不同称谓,就是软件系统中的代码.
* 工程师(`Engineers`): 维护这些代码的人.

所以当我们谈论软件系统的时候,我们其实本质上不仅仅是是在讨论代码, 同时也是在讨论它背后的维护者。 所以当我们说复杂系统的时候,我们似乎只是在表达系统中的代码(`Codes`)部分的复杂度,但是在现实的系统中,它的复杂度还包工程师(Engineers)这个纬度的复杂度.

```
System Complexity = O(Codes, Engineers)
```

所以一个系统的复杂度,它是同时关于代码和维护代码的人的复杂度. 当我真正体会到这一点之后,我慢慢学会了一点如何在复杂的系统中如何为自己想引入的改动撰写一个RFC,使得我的 RFC 能够得到更好的 Review.

### RFC

关于 RFC 如果大家感兴趣, 可以在 [Wikipedia](https://en.wikipedia.org/wiki/Request_for_Comments) 上看它的详细历史,它很有趣的介绍是如何从真正的单纯只是 *Request for Comments* 到今天几乎成为了技术领域的一种是行为准则。我自己看来, RFC 实际上是在和大家说: 我将向系统中引入改动,我的设计是这样的,我评估下来之后有这些影响,请大家来一起帮我审查并且提出建议. 所以 RFC 本质上是进行系统变化 (`ΔCodes`) 的一种表达,作为 RFC 作者做的事情是让表达能够清楚明白: 不仅包括变动的具体范围,也包括变动所引起的连锁反应;不仅包括变动的指导原则,也包括使其实现的具体措施;不仅包括变动所带来的收益,同时也表明其携带的风险; 这样才能够使得 RFC 所得到的结果是有价值的.

```
RFC = Δ(Codes)
```

### 复杂系统中的 RFC

而当我们在一个复杂系统中,为我们的即将引入的改动写一份 RFC 的时候,意味着我们的改动可能给系统中的多个组件(`Codes`) 带来影响,它可能需要和多个团队 (`Engineers`) 达成一致. 我目前喜欢的 RFC 风格是简明扼要,清晰准确的表达四个事情 (NFIC: Necessity, Feasibility, Impacts, Collaboration):

* 我为什么要进行这个改动: Necessity

充分的阐述改动的缘由来, 这项改动为什么是必要的, 也就是改动的背景和动机是什么.

* 我将如何进行改动: Feasibility

详细描述我计划如何进行这个改动,以证明它的可行性。具体的设计和实现细节让读者理解我的解决方案, 这将有助于他们更好的评估改动的可行性。

* 这项改动会带来这些影响: Impacts

充分评估即将引入的改动对整个系统所带来的影响,包括正面和负面影响,也包括了对未来发展可能产生的影响。这部分的目标是消除系统维护者的顾虑,提高他们对改动的信心。

* Collaboration

最后, 如果这次的改动需要其他团队来协助, 那么需要描述你希望得到的帮助, 需要的资源,需要的反馈, 如何协作等。

### 最后

RFC 的本质是对系统改动的一种表达,所以你可以选择一种你最合适的方式来让你的表达能够更顺畅, 使讨论更加容易的达成一致.