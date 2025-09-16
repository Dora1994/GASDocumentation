# 8. Quality of Life Suggestions

## 8. 生活质量建议

## 8.1 Gameplay Effect Containers

## 8.1 游戏效果容器

[GameplayEffectContainers](04-concepts/04-5-gameplay-effects.md) combine [GameplayEffectSpecs](04-concepts/04-5-gameplay-effects.md), [TargetData](04-concepts/04-11-targeting.md), [simple targeting](04-concepts/04-11-targeting.md), and related functionality into easy to use structures. These are great for transfering `GameplayEffectSpecs` to projectiles spawned from an ability that will then apply them on collision at a later time.

[游戏效果容器](04-concepts/04-5-gameplay-effects.md)将[游戏效果规格](04-concepts/04-5-gameplay-effects.md)、[目标数据](04-concepts/04-11-targeting.md)、[简单目标选择](04-concepts/04-11-targeting.md)和相关功能组合成易于使用的结构。这些非常适合将`游戏效果规格`传输到从能力生成的弹道，然后稍后在碰撞时应用它们。

## 8.2 Blueprint AsyncTasks to Bind to ASC Delegates

## 8.2 绑定到ASC委托的蓝图异步任务

To increase designer-friendly iteration times, especially when designing UMG Widgets for UI, create Blueprint AsyncTasks (in C++) to bind to the common change delegates on the `ASC` directly from your UMG Blueprint graphs. The only caveat is that they must be manually destroyed (like when the widget is destroyed) otherwise they will live in memory forever. The Sample Project includes three Blueprint AsyncTasks.

为了增加设计师友好的迭代时间，特别是在为UI设计UMG Widget时，创建蓝图异步任务（在C++中）以直接从UMG蓝图图中绑定到`ASC`上的常见变化委托。唯一的注意事项是它们必须手动销毁（如widget被销毁时），否则它们将永远存在于内存中。示例项目包含三个蓝图异步任务。

Listen for `Attribute` changes:

监听`属性`变化：

![Listen for Attributes Changes BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/attributeschange.png)

Listen for cooldown changes:

监听冷却变化：

![Listen for Cooldown Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

Listen for `GE` stack changes:

监听`GE`堆叠变化：

![Listen for GameplayEffect Stack Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ Back to Top](../README.md#table-of-contents)**
