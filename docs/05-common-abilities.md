# 5. Commonly Implemented Abilities and Effects

## 5. 常用实现的能力和效果

## 5.1 Stun

## 5.1 眩晕

Typically with stuns, we want to cancel all of a `Character's` active `GameplayAbilities`, prevent new `GameplayAbility` activations, and prevent movement throughout the duration of the stun. The Sample Project's Meteor `GameplayAbility` applies a stun on hit targets.

通常对于眩晕，我们希望取消`角色`的所有活跃`游戏能力`，防止新的`游戏能力`激活，并在眩晕持续期间防止移动。示例项目的流星`游戏能力`对命中目标应用眩晕。

To cancel the target's active `GameplayAbilities`, we call `AbilitySystemComponent->CancelAbilities()` when the stun [GameplayTag is added](04-concepts/04-2-gameplay-tags.md).

要取消目标的活跃`游戏能力`，我们在眩晕[游戏标签被添加](04-concepts/04-2-gameplay-tags.md)时调用`AbilitySystemComponent->CancelAbilities()`。

To prevent new `GameplayAbilities` from activating while stunned, the `GameplayAbilities` are given the stun `GameplayTag` in their [Activation Blocked Tags `GameplayTagContainer`](04-concepts/04-6-gameplay-abilities.md).

为了防止在眩晕时激活新的`游戏能力`，`游戏能力`在其[激活阻止标签`游戏标签容器`](04-concepts/04-6-gameplay-abilities.md)中被给予眩晕`游戏标签`。

To prevent movement while stunned, we override the `CharacterMovementComponent's` `GetMaxSpeed()` function to return 0 when the owner has the stun `GameplayTag`.

为了防止眩晕时移动，我们重写`CharacterMovementComponent`的`GetMaxSpeed()`函数，当拥有者具有眩晕`游戏标签`时返回0。

## 5.2 Sprint

## 5.2 冲刺

The Sample Project provides an example of how to sprint - run faster while `Left Shift` is held down.

示例项目提供了如何冲刺的示例 - 按住`左Shift`键时跑得更快。

The faster movement is handled predictively by the `CharacterMovementComponent` by sending a flag over the network to the server. See `GDCharacterMovementComponent.h/cpp` for details.

更快的移动由`CharacterMovementComponent`通过向服务器发送网络标志来预测性地处理。详细信息请参见`GDCharacterMovementComponent.h/cpp`。

The `GA` handles responding to the `Left Shift` input, tells the `CMC` to begin and stop sprinting, and to predictively charge stamina while `Left Shift` is pressed. See `GA_Sprint_BP` for details.

`GA`处理对`左Shift`输入的响应，告诉`CMC`开始和停止冲刺，并在按下`左Shift`时预测性地消耗体力。详细信息请参见`GA_Sprint_BP`。

## 5.3 Aim Down Sights

## 5.3 瞄准

The Sample Project handles this the exact same way as sprinting but decreasing the movement speed instead of increasing it.

示例项目以与冲刺完全相同的方式处理这个问题，但降低移动速度而不是增加。

See `GDCharacterMovementComponent.h/cpp` for details on predictively decreasing the movement speed.

有关预测性降低移动速度的详细信息，请参见`GDCharacterMovementComponent.h/cpp`。

See `GA_AimDownSight_BP` for details on handling the input. There is no stamina cost for aiming down sights.

有关处理输入的详细信息，请参见`GA_AimDownSight_BP`。瞄准没有体力消耗。

## 5.4 Lifesteal

## 5.4 生命偷取

I handle lifesteal inside of the damage [ExecutionCalculation](04-concepts/04-5-gameplay-effects.md). The `GameplayEffect` will have a `GameplayTag` on it like `Effect.CanLifesteal`. The `ExecutionCalculation` checks if the `GameplayEffectSpec` has that `Effect.CanLifesteal` `GameplayTag`. If the `GameplayTag` exists, the `ExecutionCalculation` [creates a dynamic `Instant` `GameplayEffect`](04-concepts/04-5-gameplay-effects.md) with the amount of health to give as the modifier and applies it back to the `Source's` `ASC`.

我在伤害[执行计算](04-concepts/04-5-gameplay-effects.md)内部处理生命偷取。`游戏效果`将有一个`游戏标签`，如`Effect.CanLifesteal`。`执行计算`检查`游戏效果规格`是否具有该`Effect.CanLifesteal` `游戏标签`。如果`游戏标签`存在，`执行计算`[创建一个动态`即时` `游戏效果`](04-concepts/04-5-gameplay-effects.md)，将要给予的生命值作为修饰符，并将其应用回`源`的`ASC`。

```c++
if (SpecAssetTags.HasTag(FGameplayTag::RequestGameplayTag(FName("Effect.Damage.CanLifesteal"))))
{
	float Lifesteal = Damage * LifestealPercent;

	UGameplayEffect* GELifesteal = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Lifesteal")));
	GELifesteal->DurationPolicy = EGameplayEffectDurationType::Instant;

	int32 Idx = GELifesteal->Modifiers.Num();
	GELifesteal->Modifiers.SetNum(Idx + 1);
	FGameplayModifierInfo& Info = GELifesteal->Modifiers[Idx];
	Info.ModifierMagnitude = FScalableFloat(Lifesteal);
	Info.ModifierOp = EGameplayModOp::Additive;
	Info.Attribute = UGDAttributeSetBase::GetHealthAttribute();

	SourceASC->ApplyGameplayEffectToSelf(GELifesteal, 1.0f, SourceASC->MakeEffectContext());
}
```

**[⬆ Back to Top](../README.md#table-of-contents)**
