# 4.6 Gameplay Abilities （游戏能力）

## 4.6.1 Gameplay Ability Definition （游戏能力定义）

[`GameplayAbilities`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/UGameplayAbility/index.html) (`GA`) are any actions or skills that an `Actor` can do in the game. More than one `GameplayAbility` can be active at one time for example sprinting and shooting a gun. These can be made in Blueprint or C++.

[`GameplayAbilities`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/UGameplayAbility/index.html) (`GA`) 是`Actor`在游戏中可以执行的任何动作或技能。可以同时激活多个`GameplayAbility`，例如冲刺和开枪。这些可以在蓝图或C++中制作。

Examples of `GameplayAbilities`:
* Jumping (跳跃)
* Sprinting (冲刺)
* Shooting a gun (开枪)
* Passively blocking an attack every X number of seconds (每X秒被动阻挡一次攻击)
* Using a potion (使用药水)
* Opening a door (开门)
* Collecting a resource (收集资源)
* Constructing a building (建造建筑)

Things that should not be implemented with `GameplayAbilities`:
* Basic movement input (基本移动输入)
* Some interactions with UIs - Don't use a `GameplayAbility` to purchase an item from a store. (某些与UI的交互 - 不要使用`GameplayAbility`从商店购买物品。)

These are not rules, just my recommendations. Your design and implementations may vary.

这些不是规则，只是我的建议。你的设计和实现可能会有所不同。

`GameplayAbilities` come with default functionality to have a level to modify the amount of change to attributes or to change the `GameplayAbility's` functionality.

`GameplayAbilities`具有默认功能，可以有一个等级来修改属性变化的数量或改变`GameplayAbility`的功能。

`GameplayAbilities` run on the owning client and/or the server depending on the [Net Execution Policy](#concepts-ga-net) but not simulated proxies. The `Net Execution Policy` determines if a `GameplayAbility` will be locally [predicted](04-10-prediction.md). They include default behavior for [optional cost and cooldown `GameplayEffects`](#concepts-ga-commit). `GameplayAbilities` use [AbilityTasks](04-7-ability-tasks.md) for actions that happen over time like waiting for an event, waiting for an attribute change, waiting for players to choose a target, or moving a `Character` with `Root Motion Source`. **Simulated clients will not run `GameplayAbilities`**. Instead, when the server runs the ability, anything that visually needs to play on the simulated proxies (like animation montages) will be replicated or RPC'd through `AbilityTasks` or [GameplayCues](04-8-gameplay-cues.md) for cosmetic things like sounds and particles.

`GameplayAbilities`根据[Net Execution Policy](#concepts-ga-net)在拥有客户端和/或服务器上运行，但不在模拟代理上运行。`Net Execution Policy`决定`GameplayAbility`是否会被本地[predicted](04-10-prediction.md)。它们包括[optional cost and cooldown`GameplayEffects`](#concepts-ga-commit)的默认行为。`GameplayAbilities`使用[AbilityTasks](04-7-ability-tasks.md)来处理随时间发生的动作，如等待事件、等待属性变化、等待玩家选择目标，或使用`Root Motion Source`移动`Character`。**模拟客户端不会运行`GameplayAbilities`**。相反，当服务器运行能力时，任何需要在模拟代理上视觉播放的内容（如动画蒙太奇）都会通过`AbilityTasks`或[GameplayCues](04-8-gameplay-cues.md)复制或RPC，用于声音和粒子等装饰性内容。

All `GameplayAbilities` will have their `ActivateAbility()` function overriden with your gameplay logic. Additional logic can be added to `EndAbility()` that runs when the `GameplayAbility` completes or is canceled.

所有`GameplayAbilities`都会用你的游戏逻辑重写其`ActivateAbility()`函数。可以向`EndAbility()`添加额外逻辑，在`GameplayAbility`完成或取消时运行。

Flowchart of a simple `GameplayAbility`:

简单`GameplayAbility`的流程图：
![Simple GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartsimple.png)

Flowchart of a more complex `GameplayAbility`:

更复杂`GameplayAbility`的流程图：
![Complex GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartcomplex.png)

Complex abilities can be implemented using multiple `GameplayAbilities` that interact (activate, cancel, etc) with each other.

复杂能力可以通过使用多个相互交互（激活、取消等）的`GameplayAbilities`来实现。

### 4.6.1.1 Replication Policy （复制策略）

Don't use this option. The name is misleading and you don't need it. [GameplayAbilitySpecs](#concepts-ga-spec) are replicated from the server to the owning client by default. As mentioned above, **`GameplayAbilities` don't run on simulated proxies**. They use `AbilityTasks` and `GameplayCues` to replicate or RPC visual changes to the simulated proxies. Dave Ratti from Epic has stated his desire to [remove this option in the future](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89).

不要使用这个选项。名称具有误导性，你不需要它。[GameplayAbilitySpecs](#concepts-ga-spec)默认从服务器复制到拥有客户端。如上所述，**`GameplayAbilities`不在模拟代理上运行**。它们使用`AbilityTasks`和`GameplayCues`来复制或RPC视觉变化到模拟代理。Epic的Dave Ratti已经表达了他[在未来移除这个选项](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)的愿望。

### 4.6.1.2 Server Respects Remote Ability Cancellation （服务器尊重远程能力取消）

This option causes trouble more often than not. It means if the client's `GameplayAbility` ends either due to cancellation or natural completion, it will force the server's version to end whether it completed or not. The latter issue is the important one, especially for locally predicted `GameplayAbilities` used by players with high latencies. Generally you will want to disable this option.

这个选项经常引起麻烦。这意味着如果客户端的`GameplayAbility`由于取消或自然完成而结束，它会强制服务器版本结束，无论是否完成。后一个问题是重要的，特别是对于高延迟玩家使用的本地预测`GameplayAbilities`。通常你会想要禁用这个选项。

### 4.6.1.3 Replicate Input Directly （直接复制输入）

Setting this option will always replicate input press and release events to the server. Epic recommends not using this and instead relying on the `Generic Replicated Events` that are built into the existing input related [AbilityTasks](04-7-ability-tasks.md) if you have your [input bound to your `ASC`](#concepts-ga-input).

设置这个选项会始终将输入按下和释放事件复制到服务器。Epic建议不使用这个，而是依赖内置在现有输入相关[AbilityTasks](04-7-ability-tasks.md)中的`通用复制事件`，如果你有[输入绑定到你的`ASC`](#concepts-ga-input)。

Epic's comment:

Epic的注释：
```c++
/** Direct Input state replication. These will be called if bReplicateInputDirectly is true on the ability and is generally not a good thing to use. (Instead, prefer to use Generic Replicated Events). */
```

**[⬆ Back to Top](../README.md#table-of-contents)**
