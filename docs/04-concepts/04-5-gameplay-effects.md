# 4.5 Gameplay Effects （游戏效果）

## 4.5.1 Gameplay Effect Definition （游戏效果定义）

[`GameplayEffects`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html) (`GE`) are the vessels through which abilities change [Attributes](04-3-attributes.md) and [GameplayTags](04-2-gameplay-tags.md) on themselves and others. They can cause immediate `Attribute` changes like damage or healing or apply long term status buff/debuffs like a movespeed boost or stunning. The `UGameplayEffect` class is a meant to be a **data-only** class that defines a single gameplay effect. No additional logic should be added to `GameplayEffects`. Typically designers will create many Blueprint child classes of `UGameplayEffect`.

[`GameplayEffects`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html) (`GE`) 是能力改变自身和其他对象的[Attributes](04-3-attributes.md)和[GameplayTags](04-2-gameplay-tags.md)的载体。它们可以造成立即的`Attributes`变化，如伤害或治疗，或应用长期状态增益/减益，如移动速度提升或眩晕。`UGameplayEffect`类是一个**仅数据**类，定义单个游戏效果。不应该向`GameplayEffects`添加额外的逻辑。通常设计师会创建许多`UGameplayEffect`的蓝图子类。

`GameplayEffects` change `Attributes` through [Modifiers](#concepts-ge-mods) and [Executions (`GameplayEffectExecutionCalculation`)](#concepts-ge-ec).

`GameplayEffects`通过[Modifiers](#concepts-ge-mods)和[Executions (`GameplayEffectExecutionCalculation`)](#concepts-ge-ec)来改变`Attributes`。

`GameplayEffects` have three types of duration: `Instant`, `Duration`, and `Infinite`.

`GameplayEffects`有三种持续时间类型：`Instant`（即时）、`Duration`（持续时间）和`Infinite`（无限）。

Additionally, `GameplayEffects` can add/execute [GameplayCues](04-8-gameplay-cues.md). An `Instant` `GameplayEffect` will call `Execute` on the `GameplayCue` `GameplayTags` whereas a `Duration` or `Infinite` `GameplayEffect` will call `Add` and `Remove` on the `GameplayCue` `GameplayTags`.

此外，`GameplayEffects`可以添加/执行[GameplayCues](04-8-gameplay-cues.md)。`Instant` `GameplayEffect`会在`GameplayCue` `GameplayTags`上调用`Execute`，而`Duration`或`Infinite` `GameplayEffect`会在`GameplayCue` `GameplayTags`上调用`Add`和`Remove`。

| Duration Type （持续时间类型） | GameplayCue Event （游戏提示事件） | When to use （何时使用）                                                                                                                                                                                                                                |
| ------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Instant` （即时）     | Execute   （执行）        | For immediate permanent changes to `Attribute's` `BaseValue`. `GameplayTags` will not be applied, not even for a frame. （用于对`Attribute`的`BaseValue`进行立即永久性更改。`GameplayTags`不会被应用，甚至不会持续一帧。）                                                                                                                   |
| `Duration` （持续时间）    | Add & Remove      (添加和移除）        | For temporary changes to `Attribute's` `CurrentValue` and to apply `GameplayTags` that will be removed when the `GameplayEffect` expires or is manually removed. The duration is specified in the `UGameplayEffect` class/Blueprint. （用于对`Attribute`的`CurrentValue`进行临时更改，并应用在`GameplayEffect`过期或手动移除时将被移除的`GameplayTags`。持续时间在`UGameplayEffect`类/蓝图中指定。）      |
| `Infinite` （无限）    | Add & Remove      (添加和移除）        | For temporary changes to `Attribute's` `CurrentValue` and to apply `GameplayTags` that will be removed when the `GameplayEffect` is removed. These will never expire on their own and must be manually removed by an ability or the `ASC`. （用于对`Attribute`的`CurrentValue`进行临时更改，并应用在`GameplayEffect`被移除时将被移除的`GameplayTags`。这些永远不会自行过期，必须由能力或`ASC`手动移除。） |

`Duration` and `Infinite` `GameplayEffects` have the option of applying `Periodic Effects` that apply its `Modifiers` and `Executions` every `X` seconds as defined by its `Period`. `Periodic Effects` are treated as `Instant` `GameplayEffects` when it comes to changing the `Attribute's` `BaseValue` and `Executing` `GameplayCues`. These are useful for damage over time (DOT) type effects. **Note:** `Periodic Effects` cannot be [predicted](04-10-prediction.md).

`Duration`和`Infinite` `GameplayEffects`可以选择应用`周期性效果`，根据其`Period`定义每`X`秒应用其`Modifiers`和`Executions`。在改变`Attribute`的`BaseValue`和`Executing` `GameplayCues`方面，`Periodic Effects`被视为`Instant` `GameplayEffects`。这些对于持续伤害(DOT)类型效果很有用。**注意：**`Periodic Effects`无法被[predicted](04-10-prediction.md)。

`Duration` and `Infinite` `GameplayEffects` can be temporarily turned off and on after application if their `Ongoing Tag Requirements` are not met/met ([Gameplay Effect Tags](#concepts-ge-tags)). Turning off a `GameplayEffect` removes the effects of its `Modifiers` and applied `GameplayTags` but does not remove the `GameplayEffect`. Turning the `GameplayEffect` back on reapplies its `Modifiers` and `GameplayTags`.

如果`Ongoing Tag Requirements`未满足/满足（[Gameplay Effect Tags](#concepts-ge-tags)），`Duration`和`Infinite` `GameplayEffects`可以在应用后临时关闭和开启。关闭`GameplayEffect`会移除其`Modifiers`和应用的`GameplayTags`的效果，但不会移除`GameplayEffect`。重新开启`GameplayEffect`会重新应用其`Modifiers`和`GameplayTags`。

If you need to manually recalculate the `Modifiers` of a `Duration` or `Infinite` `GameplayEffect` (say you have an `MMC` that uses data that doesn't come from `Attributes`), you can call `UAbilitySystemComponent::ActiveGameplayEffects.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)` with the same level that it already has using `UAbilitySystemComponent::ActiveGameplayEffects.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()`. `Modifiers` that are based on backing `Attributes` automatically update when those backing `Attributes` update. The key functions of `SetActiveGameplayEffectLevel()` to update the `Modifiers` are:

如果你需要手动重新计算`Duration`或`Infinite` `GameplayEffect`的`Modifiers`（比如你有一个使用不来自`Attributes`的数据的`MMC`），你可以使用`UAbilitySystemComponent::ActiveGameplayEffects.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()`获取已有的等级，然后用相同等级调用`UAbilitySystemComponent::ActiveGameplayEffects.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)`。基于支持`Attributes`的`Modifiers`会在这些支持`Attributes`更新时自动更新。`SetActiveGameplayEffectLevel()`更新`Modifiers`的关键函数是：

```C++
MarkItemDirty(Effect);
Effect.Spec.CalculateModifierMagnitudes();
// Private function otherwise we'd call these three functions without needing to set the level to what it already is
UpdateAllAggregatorModMagnitudes(Effect);
```

`GameplayEffects` are not typically instantiated. When an ability or `ASC` wants to apply a `GameplayEffect`, it creates a [GameplayEffectSpec](#concepts-ge-spec) from the `GameplayEffect's` `ClassDefaultObject`. Successfully applied `GameplayEffectSpecs` are then added to a new struct called `FActiveGameplayEffect` which is what the `ASC` keeps track of in a special container struct called `ActiveGameplayEffects`.

`GameplayEffects`通常不会被实例化。当能力或`ASC`想要应用`GameplayEffect`时，它会从`GameplayEffect`的`ClassDefaultObject`创建一个[GameplayEffectSpec](#concepts-ge-spec)。成功应用的`GameplayEffectSpecs`然后被添加到一个名为`FActiveGameplayEffect`的新结构中，这是`ASC`在名为`ActiveGameplayEffects`的特殊容器结构中跟踪的内容。

## 4.5.2 Applying Gameplay Effects （应用游戏效果）

`GameplayEffects` can be applied in many ways from functions on [GameplayAbilities](04-6-gameplay-abilities.md) and functions on the `ASC` and usually take the form of `ApplyGameplayEffectTo`. The different functions are essentially convenience functions that will eventually call `UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf()` on the `Target`.

`GameplayEffects`可以通过[GameplayAbilities](04-6-gameplay-abilities.md)上的函数和`ASC`上的函数以多种方式应用，通常采用`ApplyGameplayEffectTo`的形式。不同的函数本质上是便利函数，最终会在`Target`上调用`UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf()`。

To apply `GameplayEffects` outside of a `GameplayAbility` for example from a projectile, you need to get the `Target's` `ASC` and use one of its functions to `ApplyGameplayEffectToSelf`.

要在`GameplayAbility`外部应用`GameplayEffects`（例如从弹道），你需要获取`Target`的`ASC`并使用其中一个函数来`ApplyGameplayEffectToSelf`。

You can listen for when any `Duration` or `Infinite` `GameplayEffects` are applied to an `ASC` by binding to its delegate:

你可以通过绑定到委托来监听任何`Duration`或`Infinite` `GameplayEffects`何时应用到`ASC`：

```c++
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);
```

The callback function:

回调函数：

```c++
virtual void OnActiveGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);
```

**[⬆ Back to Top](../README.md#table-of-contents)**
