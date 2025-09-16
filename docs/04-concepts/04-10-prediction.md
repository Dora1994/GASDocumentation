# 4.10 Prediction

GAS comes out of the box with support for client-side prediction; however, it does not predict everything. Client-side prediction in GAS means that the client does not have to wait for the server's permission to activate a `GameplayAbility` and apply `GameplayEffects`. It can "predict" the server giving it permission to do this and predict the targets that it would apply `GameplayEffects` to. The server then runs the `GameplayAbility` network latency-time after the client activates and tells the client if he was correct or not in his predictions. If the client was wrong in any of his predictions, he will "roll back" his changes from his "mispredictions" to match the server.

The definitive source for GAS-related prediction is `GameplayPrediction.h` in the plugin source code.

Epic's mindset is to only predict what you "can get away with". For example, Paragon and Fortnite do not predict damage. Most likely they use [ExecutionCalculations](04-5-gameplay-effects.md) for their damage which cannot be predicted anyway. This is not to say that you can't try to predict certain things like damage. By all means if you do it and it works well for you then that's great.

> ... we are also not all in on a "predict everything: seamlessly and automatically" solution. We still feel player prediction is best kept to a minimum (meaning: predict the minimum amount of stuff you can get away with).

*Dave Ratti from Epic's comment from the new [Network Prediction Plugin](#concepts-p-npp)*

**What is predicted:**
> * Ability activation
> *	Triggered Events
> *	GameplayEffect application:
>    * Attribute modification (EXCEPTIONS: Executions do not currently predict, only attribute modifiers)
>    * GameplayTag modification
> * Gameplay Cue events (both from within predictive gameplay effect and on their own)
> * Montages
> * Movement (built into UE's UCharacterMovement)

**What is not predicted:**
> * GameplayEffect removal
> * GameplayEffect periodic effects (dots ticking)

*From `GameplayPrediction.h`*

While we can predict `GameplayEffect` application, we cannot predict `GameplayEffect` removal. One way that we can work around this limitation is to predict the inverse effect when we want to remove a `GameplayEffect`. Say we predict a movement speed slow of 40%. We can predictively remove it by applying a movement speed buff of 40%. Then remove both `GameplayEffects` at the same time. This is not appropriate for every scenario and support for predicting `GameplayEffect` removal is still needed. Dave Ratti from Epic has expressed desire to add it to a [future iteration of GAS](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89).

Because we cannot predict the removal of `GameplayEffects`, we cannot fully predict `GameplayAbility` cooldowns and there is no inverse `GameplayEffect` workaround for them. The server's replicated `Cooldown GE` will exist on the client and any attempts to bypass this (with `Minimal` replication mode for example) will be rejected by the server. This means clients with higher latencies take longer to tell the server to go on cooldown and to receive the removal of the server's `Cooldown GE`. This means players with higher latencies will have a lower rate of fire than players with lower latencies, giving them a disadvantage against lower latency players. Fortnite avoids this issue by using custom bookkeeping instead of `Cooldown GEs`.

Regarding predicting damage, I personally do not recommend it despite it being one of the first things that most people try when starting with GAS. I especially do not recommend trying to predict death. While you can predict damage, doing so is tricky. If you mispredict applying damage, the player will see the enemy's health jump back up. This can be especially awkward and frustrating if you try to predict death. Say you mispredict a `Character's` death and it starts ragdolling only to stop ragdolling and continue shooting at you when the server corrects it.

**[⬆ Back to Top](../README.md#table-of-contents)**
