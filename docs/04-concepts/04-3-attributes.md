# 4.3 Attributes （属性）

## 4.3.1 Attribute Definition （属性定义）

`Attributes` are float values defined by the struct [`FGameplayAttributeData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html). These can represent anything from the amount of health a character has to the character's level to the number of charges that a potion has. If it is a gameplay-related numerical value belonging to an `Actor`, you should consider using an `Attribute` for it. `Attributes` should generally only be modified by [GameplayEffects](04-5-gameplay-effects.md) so that the ASC can [predict](04-10-prediction.md) the changes.

`Attributes`是由结构体[`FGameplayAttributeData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html)定义的浮点值。这些可以表示从角色拥有的生命值数量到角色等级到药水充能次数的任何东西。如果它是属于 `Actor`的游戏相关数值，你应该考虑为它使用 `Attribute`。`Attribute`通常只应该由[GameplayEffects](04-5-gameplay-effects.md)修改，这样ASC就可以[predict](04-10-prediction.md)变化。

`Attributes` are defined by and live in an [AttributeSet](04-4-attribute-set.md). The `AttributeSet` is responsible for replicating `Attributes` that are marked for replication. See the section on [AttributeSets](04-4-attribute-set.md) for how to define `Attributes`.

`Attributes`由[AttributeSet](04-4-attribute-set.md)定义并存在于其中。`AttributeSet`负责复制标记为复制的 `Attributes`。有关如何定义 `Attributes`，请参见[AttributeSets](04-4-attribute-set.md)部分。

**Tip:** If you don't want an `Attribute` to show up in the Editor's list of `Attributes`, you can use the `Meta = (HideInDetailsView)` `property specifier`.

**提示：**如果你不希望 `Attribute`出现在编辑器的 `Attributes`列表中，你可以使用 `Meta = (HideInDetailsView)` `property specifier`。

## 4.3.2 BaseValue vs CurrentValue （基础值 vs 当前值）

An `Attribute` is composed of two values - a `BaseValue` and a `CurrentValue`. The `BaseValue` is the permanent value of the `Attribute` whereas the `CurrentValue` is the `BaseValue` plus temporary modifications from `GameplayEffects`. For example, your `Character` may have a movespeed `Attribute` with a `BaseValue` of 600 units/second. Since there are no `GameplayEffects` modifying the movespeed yet, the `CurrentValue` is also 600 u/s. If she gets a temporary 50 u/s movespeed buff, the `BaseValue` stays the same at 600 u/s while the `CurrentValue` is now 600 + 50 for a total of 650 u/s. When the movespeed buff expires, the `CurrentValue` reverts back to the `BaseValue` of 600 u/s.

`Attribute`由两个值组成 - `BaseValue`和 `CurrentValue`。`BaseValue`是 `Attribute`的永久值，而 `CurrentValue`是 `BaseValue`加上来自 `GameplayEffects`的临时修改。例如，你的 `Character`可能有一个移动速度 `Attribute`，`BaseValue`为600单位/秒。由于还没有 `GameplayEffects`修改移动速度，`CurrentValue`也是600 u/s。如果她获得一个临时的50 u/s移动速度增益，`BaseValue`保持600 u/s不变，而 `CurrentValue`现在是600 + 50，总共650 u/s。当移动速度增益过期时，`CurrentValue`恢复到600 u/s的 `BaseValue`。

Often beginners to GAS will confuse `BaseValue` with a maximum value for an `Attribute` and try to treat it as such. This is an incorrect approach. Maximum values for `Attributes` that can change or are referenced in abilities or UI should be treated as separate `Attributes`. For hardcoded maximum and minimum values, there is a way to define a `DataTable` with `FAttributeMetaData` that can set maximum and minimum values, but Epic's comment above the struct calls it a "work in progress". See `AttributeSet.h` for more information. To prevent confusion, I recommend that maximum values that can be referenced in abilities or UI be made as separate `Attributes` and hardcoded maximum and minimum values that are only used for clamping `Attributes` be defined as hardcoded floats in the `AttributeSet`. Clamping of `Attributes` is discussed in [PreAttributeChange()](04-4-attribute-set.md) for changes to the `CurrentValue` and [PostGameplayEffectExecute()](04-4-attribute-set.md) for changes to the `BaseValue` from `GameplayEffects`.

