# 5. Commonly Implemented Abilities and Effects (常用实现的能力和效果)

## 5.1 Stun (眩晕)

Typically with stuns, we want to cancel all of a `Character's` active `GameplayAbilities`, prevent new `GameplayAbility` activations, and prevent movement throughout the duration of the stun. The Sample Project's Meteor `GameplayAbility` applies a stun on hit targets.

通常对于眩晕，我们希望取消`角色`的所有活跃`GameplayAbilities`，防止新的`GameplayAbility`激活，并在眩晕持续期间防止移动。示例项目的流星`GameplayAbility`对命中目标应用眩晕。

To cancel the target's active `GameplayAbilities`, we call `AbilitySystemComponent->CancelAbilities()` when the stun [GameplayTag is added](04-concepts/04-2-gameplay-tags.md).

要取消目标的活跃`GameplayAbility`，我们在眩晕[GameplayTag is added](04-concepts/04-2-gameplay-tags.md)时调用`AbilitySystemComponent->CancelAbilities()`。

To prevent new `GameplayAbilities` from activating while stunned, the `GameplayAbilities` are given the stun `GameplayTag` in their [Activation Blocked Tags `GameplayTagContainer`](04-concepts/04-6-gameplay-abilities.md).

为了防止在眩晕时激活新的`GameplayAbility`，`GameplayAbility`在其[Activation Blocked Tags`GameplayTagContainer`](04-concepts/04-6-gameplay-abilities.md)中被给予眩晕`GameplayTag`。

To prevent movement while stunned, we override the `CharacterMovementComponent's` `GetMaxSpeed()` function to return 0 when the owner has the stun `GameplayTag`.

为了防止眩晕时移动，我们重写`CharacterMovementComponent`的`GetMaxSpeed()`函数，当拥有者具有眩晕`GameplayTag`时返回0。

## 5.2 Sprint (冲刺)

The Sample Project provides an example of how to sprint - run faster while `Left Shift` is held down.

示例项目提供了如何冲刺的示例 - 按住`左Shift`键时跑得更快。

The faster movement is handled predictively by the `CharacterMovementComponent` by sending a flag over the network to the server. See `GDCharacterMovementComponent.h/cpp` for details.

更快的移动由`CharacterMovementComponent`通过向服务器发送网络标志来预测性地处理。详细信息请参见`GDCharacterMovementComponent.h/cpp`。

The `GA` handles responding to the `Left Shift` input, tells the `CMC` to begin and stop sprinting, and to predictively charge stamina while `Left Shift` is pressed. See `GA_Sprint_BP` for details.

`GA`处理对`左Shift`输入的响应，告诉`CMC`开始和停止冲刺，并在按下`左Shift`时预测性地消耗体力。详细信息请参见`GA_Sprint_BP`。

## 5.3 Aim Down Sights (瞄准)

The Sample Project handles this the exact same way as sprinting but decreasing the movement speed instead of increasing it.

示例项目以与冲刺完全相同的方式处理这个问题，但降低移动速度而不是增加。

See `GDCharacterMovementComponent.h/cpp` for details on predictively decreasing the movement speed.

有关预测性降低移动速度的详细信息，请参见`GDCharacterMovementComponent.h/cpp`。

See `GA_AimDownSight_BP` for details on handling the input. There is no stamina cost for aiming down sights.

有关处理输入的详细信息，请参见`GA_AimDownSight_BP`。瞄准没有体力消耗。

## 5.4 Lifesteal (生命偷取)

I handle lifesteal inside of the damage [ExecutionCalculation](04-concepts/04-5-gameplay-effects.md). The `GameplayEffect` will have a `GameplayTag` on it like `Effect.CanLifesteal`. The `ExecutionCalculation` checks if the `GameplayEffectSpec` has that `Effect.CanLifesteal` `GameplayTag`. If the `GameplayTag` exists, the `ExecutionCalculation` [creates a dynamic `Instant` `GameplayEffect`](04-concepts/04-5-gameplay-effects.md) with the amount of health to give as the modifier and applies it back to the `Source's` `ASC`.

我在伤害[ExecutionCalculation](04-concepts/04-5-gameplay-effects.md)内部处理生命偷取。`GameplayEffect`将有一个`GameplayTag`，如`Effect.CanLifesteal`。`ExecutionCalculation`检查`GameplayEffectSpec`是否具有该`Effect.CanLifesteal` `GameplayTag`。如果`GameplayTag`存在，`ExecutionCalculation`[创建一个动态`Instant` `GameplayEffect`](04-concepts/04-5-gameplay-effects.md)，将要给予的生命值作为修饰符，并将其应用回`Source's` `ASC`。

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
