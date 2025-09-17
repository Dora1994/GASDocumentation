# 1. Intro to the GameplayAbilitySystem Plugin （GameplayAbilitySystem插件介绍）

From the [Official Documentation](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html):
>The Gameplay Ability System is a highly-flexible framework for building abilities and attributes of the type you might find in an RPG or MOBA title. You can build actions or passive abilities for the characters in your games to use, status effects that can build up or wear down various attributes as a result of these actions, implement "cooldown" timers or resource costs to regulate the usage of these actions, change the level of the ability and its effects at each level, activate particle or sound effects, and more. Put simply, this system can help you to design, implement, and efficiently network in-game abilities as simple as jumping or as complex as your favorite character's ability set in any modern RPG or MOBA title.

>游戏能力系统是一个高度灵活的框架，用于构建RPG或MOBA类型游戏中常见的能力和属性。你可以为游戏中的角色构建主动或被动能力，构建这些动作导致的可以增强或削弱各种属性的状态效果，实现"冷却"计时器或资源消耗来调节这些动作的使用，改变能力的等级及其在每个等级的效果，激活粒子或音效等等。简单来说，这个系统可以帮助你设计、实现并高效地网络化游戏内能力，从简单的跳跃到任何现代RPG或MOBA游戏中你最喜欢的角色能力集那样复杂。

The GameplayAbilitySystem plugin is developed by Epic Games and comes with Unreal Engine. It has been battle tested in AAA commercial games such as Paragon and Fortnite among others.

GameplayAbilitySystem插件由Epic Games开发，随虚幻引擎一起提供。它已经在Paragon和Fortnite等AAA商业游戏中经过实战测试。

The plugin provides an out-of-the-box solution in single and multiplayer games for （为单人和多人游戏提供开箱即用的解决方案）:
* Implementing level-based character abilities or skills with optional costs and cooldowns ([GameplayAbilities](../04-concepts/04-6-gameplay-abilities.md))  （实现基于等级的角色能力或技能，具有可选的消耗和冷却时间）
* Manipulating numerical `Attributes` belonging to actors ([Attributes](../04-concepts/04-3-attributes.md))  （操作属于Actor的数值`属性`）
* Applying status effects to actors ([GameplayEffects](../04-concepts/04-5-gameplay-effects.md))  （对Actor应用状态效果）
* Applying `GameplayTags` to actors ([GameplayTags](../04-concepts/04-2-gameplay-tags.md))  （对Actor应用`游戏标签`）
* Spawning visual or sound effects ([GameplayCues](../04-concepts/04-8-gameplay-cues.md))  （生成视觉或音效）
* Replication of everything mentioned above （复制上述所有内容）

In multiplayer games, GAS provides support for [client-side prediction](../04-concepts/04-10-prediction.md) of (在多人游戏中，GAS提供对[客户端预测](../04-concepts/04-10-prediction.md)的支持):
* Ability activation (能力激活)
* Playing animation montages (播放动画蒙太奇)
* Changes to `Attributes` (属性变化)
* Applying `GameplayTags` (应用`游戏标签`)
* Spawning `GameplayCues` (生成`游戏提示`)
* Movement via `RootMotionSource` functions connected to the `CharacterMovementComponent`. (通过连接到`角色移动组件`的`根运动源`函数进行移动)

**GAS must be set up in C++**, but `GameplayAbilities` and `GameplayEffects` can be created in Blueprint by the designers.

**GAS必须在C++中设置**，但`游戏能力`和`游戏效果`可以由设计师在蓝图中创建。

Current issues with GAS:
* `GameplayEffect` latency reconciliation (can't predict ability cooldowns resulting in players with higher latencies having lower rate of fire for low cooldown abilities compared to players with lower latencies).
* Cannot predict the removal of `GameplayEffects`. We can however predict adding `GameplayEffects` with the inverse effects, effectively removing them. This is not always appropriate or feasible and still remains an issue.
* Lack of boilerplate templates, multiplayer examples, and documentation. Hopefully this helps with that!

GAS的当前问题：
* `游戏效果`延迟协调（无法预测能力冷却时间，导致延迟较高的玩家在低冷却能力上比延迟较低的玩家射速更低）
* 无法预测`游戏效果`的移除。然而，我们可以通过预测添加具有反向效果的`游戏效果`来有效移除它们。这并不总是合适或可行的，仍然是一个问题
* 缺乏样板模板、多人游戏示例和文档。希望这能有所帮助！

**[⬆ Back to Top](../README.md#table-of-contents)**
