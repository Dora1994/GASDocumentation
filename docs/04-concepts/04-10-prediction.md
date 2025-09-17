# 4.10 Prediction （预测）

GAS comes out of the box with support for client-side prediction; however, it does not predict everything. Client-side prediction in GAS means that the client does not have to wait for the server's permission to activate a `GameplayAbility` and apply `GameplayEffects`. It can "predict" the server giving it permission to do this and predict the targets that it would apply `GameplayEffects` to. The server then runs the `GameplayAbility` network latency-time after the client activates and tells the client if he was correct or not in his predictions. If the client was wrong in any of his predictions, he will "roll back" his changes from his "mispredictions" to match the server.

GAS开箱即用支持客户端预测；然而，它并不预测所有内容。GAS中的客户端预测意味着客户端不必等待服务器的许可来激活`GameplayAbility`和应用`GameplayEffects`。它可以"预测"服务器给予它这样做的许可，并预测它将应用`GameplayEffects`的目标。然后服务器在客户端激活后的网络延迟时间运行`GameplayAbility`，并告诉客户端他的预测是否正确。如果客户端的任何预测都是错误的，他将从他的"错误预测"中"回滚"他的更改以匹配服务器。

The definitive source for GAS-related prediction is `GameplayPrediction.h` in the plugin source code.

GAS相关预测的权威来源是插件源代码中的`GameplayPrediction.h`。

Epic's mindset is to only predict what you "can get away with". For example, Paragon and Fortnite do not predict damage. Most likely they use [ExecutionCalculations](04-5-gameplay-effects.md) for their damage which cannot be predicted anyway. This is not to say that you can't try to predict certain things like damage. By all means if you do it and it works well for you then that's great.

Epic的思维方式是只预测你"能够侥幸通过"的内容。例如，Paragon和Fortnite不预测伤害。他们很可能使用[ExecutionCalculations](04-5-gameplay-effects.md)来计算伤害，这无论如何都无法预测。这并不是说你不能尝试预测某些东西，比如伤害。如果你这样做并且对你效果很好，那就太好了。

> ... we are also not all in on a "predict everything: seamlessly and automatically" solution. We still feel player prediction is best kept to a minimum (meaning: predict the minimum amount of stuff you can get away with).

> ...我们也不完全支持"预测一切：无缝和自动"的解决方案。我们仍然认为玩家预测最好保持在最低限度（意思是：预测你能够侥幸通过的最少量的东西）。

*Dave Ratti from Epic's comment from the new [Network Prediction Plugin](#concepts-p-npp)*

*Dave Ratti来自Epic关于新[Network Prediction Plugin](#concepts-p-npp)的评论*

**What is predicted: （可以预测的内容）**
> * Ability activation (能力激活)
> *	Triggered Events (触发事件)
> *	GameplayEffect application (GameplayEffect应用):
>    * Attribute modification (EXCEPTIONS: Executions do not currently predict, only attribute modifiers) （属性修改（例外：Executions目前不预测，只有属性修饰符））
>    * GameplayTag modification (GameplayTag修改)
> * Gameplay Cue events (both from within predictive gameplay effect and on their own) （Gameplay Cue事件（来自预测性游戏效果内部和独立的））
> * Montages (蒙太奇)
> * Movement (built into UE's UCharacterMovement) （移动（内置于UE的UCharacterMovement中））

**What is not predicted: （不能预测的内容）**
> * GameplayEffect removal (GameplayEffect移除)
> * GameplayEffect periodic effects (dots ticking) （GameplayEffect周期性效果（持续伤害））

*From `GameplayPrediction.h` （来自`GameplayPrediction.h`）*

While we can predict `GameplayEffect` application, we cannot predict `GameplayEffect` removal. One way that we can work around this limitation is to predict the inverse effect when we want to remove a `GameplayEffect`. Say we predict a movement speed slow of 40%. We can predictively remove it by applying a movement speed buff of 40%. Then remove both `GameplayEffects` at the same time. This is not appropriate for every scenario and support for predicting `GameplayEffect` removal is still needed. Dave Ratti from Epic has expressed desire to add it to a [future iteration of GAS](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89).

虽然我们可以预测`GameplayEffect`的应用，但我们无法预测`GameplayEffect`的移除。当我们想要移除`GameplayEffect`时，我们可以通过预测反向效果来解决这个限制。比如我们预测40%的移动速度减慢。我们可以通过应用40%的移动速度增益来预测性地移除它。然后同时移除两个`GameplayEffects`。这并不适用于每种场景，仍然需要对预测`GameplayEffect`移除的支持。Epic的Dave Ratti已经表达了将其添加到[GAS的未来版本](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)的愿望。

Because we cannot predict the removal of `GameplayEffects`, we cannot fully predict `GameplayAbility` cooldowns and there is no inverse `GameplayEffect` workaround for them. The server's replicated `Cooldown GE` will exist on the client and any attempts to bypass this (with `Minimal` replication mode for example) will be rejected by the server. This means clients with higher latencies take longer to tell the server to go on cooldown and to receive the removal of the server's `Cooldown GE`. This means players with higher latencies will have a lower rate of fire than players with lower latencies, giving them a disadvantage against lower latency players. Fortnite avoids this issue by using custom bookkeeping instead of `Cooldown GEs`.

因为我们无法预测`GameplayEffects`的移除，我们无法完全预测`GameplayAbility`冷却时间，对它们也没有反向`GameplayEffect`的解决方法。服务器复制的`Cooldown GE`将存在于客户端上，任何绕过这个的尝试（例如使用`Minimal`复制模式）都会被服务器拒绝。这意味着延迟较高的客户端需要更长时间告诉服务器进入冷却并接收服务器`Cooldown GE`的移除。这意味着延迟较高的玩家将比延迟较低的玩家有更低的射速，在与低延迟玩家对抗时处于劣势。Fortnite通过使用自定义记录而不是`Cooldown GEs`来避免这个问题。

Regarding predicting damage, I personally do not recommend it despite it being one of the first things that most people try when starting with GAS. I especially do not recommend trying to predict death. While you can predict damage, doing so is tricky. If you mispredict applying damage, the player will see the enemy's health jump back up. This can be especially awkward and frustrating if you try to predict death. Say you mispredict a `Character's` death and it starts ragdolling only to stop ragdolling and continue shooting at you when the server corrects it.

关于预测伤害，尽管这是大多数人开始使用GAS时首先尝试的事情之一，但我个人不建议这样做。我特别不建议尝试预测死亡。虽然你可以预测伤害，但这样做很棘手。如果你错误预测了应用伤害，玩家会看到敌人的生命值跳回去。如果你尝试预测死亡，这可能会特别尴尬和令人沮丧。比如你错误预测了`Character`的死亡，它开始布娃娃化，但当服务器纠正时又停止布娃娃化并继续向你射击。

**[⬆ Back to Top](../README.md#table-of-contents)**
