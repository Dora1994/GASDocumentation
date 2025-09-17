# 4.4 Attribute Set （属性集）

## 4.4.1 Attribute Set Definition （属性集定义）

The `AttributeSet` defines, holds, and manages changes to `Attributes`. Developers should subclass from [`UAttributeSet`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAttributeSet/index.html). Creating an `AttributeSet` in an `OwnerActor's` constructor automatically registers it with its `ASC`. **This must be done in C++**.

`AttributeSet`定义、持有并管理`Attributes`的变化。开发者应该从[`UAttributeSet`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAttributeSet/index.html)继承子类。在`OwnerActor`的构造函数中创建`AttributeSet`会自动将其注册到其`ASC`。**这必须在C++中完成**。

## 4.4.2 Attribute Set Design （属性集设计）

An `ASC` may have one or many `AttributeSets`. AttributeSets have negligible memory overhead so how many `AttributeSets` to use is an organizational decision left up to the developer.

一个`ASC`可以有一个或多个`AttributeSets`。AttributeSets的内存开销可以忽略不计，所以使用多少个`AttributeSets`是留给开发者的组织决策。

It is acceptable to have one large monolithic `AttributeSet` shared by every `Actor` in your game and only use attributes if needed while ignoring unused attributes.

可以接受的做法是让游戏中的每个`Actor`共享一个大型整体`AttributeSet`，只在需要时使用属性，同时忽略未使用的属性。

Alternatively, you may choose to have more than one `AttributeSet` representing groupings of `Attributes` that you selectively add to your `Actors` as needed. For example, you could have an `AttributeSet` for health related `Attributes`, an `AttributeSet` for mana related `Attributes`, and so on. In a MOBA game, heroes might need mana but minions might not. Therefore the heroes would get the mana `AttributeSet` and minions would not.

或者，你可以选择拥有多个`AttributeSet`来表示`Attributes`的分组，根据需要有选择地添加到你的`Actors`中。例如，你可以有一个用于生命相关`Attributes`的`AttributeSet`，一个用于法力相关`Attributes`的`AttributeSet`，等等。在MOBA游戏中，英雄可能需要法力但小兵可能不需要。因此英雄会获得法力`AttributeSet`而小兵不会。

Additionally, `AttributeSets` can be subclassed as another means of selectively choosing which `Attributes` an `Actor` has. `Attributes` are internally referred to as `AttributeSetClassName.AttributeName`. When you subclass an `AttributeSet`, all of the `Attributes` from the parent class will still have the parent class's name as the prefix.

此外，`AttributeSets`可以被子类化作为选择性选择`Actor`拥有哪些`Attributes`的另一种方式。`Attributes`在内部被称为`AttributeSetClassName.AttributeName`。当你子类化`AttributeSet`时，来自父类的所有`Attributes`仍将以父类的名称作为前缀。

While you can have more than one `AttributeSet`, you should not have more than one `AttributeSet` of the same class on an `ASC`. If you have more than one `AttributeSet` from the same class, it won't know which `AttributeSet` to use and will just pick one.

虽然你可以有多个`AttributeSet`，但不应该在一个`ASC`上有多个同一类的`AttributeSet`。如果你有多个来自同一类的`AttributeSet`，它不知道使用哪个`AttributeSet`，只会随便选择一个。

### 4.4.2.1 Subcomponents with Individual Attributes （具有独立属性的子组件）

In the scenario where you have multiple damageable components on a `Pawn` like individually damageable armor pieces, I recommend that if you know the maximum number of damageable components that a `Pawn` could have to make that many health `Attributes` on one `AttributeSet` - DamageableCompHealth0, DamageableCompHealth1, etc. to represent logical 'slots' for those damageable components. In your damageable component class instance, assign the slot number `Attribute` that can be read by `GameplayAbilities` or [Executions](04-5-gameplay-effects.md) to know which `Attribute` to apply damage to. `Pawns` that have less than the maximum number or zero of damageable components are fine. Just because a `AttributeSet` has an `Attribute`, doesn't mean that you have to use it. Unused `Attributes` take up trivial amount of memory.

在你的`Pawn`上有多个可损坏组件（如可单独损坏的护甲片）的情况下，我建议如果你知道`Pawn`可能拥有的可损坏组件的最大数量，就在一个`AttributeSet`上创建那么多生命`Attributes` - DamageableCompHealth0、DamageableCompHealth1等，来表示这些可损坏组件的逻辑"槽位"。在你的可损坏组件类实例中，分配槽位编号`Attribute`，可以被`GameplayAbilities`或[Executions](04-5-gameplay-effects.md)读取，以知道将伤害应用到哪个`Attribute`。拥有少于最大数量或零个可损坏组件的`Pawns`是可以的。仅仅因为`AttributeSet`有一个`Attribute`，并不意味着你必须使用它。未使用的`Attribute`占用微不足道的内存量。

If your subcomponents need many `Attributes` each, there's potentially an unbounded number of subcomponents, the subcomponents can detach and be used by other players (e.g. weapons), or for any other reason this approach doesn't work for you, I'd recommend switching away from `Attributes` and instead store plain old floats on the components. See [Item Attributes](#concepts-as-design-itemattributes).

如果你的子组件每个都需要许多`Attributes`，可能有无限数量的子组件，子组件可以分离并被其他玩家使用（例如武器），或者由于任何其他原因这种方法对你不起作用，我建议放弃使用`Attributes`，而是在组件上存储普通的浮点数。参见[物品属性](#concepts-as-design-itemattributes)。

### 4.4.2.2 Adding and Removing AttributeSets at Runtime （运行时添加和移除AttributeSets）

`AttributeSets` can be added and removed from an `ASC` at runtime; however, removing `AttributeSets` can be dangerous. For example, if an `AttributeSet` is removed on a client before the server and an `Attribute` value change is replicated to client, the `Attribute` won't find its `AttributeSet` and crash the game.

`AttributeSets`可以在运行时从`ASC`中添加和移除；然而，移除`AttributeSets`可能是危险的。例如，如果`AttributeSet`在客户端上比服务器先被移除，而`Attributes`值变化被复制到客户端，`Attributes`将找不到其`AttributeSet`并使游戏崩溃。

On weapon add to inventory:

向库存添加武器时：

```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().AddUnique(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

On weapon remove from inventory:

从库存移除武器时：

```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().Remove(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

### 4.4.2.3 Item Attributes (Weapon Ammo) （物品属性（武器弹药））

There's a few ways to implement equippable items with `Attributes` (weapon ammo, armor durability, etc). All of these approaches store values directly on the item. This is necessary for items that can be equipped by more than one player over its lifetime.

有几种方式可以实现带有`Attributes`的可装备物品（武器弹药、护甲耐久度等）。所有这些方法都直接在物品上存储值。这对于在其生命周期内可以被多个玩家装备的物品是必要的。

> 1. Use plain floats on the item (**Recommended**)
> 1. Separate `AttributeSet` on the item
> 1. Separate `ASC` on the item

> 1. 在物品上使用普通浮点数（**推荐**）
> 1. 在物品上使用单独的`AttributeSet`
> 1. 在物品上使用单独的`ASC`

**[⬆ Back to Top](../README.md#table-of-contents)**