GAS的初学者经常将 `BaseValue`与 `Attribute`的最大值混淆，并试图将其视为最大值。这是一种错误的方法。可以在能力或UI中引用或变化的 `Attribute`的最大值应该被视为单独的 `Attribute`。对于硬编码的最大值和最小值，有一种方法可以定义一个带有 `FAttributeMetaData`的 `DataTable`来设置最大值和最小值，但Epic在该结构上方的注释称其为"正在进行的工作"。更多信息请参见 `AttributeSet.h`。为了防止混淆，我建议可以在能力或UI中引用的最大值应该作为单独的 `Attribute`，而仅用于限制 `Attribute`的硬编码最大值和最小值应该在 `AttributeSet`中定义为硬编码浮点数。`Attribute`的限制在[PreAttributeChange()](04-4-attribute-set.md)中讨论 `CurrentValue`的变化，在[PostGameplayEffectExecute()](04-4-attribute-set.md)中讨论来自 `GameplayEffects`的 `BaseValue`变化。

Permanent changes to the `BaseValue` come from `Instant` `GameplayEffects` whereas `Duration` and `Infinite` `GameplayEffects` change the `CurrentValue`. Periodic `GameplayEffects` are treated like instant `GameplayEffects` and change the `BaseValue`.

对 `BaseValue`的永久变化来自 `Instant` `GameplayEffects`，而 `Duration`和 `Infinite` `GameplayEffects`改变 `CurrentValue`。周期性 `GameplayEffects`被视为即时 `GameplayEffects`并改变 `BaseValue`。

## 4.3.3 Meta Attributes （元属性）

Some `Attributes` are treated as placeholders for temporary values that are intended to interact with `Attributes`. These are called `Meta Attributes`. For example, we commonly define damage as a `Meta Attribute`. Instead of a `GameplayEffect` directly changing our health `Attribute`, we use a `Meta Attribute` called damage as a placeholder. This way the damage value can be modified with buffs and debuffs in an [GameplayEffectExecutionCalculation](04-5-gameplay-effects.md) and can be further manipulated in the `AttributeSet`, for example subtracting the damage from a current shield `Attribute`, before finally subtracting the remainder from the health `Attribute`. The damage `Meta Attribute` has no persistence between `GameplayEffects` and is overriden by every one. `Meta Attributes` are not typically replicated.

一些 `Attributes`被视为临时值的占位符，这些临时值旨在与 `Attribute`交互。这些被称为 `Meta Attributes`。例如，我们通常将伤害定义为 `Meta Attribute`。不是让 `GameplayEffect`直接改变我们的生命 `Attribute`，而是使用名为伤害的 `Meta Attribute`作为占位符。这样伤害值可以在[GameplayEffectExecutionCalculation](04-5-gameplay-effects.md)中通过增益和减益进行修改，并可以在 `AttributeSet`中进一步操作，例如从当前护盾 `Attribute`中减去伤害，然后最终从生命 `Attribute`中减去剩余部分。伤害 `Meta Attribute`在 `GameplayEffects`之间没有持久性，每次都会被覆盖。`Meta Attributes`通常不会被复制。

`Meta Attributes` provide a good logical separation for things like damage and healing between "How much damage did we do?" and "What do we do with this damage?". This logical separation means our `Gameplay Effects` and `Execution Calculations` don't need to know how the Target handles the damage. Continuing our damage example, the `Gameplay Effect` determines how much damage and then the `AttributeSet` decides what to do with that damage. Not all characters may have the same `Attributes`, especially if you use subclassed `AttributeSets`. The base `AttributeSet` class may only have a health `Attribute`, but a subclassed `AttributeSet` may add a shield `Attribute`. The subclassed `AttributeSet` with the shield `Attribute` would distribute the damage received differently than the base `AttributeSet` class.

`Meta Attributes`为伤害和治疗等事物提供了良好的逻辑分离，在"我们造成了多少伤害？"和"我们如何处理这个伤害？"之间。这种逻辑分离意味着我们的 `GameplayEffects`和 `Execution Calculations`不需要知道目标如何处理伤害。继续我们的伤害示例，`GameplayEffect`确定造成多少伤害，然后 `AttributeSet`决定如何处理这个伤害。不是所有角色都有相同的 `Attributes`，特别是如果你使用子类化的 `AttributeSet`。基础 `AttributeSet`类可能只有生命 `Attribute`，但子类化的 `AttributeSet`可能添加护盾 `Attribute`。带有护盾 `Attribute`的子类化 `AttributeSet`会以不同于基础 `AttributeSet`类的方式分配受到的伤害。

While `Meta Attributes` are a good design pattern, they are not mandatory. If you only ever have one `Execution Calculation` used for all instances of damage and one `Attribute Set` class shared by all characters, then you may be fine doing the damage distribution to health, shields, etc. inside of the `Execution Calculation` and directly modifying those `Attributes`. You'll only be sacrificing flexibility, but that may be okay for you.

虽然 `Meta Attributes`是一个好的设计模式，但它们不是强制性的。如果你只有一个用于所有伤害实例的 `Execution Calculation`和一个由所有角色共享的 `Attribute Set`类，那么你可能可以在 `Execution Calculation`内部进行伤害分配到生命、护盾等，并直接修改那些 `Attributes`。你只会牺牲灵活性，但这可能对你来说是可以的。

