# 2. Sample Project

## 2. 示例项目

A multiplayer third person shooter sample project is included with this documentation aimed at people new to the GameplayAbilitySystem Plugin but not new to Unreal Engine. Users are expected to know C++, Blueprints, UMG, Replication, and other intermediate topics in UE. This project provides an example of how to set up a basic third person shooter multiplayer-ready project with the `AbilitySystemComponent` (`ASC`) on the `PlayerState` class for player/AI controlled heroes and the `ASC` on the `Character` class for AI controlled minions.

本文档包含一个多人第三人称射击游戏示例项目，面向对GameplayAbilitySystem插件不熟悉但对虚幻引擎不陌生的用户。用户需要了解C++、蓝图、UMG、网络复制以及UE中的其他中级主题。该项目提供了如何设置基本第三人称射击游戏多人就绪项目的示例，其中玩家/AI控制的英雄在`PlayerState`类上使用`AbilitySystemComponent`（`ASC`），AI控制的小兵在`Character`类上使用`ASC`。

The goal is to keep this project simple while showing the GAS basics and demonstrating some commonly requested abilities with well-commented code. Because of its beginner focus, the project does not show advanced topics like [predicting projectiles](../04-concepts/04-10-prediction.md).

目标是保持项目简单，同时展示GAS基础知识并通过注释良好的代码演示一些常用的能力。由于其面向初学者的重点，该项目不展示高级主题，如[预测弹道](../04-concepts/04-10-prediction.md)。

Concepts demonstrated:
* `ASC` on `PlayerState` vs `Character`
* Replicated `Attributes`
* Replicated animation montages
* `GameplayTags`
* Applying and removing `GameplayEffects` inside of and externally from `GameplayAbilities`
* Applying damage mitigated by armor to change health of a character
* `GameplayEffectExecutionCalculations`
* Stun effect
* Death and respawn
* Spawning actors (projectiles) from an ability on the server
* Predictively changing the local player's speed with aim down sights and sprinting
* Constantly draining stamina to sprint
* Using mana to cast abilities
* Passive abilities
* Stacking `GameplayEffects`
* Targeting actors
* `GameplayAbilities` created in Blueprint
* `GameplayAbilities` created in C++
* Instanced per `Actor` `GameplayAbilities`
* Non-Instanced `GameplayAbilities` (Jump)
* Static `GameplayCues` (FireGun projectile impact particle effect)
* Actor `GameplayCues` (Sprint and Stun particle effects)

演示的概念：
* `PlayerState`与`Character`上的`ASC`
* 复制的`属性`
* 复制的动画蒙太奇
* `游戏标签`
* 在`游戏能力`内部和外部应用和移除`游戏效果`
* 应用被护甲减伤的伤害来改变角色生命值
* `游戏效果执行计算`
* 眩晕效果
* 死亡和重生
* 从服务器上的能力生成Actor（弹道）
* 通过瞄准和冲刺预测性地改变本地玩家速度
* 持续消耗体力来冲刺
* 使用法力值来施放能力
* 被动能力
* 堆叠`游戏效果`
* 目标Actor
* 在蓝图中创建的`游戏能力`
* 在C++中创建的`游戏能力`
* 每个`Actor`实例化的`游戏能力`
* 非实例化的`游戏能力`（跳跃）
* 静态`游戏提示`（开火枪弹道撞击粒子效果）
* Actor`游戏提示`（冲刺和眩晕粒子效果）

The hero class has the following abilities:

英雄类具有以下能力：

| Ability                    | Input Bind          | Predicted  | C++ / Blueprint | Description                                                                                                                                                                  |
| -------------------------- | ------------------- | ---------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Jump                       | Space Bar           | Yes        | C++             | Makes the hero jump.                                                                                                                                                         |
| Gun                        | Left Mouse Button   | No         | C++             | Fires a projectile from the hero's gun. The animation is predicted but the projectile is not.                                                                                |
| Aim Down Sights            | Right Mouse Button  | Yes        | Blueprint       | While the button is held, the hero will walk slower and the camera will zoom in to allow more precise shots with the gun.                                                    |
| Sprint                     | Left Shift          | Yes        | Blueprint       | While the button is held, the hero will run faster draining stamina.                                                                                                         |
| Forward Dash               | Q                   | Yes        | Blueprint       | The hero dashes forward at the cost of stamina.                                                                                                                              |
| Passive Armor Stacks       | Passive             | No         | Blueprint       | Every 4 seconds the hero gains a stack of armor up to a maximum of 4 stacks. Receiving damage removes one stack of armor.                                                    |
| Meteor                     | R                   | No         | Blueprint       | Player targets a location to drop a meteor on the enemies causing damage and stunning them. The targeting is predicted while spawning the meteor is not.                     |

| 能力                       | 输入绑定            | 预测       | C++ / 蓝图      | 描述                                                                                                                                                                          |
| -------------------------- | ------------------- | ---------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 跳跃                       | 空格键              | 是         | C++             | 让英雄跳跃。                                                                                                                                                                   |
| 开火                       | 左鼠标按钮          | 否         | C++             | 从英雄的枪发射弹道。动画是预测的，但弹道不是。                                                                                                                                |
| 瞄准                       | 右鼠标按钮          | 是         | 蓝图            | 按住按钮时，英雄会走得更慢，相机会放大以允许更精确的射击。                                                                                                                    |
| 冲刺                       | 左Shift键           | 是         | 蓝图            | 按住按钮时，英雄会跑得更快并消耗体力。                                                                                                                                        |
| 前冲                       | Q键                 | 是         | 蓝图            | 英雄以消耗体力为代价向前冲刺。                                                                                                                                                |
| 被动护甲堆叠               | 被动                | 否         | 蓝图            | 每4秒英雄获得一层护甲，最多4层。受到伤害会移除一层护甲。                                                                                                                       |
| 流星                       | R键                 | 否         | 蓝图            | 玩家瞄准一个位置向敌人投下流星，造成伤害并眩晕他们。瞄准是预测的，但生成流星不是。                                                                                            |

It does not matter if `GameplayAbilities` are created in C++ or Blueprint. A mixture of the two were used here for example of how to do them in each language.

`游戏能力`是在C++还是蓝图中创建并不重要。这里混合使用了两者作为如何在每种语言中实现它们的示例。

Minions do not come with any predefined `GameplayAbilities`. The Red Minions have more health regen while the Blue Minions have higher starting health.

小兵没有预定义的`游戏能力`。红色小兵有更多的生命回复，而蓝色小兵有更高的初始生命值。

For `GameplayAbility` naming, I used the suffix `_BP` to denote the `GameplayAbility's` logic was created in Blueprint. The lack of suffix means the logic was created in C++.

对于`游戏能力`命名，我使用后缀`_BP`来表示`游戏能力`的逻辑是在蓝图中创建的。没有后缀意味着逻辑是在C++中创建的。

**Blueprint Asset Naming Prefixes**

**蓝图资源命名前缀**

| Prefix      | Asset Type          |
| ----------- | ------------------- |
| GA_         | GameplayAbility     |
| GC_         | GameplayCue         |
| GE_         | GameplayEffect      |

| 前缀        | 资源类型            |
| ----------- | ------------------- |
| GA_         | 游戏能力            |
| GC_         | 游戏提示            |
| GE_         | 游戏效果            |

**[⬆ Back to Top](../README.md#table-of-contents)**
