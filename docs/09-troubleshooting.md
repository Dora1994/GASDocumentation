# 9. Troubleshooting （故障排除）

## 9.1 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`

You need to [initialize the `ASC` on the client](04-concepts/04-1-ability-system-component.md).

你需要[在客户端初始化`ASC`](04-concepts/04-1-ability-system-component.md)。

## 9.2 `ScriptStructCache` errors

You need to call [`UAbilitySystemGlobals::InitGlobalData()`](04-concepts/04-9-ability-system-globals.md).

你需要调用[`UAbilitySystemGlobals::InitGlobalData()`](04-concepts/04-9-ability-system-globals.md)。

## 9.3 Animation Montages are not replicating to clients （动画蒙太奇没有复制到客户端）

Make sure that you're using the `PlayMontageAndWait` Blueprint node instead of `PlayMontage` in your [GameplayAbilities](04-concepts/04-6-gameplay-abilities.md). This [AbilityTask](04-concepts/04-7-ability-tasks.md) replicates the montage through the `ASC` automatically whereas the `PlayMontage` node does not.

确保你在[GameplayAbilities](04-concepts/04-6-gameplay-abilities.md)中使用`PlayMontageAndWait`蓝图节点而不是`PlayMontage`。这个[AbilityTask](04-concepts/04-7-ability-tasks.md)通过`ASC`自动复制蒙太奇，而`PlayMontage`节点不会。

## 9.4 Duplicating Blueprint Actors is setting AttributeSets to nullptr （复制蓝图Actor将AttributeSets设置为nullptr）


There is a [bug in Unreal Engine](https://issues.unrealengine.com/issue/UE-81109) that will set `AttributeSet` pointers on your classes to nullptr for Blueprint Actor classes that are duplicated from existing Blueprint Actor classes. There are a few workarounds for this. I've had success not creating bespoke `AttributeSet` pointers on my classes (no pointer in the .h, not calling `CreateDefaultSubobject` in the constructor) and instead just directly adding `AttributeSets` to the `ASC` in `PostInitializeComponents()` (not shown in the Sample Project). The replicated `AttributeSets` will still live in the `ASC's` `SpawnedAttributes` array. It would look something like this:

虚幻引擎中有一个[bug](https://issues.unrealengine.com/issue/UE-81109)，对于从现有蓝图Actor类复制的蓝图Actor类，会将你类上的`AttributeSet`指针设置为nullptr。对此有一些解决方法。我成功地没有在我的类上创建定制的`AttributeSet`指针（.h中没有指针，构造函数中不调用`CreateDefaultSubobject`），而是在`PostInitializeComponents()`中直接将`AttributeSets`添加到`ASC`（示例项目中未显示）。复制的`AttributeSets`仍将存在于`ASC`的`SpawnedAttributes`数组中。它看起来像这样：

```c++
void AGDPlayerState::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->AddSet<UGDAttributeSetBase>();
		// ... any other AttributeSets that you may have
	}
}
```

In this scenario, you would read and set the values in the `AttributeSet` using the functions on the `ASC` instead of [calling functions on the `AttributeSet` made from the macros](04-concepts/04-4-attribute-set.md).

在这种情况下，你将使用`ASC`上的函数读取和设置`AttributeSet`中的值，而不是[调用从宏创建的`AttributeSet`上的函数](04-concepts/04-4-attribute-set.md)。

```c++
/** Returns current (final) value of an attribute */
float GetNumericAttribute(const FGameplayAttribute &Attribute) const;

/** Sets the base value of an attribute. Existing active modifiers are NOT cleared and will act upon the new base value. */
void SetNumericAttributeBase(const FGameplayAttribute &Attribute, float NewBaseValue);
```

```c++
/** 返回属性的当前（最终）值 */
float GetNumericAttribute(const FGameplayAttribute &Attribute) const;

/** 设置属性的基础值。现有的活跃修饰符不会被清除，并将作用于新的基础值。 */
void SetNumericAttributeBase(const FGameplayAttribute &Attribute, float NewBaseValue);
```

So the `GetHealth()` would look something like:

所以`GetHealth()`看起来像这样：

```c++
float AGDPlayerState::GetHealth() const
{
	return AbilitySystemComponent->GetNumericAttribute(UGDAttributeSetBase::GetHealthAttribute());
}
```

**[⬆ Back to Top](../README.md#table-of-contents)**
