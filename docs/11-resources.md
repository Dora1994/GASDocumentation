# 11. Other Resources

## 11. 其他资源

* [Official Documentation](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)
* Source Code!
   * Especially `GameplayPrediction.h`
* [Lyra Sample Project by Epic](https://unrealengine.com/marketplace/en-US/learn/lyra)
* [Action RPG Sample Project by Epic](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)
* [Unreal Slackers Discord](https://unrealslackers.org/) has a text channel dedicated to GAS `#gameplay-ability-system`
   * Check pinned messages
* [GitHub repository of resources by Dan 'Pan'](https://github.com/Pantong51/GASContent)
* [YouTube Videos by SabreDartStudios](https://www.youtube.com/channel/UCCFUhQ6xQyjXDZ_d6X_H_-A)

* [官方文档](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)
* 源代码！
   * 特别是`GameplayPrediction.h`
* [Epic的Lyra示例项目](https://unrealengine.com/marketplace/en-US/learn/lyra)
* [Epic的动作RPG示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)
* [Unreal Slackers Discord](https://unrealslackers.org/)有一个专门用于GAS的文本频道`#gameplay-ability-system`
   * 查看置顶消息
* [Dan 'Pan'的资源GitHub仓库](https://github.com/Pantong51/GASContent)
* [SabreDartStudios的YouTube视频](https://www.youtube.com/channel/UCCFUhQ6xQyjXDZ_d6X_H_-A)

## 11.1 Q&A With Epic Game's Dave Ratti

## 11.1 与Epic Games的Dave Ratti的问答

### 11.1.1 Community Questions 1

### 11.1.1 社区问题1

[Dave Ratti responses to the Unreal Slackers Discord Server community questions about GAS](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89):

[Dave Ratti对Unreal Slackers Discord服务器关于GAS的社区问题的回答](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)：

1. How can we create scoped prediction windows on demand outside or irrespective of `GameplayAbilities`? For example, how can a fire and forget projectile locally predict a damage `GameplayEffect` when it hits an enemy?

1. 我们如何在不依赖`游戏能力`的情况下按需创建作用域预测窗口？例如，一个发射后不管的弹道如何在击中敌人时本地预测伤害`游戏效果`？

> The PredictionKey system is not really meant to do this. Fundamentally this systems works by a client initiating a predictive action, telling the server about it with a key, and then both client and server running the same thing and associating predictive side effects with the given prediction key. For example, "I am predictively activating an ability" or "I have produced target data and am going to predictively run the part of the ability graph after the WaitTargetData task".
>
> With this pattern, the PredictionKey "bounces" off the server and comes back to the client via UAbilitySystemComponent::ReplicatedPredictionKeyMap (replicated property). Once the key is replicated back from the server, the client is able to undo all of the locally predictive side effects (GameplayCues, GameplayEffects): the replicated versions *will be there* and if they aren't then it was a misprediction. Knowing exactly when to undo the predictive side effects is crucial here: if you are too early you will see gaps, if you are too late you will have "double". (Note this is referring to stateful prediction, like a looping GameplayCue of a duration based Gameplay Effect. "Burst" GameplayCues and instant Gameplay Effects are never "undone" or rolled back. They are just skipped on the client if there is a prediction key associated with them).
>
> To further hit home the point: it's crucial that predictive action is something the server does not do on their own, but only does so when the client tells them to. So having a generic "Create a key on demand and tell the server so I can run something" does not work unless that "something" is something the server will only do once told to by the client.
>
> Backing up to the original question: something like a fire and forget projectile. Both Paragon and Fornite have projectile actor classes that use GameplayCues. However we do not use the Prediction Key system to do these. Instead we have a concept on Non-Replicated GameplayCues. GameplayCues that just fire off locally and are skipped by the server completely. Essentially all these are direct calls to UGameplayCueManager::HandleGameplayCue. They do not route through the UAbilitySystemComponent so no prediction key checks / early returns are made.
>
> The downside with non replicated GameplayCues is that, well, they are not replicated. So its up to the projectile class/blueprint to make sure the code paths that call these functions are running on everyone. We have for cues startup (called in BeginPlay), explosion, hit wall/character, etc.
>
> These type of events are already generated client side, so calling into a non replicated gameplay cue was no big deal. Complicated blueprints can be tricky, and are up to the author to make sure they understand what is running where.

> 预测键系统并不是为了做这个而设计的。从根本上说，这个系统的工作原理是客户端发起一个预测动作，用一个键告诉服务器，然后客户端和服务器都运行相同的东西，并将预测副作用与给定的预测键关联。例如，"我正在预测性地激活一个能力"或"我已经产生了目标数据，并将在WaitTargetData任务之后预测性地运行能力图的那部分"。
>
> 使用这种模式，预测键从服务器"反弹"并通过UAbilitySystemComponent::ReplicatedPredictionKeyMap（复制属性）回到客户端。一旦键从服务器复制回来，客户端就能够撤销所有本地预测副作用（游戏提示、游戏效果）：复制的版本*将在那里*，如果它们不在，那么这就是一个错误预测。确切知道何时撤销预测副作用在这里至关重要：如果你太早，你会看到间隙，如果你太晚，你会有"双重"。（注意这是指有状态的预测，如基于持续时间的游戏效果的循环游戏提示。"爆发"游戏提示和即时游戏效果永远不会被"撤销"或回滚。如果有关联的预测键，它们只是在客户端被跳过）。
>
> 进一步强调这一点：预测动作是服务器不会自己做的事情，只有在客户端告诉它们时才这样做，这是至关重要的。所以有一个通用的"按需创建键并告诉服务器，这样我就可以运行某些东西"是行不通的，除非那个"某些东西"是服务器只有在客户端告诉它时才会做的事情。
>
> 回到原始问题：像发射后不管的弹道这样的东西。Paragon和Fortnite都有使用游戏提示的弹道Actor类。然而，我们不使用预测键系统来做这些。相反，我们有一个非复制游戏提示的概念。游戏提示只是在本地触发，服务器完全跳过。本质上，这些都是对UGameplayCueManager::HandleGameplayCue的直接调用。它们不通过UAbilitySystemComponent路由，所以不进行预测键检查/早期返回。
>
> 非复制游戏提示的缺点是，嗯，它们不被复制。所以弹道类/蓝图要确保调用这些函数的代码路径在每个人身上运行。我们有提示启动（在BeginPlay中调用）、爆炸、击中墙壁/角色等。
>
> 这些类型的事件已经在客户端生成，所以调用非复制的游戏提示不是什么大问题。复杂的蓝图可能很棘手，这取决于作者确保他们理解什么在哪里运行。

**[⬆ Back to Top](../README.md#table-of-contents)**
