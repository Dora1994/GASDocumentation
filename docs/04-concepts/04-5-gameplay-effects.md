# 4.5 Gameplay Effects

## 4.5.1 Gameplay Effect Definition

[`GameplayEffects`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html) (`GE`) are the vessels through which abilities change [Attributes](04-3-attributes.md) and [GameplayTags](04-2-gameplay-tags.md) on themselves and others. They can cause immediate `Attribute` changes like damage or healing or apply long term status buff/debuffs like a movespeed boost or stunning. The `UGameplayEffect` class is a meant to be a **data-only** class that defines a single gameplay effect. No additional logic should be added to `GameplayEffects`. Typically designers will create many Blueprint child classes of `UGameplayEffect`.

`GameplayEffects` change `Attributes` through [Modifiers](#concepts-ge-mods) and [Executions (`GameplayEffectExecutionCalculation`)](#concepts-ge-ec).

`GameplayEffects` have three types of duration: `Instant`, `Duration`, and `Infinite`.

Additionally, `GameplayEffects` can add/execute [GameplayCues](04-8-gameplay-cues.md). An `Instant` `GameplayEffect` will call `Execute` on the `GameplayCue` `GameplayTags` whereas a `Duration` or `Infinite` `GameplayEffect` will call `Add` and `Remove` on the `GameplayCue` `GameplayTags`.

| Duration Type | GameplayCue Event | When to use                                                                                                                                                                                                                                |
| ------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Instant`     | Execute           | For immediate permanent changes to `Attribute's` `BaseValue`. `GameplayTags` will not be applied, not even for a frame.                                                                                                                    |
| `Duration`    | Add & Remove      | For temporary changes to `Attribute's` `CurrentValue` and to apply `GameplayTags` that will be removed when the `GameplayEffect` expires or is manually removed. The duration is specified in the `UGameplayEffect` class/Blueprint.       |
| `Infinite`    | Add & Remove      | For temporary changes to `Attribute's` `CurrentValue` and to apply `GameplayTags` that will be removed when the `GameplayEffect` is removed. These will never expire on their own and must be manually removed by an ability or the `ASC`. |

`Duration` and `Infinite` `GameplayEffects` have the option of applying `Periodic Effects` that apply its `Modifiers` and `Executions` every `X` seconds as defined by its `Period`. `Periodic Effects` are treated as `Instant` `GameplayEffects` when it comes to changing the `Attribute's` `BaseValue` and `Executing` `GameplayCues`. These are useful for damage over time (DOT) type effects. **Note:** `Periodic Effects` cannot be [predicted](04-10-prediction.md).

`Duration` and `Infinite` `GameplayEffects` can be temporarily turned off and on after application if their `Ongoing Tag Requirements` are not met/met ([Gameplay Effect Tags](#concepts-ge-tags)). Turning off a `GameplayEffect` removes the effects of its `Modifiers` and applied `GameplayTags` but does not remove the `GameplayEffect`. Turning the `GameplayEffect` back on reapplies its `Modifiers` and `GameplayTags`.

If you need to manually recalculate the `Modifiers` of a `Duration` or `Infinite` `GameplayEffect` (say you have an `MMC` that uses data that doesn't come from `Attributes`), you can call `UAbilitySystemComponent::ActiveGameplayEffects.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)` with the same level that it already has using `UAbilitySystemComponent::ActiveGameplayEffects.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()`. `Modifiers` that are based on backing `Attributes` automatically update when those backing `Attributes` update. The key functions of `SetActiveGameplayEffectLevel()` to update the `Modifiers` are:

```C++
MarkItemDirty(Effect);
Effect.Spec.CalculateModifierMagnitudes();
// Private function otherwise we'd call these three functions without needing to set the level to what it already is
UpdateAllAggregatorModMagnitudes(Effect);
```

`GameplayEffects` are not typically instantiated. When an ability or `ASC` wants to apply a `GameplayEffect`, it creates a [GameplayEffectSpec](#concepts-ge-spec) from the `GameplayEffect's` `ClassDefaultObject`. Successfully applied `GameplayEffectSpecs` are then added to a new struct called `FActiveGameplayEffect` which is what the `ASC` keeps track of in a special container struct called `ActiveGameplayEffects`.

## 4.5.2 Applying Gameplay Effects

`GameplayEffects` can be applied in many ways from functions on [GameplayAbilities](04-6-gameplay-abilities.md) and functions on the `ASC` and usually take the form of `ApplyGameplayEffectTo`. The different functions are essentially convenience functions that will eventually call `UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf()` on the `Target`.

To apply `GameplayEffects` outside of a `GameplayAbility` for example from a projectile, you need to get the `Target's` `ASC` and use one of its functions to `ApplyGameplayEffectToSelf`.

You can listen for when any `Duration` or `Infinite` `GameplayEffects` are applied to an `ASC` by binding to its delegate:
```c++
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);
```
The callback function:
```c++
virtual void OnActiveGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);
```

**[⬆ Back to Top](../README.md#table-of-contents)**