## 4.3.4 Responding to Attribute Changes （响应属性变化）

To listen for when an `Attribute` changes to update the UI or other gameplay, use `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`. This function returns a delegate that you can bind to that will be automatically called whenever an `Attribute` changes. The delegate provides a `FOnAttributeChangeData` parameter with the `NewValue`, `OldValue`, and `FGameplayEffectModCallbackData`. **Note:** The `FGameplayEffectModCallbackData` will only be set on the server.

要监听 `Attribute`变化以更新UI或其他游戏玩法，使用 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`。这个函数返回一个委托，你可以绑定它，每当 `Attribute`变化时都会自动调用。委托提供一个 `FOnAttributeChangeData`参数，包含 `NewValue`、`OldValue`和 `FGameplayEffectModCallbackData`。**注意：**`FGameplayEffectModCallbackData`只会在服务器上设置。

```c++
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(AttributeSetBase->GetHealthAttribute()).AddUObject(this, &AGDPlayerState::HealthChanged);
```

```c++
virtual void HealthChanged(const FOnAttributeChangeData& Data);
```

The Sample Project binds to the `Attribute` value changed delegates on the `GDPlayerState` to update the HUD and to respond to player death when health reaches zero.

示例项目在 `GDPlayerState`上绑定到 `Attribute`值变化委托，以更新HUD并在生命值达到零时响应玩家死亡。

A custom Blueprint node that wraps this into an `ASyncTask` is included in the Sample Project. It is used in the `UI_HUD` UMG Widget to update the health, mana, and stamina values. This `AsyncTask` will live forever until manually called `EndTask()`, which we do in the UMG Widget's `Destruct` event. See `AsyncTaskAttributeChanged.h/cpp`.

示例项目包含一个自定义蓝图节点，将其包装成 `ASyncTask`。它用于 `UI_HUD` UMG Widget中更新生命、法力和体力值。这个 `AsyncTask`将永远存在，直到手动调用 `EndTask()`，我们在UMG Widget的 `Destruct`事件中这样做。参见 `AsyncTaskAttributeChanged.h/cpp`。

![Listen for Attribute Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/attributechange.png)

## 4.3.5 Derived Attributes （派生属性）

To make an `Attribute` that has some or all of its value derived from one or more other `Attributes`, use an `Infinite` `GameplayEffect` with one or more `Attribute Based` or [MMC](04-5-gameplay-effects.md) [Modifiers](04-5-gameplay-effects.md). The `Derived Attribute` will update automatically when an `Attribute` that it depends on is updated.

要创建一个其值的部分或全部来自一个或多个其他 `Attribute`的 `Attribute`，使用带有 `Attribute Based`或[MMC](04-5-gameplay-effects.md) [Modifiers](04-5-gameplay-effects.md)的 `Infinite` `GameplayEffect`。当 `Derived Attribute`依赖的 `Attribute`更新时，它会自动更新。

The final formula for all the `Modifiers` on a `Derived Attribute` is the same formula for `Modifier Aggregators`. If you need calculations to happen in a certain order, do it all inside of an `MMC`.

`Derived Attribute`上所有 `Modifiers`的最终公式与 `Modifier Aggregators`的公式相同。如果你需要按特定顺序进行计算，在 `MMC`内部完成所有计算。

```
((CurrentValue + Additive) * Multiplicitive) / Division
```

**Note:** If playing with multiple clients in PIE, you need to disable `Run Under One Process` in the Editor Preferences otherwise the `Derived Attributes` will not update when their independent `Attributes` update on clients other than the first.

**注意：**如果在PIE中使用多个客户端，你需要在编辑器首选项中禁用 `在一个进程中运行`，否则当独立 `Attribute`在除第一个之外的客户端上更新时，`Derived Attribute`不会更新。

In this example, we have an `Infinite` `GameplayEffect` that derives the value of `TestAttrA` from the `Attributes`, `TestAttrB` and `TestAttrC`, in the formula `TestAttrA = (TestAttrA + TestAttrB) * ( 2 * TestAttrC)`. `TestAttrA` recalculates its value automatically whenever any of the `Attributes` update their values.

在这个示例中，我们有一个 `Infinite` `GameplayEffect`，它从 `Attributes` `TestAttrB`和 `TestAttrC`派生 `TestAttrA`的值，公式为 `TestAttrA = (TestAttrA + TestAttrB) * ( 2 * TestAttrC)`。每当任何 `Attributes`更新其值时，`TestAttrA`都会自动重新计算其值。

![Derived Attribute Example](https://github.com/tranek/GASDocumentation/raw/master/Images/derivedattribute.png)

**[⬆ Back to Top](../README.md#table-of-contents)**
