# 4.1 Ability System Component （能力系统组件）


The `AbilitySystemComponent` (`ASC`) is the heart of GAS. It's a `UActorComponent` ([`UAbilitySystemComponent`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemComponent/index.html)) that handles all interactions with the system. Any `Actor` that wishes to use [GameplayAbilities](04-6-gameplay-abilities.md), have [Attributes](04-3-attributes.md), or receive [GameplayEffects](04-5-gameplay-effects.md) must have one `ASC` attached to them. These objects all live inside of and are managed and replicated by (with the exception of `Attributes` which are replicated by their [AttributeSet](04-4-attribute-set.md)) the `ASC`. Developers are expected but not required to subclass this.

`AbilitySystemComponent`（`ASC`）是GAS的核心。它是一个`UActorComponent`（[`UAbilitySystemComponent`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemComponent/index.html)），处理与系统的所有交互。任何希望使用[GameplayAbilities](04-6-gameplay-abilities.md)、拥有[Attributes](04-3-attributes.md)或接收[GameplayEffects](04-5-gameplay-effects.md)的`Actor`都必须附加一个`ASC`。这些对象都存在于`ASC`内部，并由`ASC`管理和复制（除了由[AttributeSet](04-4-attribute-set.md)复制的`Attributes`）。开发者应该但不必须子类化它。

The `Actor` with the `ASC` attached to it is referred to as the `OwnerActor` of the `ASC`. The physical representation `Actor` of the `ASC` is called the `AvatarActor`. The `OwnerActor` and the `AvatarActor` can be the same `Actor` as in the case of a simple AI minion in a MOBA game. They can also be different `Actors` as in the case of a player controlled hero in a MOBA game where the `OwnerActor` is the `PlayerState` and the `AvatarActor` is the hero's `Character` class. Most `Actors` will have the `ASC` on themselves. If your `Actor` will respawn and need persistence of `Attributes` or `GameplayEffects` between spawns (like a hero in a MOBA), then the ideal location for the `ASC` is on the `PlayerState`.

附加了`ASC`的`Actor`被称为`ASC`的`OwnerActor`。`ASC`的物理表示`Actor`被称为`AvatarActor`。`OwnerActor`和`AvatarActor`可以是同一个`Actor`，就像MOBA游戏中简单的AI小兵一样。它们也可以是不同的`Actor`，就像MOBA游戏中玩家控制的英雄，其中`OwnerActor`是`PlayerState`，`AvatarActor`是英雄的`Character`类。大多数`Actor`会在自己身上有`ASC`。如果你的`Actor`会重生并且需要在重生之间保持`Attributes`或`GameplayEffects`的持久性（就像MOBA中的英雄），那么`ASC`的理想位置是在`PlayerState`上。

**Note:** If your `ASC` is on your `PlayerState`, then you will need to increase the `NetUpdateFrequency` of your `PlayerState`. It defaults to a very low value on the `PlayerState` and can cause delays or perceived lag before changes to things like `Attributes` and `GameplayTags` happen on the clients. Be sure to enable [`Adaptive Network Update Frequency`](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/Properties/index.html#adaptivenetworkupdatefrequency), Fortnite uses it.

**注意：**如果你的`ASC`在你的`PlayerState`上，那么你需要增加你的`PlayerState`的`NetUpdateFrequency`。它在`PlayerState`上默认为一个非常低的值，可能会导致在客户端上`Attributes`和`GameplayTags`等变化发生之前的延迟或感知延迟。确保启用[`Adaptive Network Update Frequency`](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/Properties/index.html#adaptivenetworkupdatefrequency)，Fortnite使用了它。

Both, the `OwnerActor` and the `AvatarActor` if different `Actors`, should implement the `IAbilitySystemInterface`. This interface has one function that must be overriden, `UAbilitySystemComponent* GetAbilitySystemComponent() const`, which returns a pointer to its `ASC`. `ASCs` interact with each other internally to the system by looking for this interface function.

如果`OwnerActor`和`AvatarActor`是不同的`Actor`，两者都应该实现`IAbilitySystemInterface`。这个接口有一个必须重写的函数，`UAbilitySystemComponent* GetAbilitySystemComponent() const`，它返回指向其`ASC`的指针。`ASCs`通过查找这个接口函数在系统内部相互交互。

The `ASC` holds its current active `GameplayEffects` in `FActiveGameplayEffectsContainer ActiveGameplayEffects`.

`ASC`在`FActiveGameplayEffectsContainer ActiveGameplayEffects`中保存其当前活跃的`GameplayEffects`。

The `ASC` holds its granted `Gameplay Abilities` in `FGameplayAbilitySpecContainer ActivatableAbilities`. Any time that you plan to iterate over `ActivatableAbilities.Items`, be sure to add `ABILITYLIST_SCOPE_LOCK();` above your loop to lock the list from changing (due to removing an ability). Every `ABILITYLIST_SCOPE_LOCK();` in scope increments `AbilityScopeLockCount` and then decrements when it falls out of scope. Do not try to remove an ability inside the scope of `ABILITYLIST_SCOPE_LOCK();` (the clear ability functions check `AbilityScopeLockCount` internally to prevent removing abilities if the list is locked).

`ASC`在`FGameplayAbilitySpecContainer ActivatableAbilities`中保存其授予的`GameplayAbilities`。任何时候你计划遍历`ActivatableAbilities.Items`时，确保在循环上方添加`ABILITYLIST_SCOPE_LOCK();`来锁定列表以防止变化（由于移除能力）。范围内的每个`ABILITYLIST_SCOPE_LOCK();`都会增加`AbilityScopeLockCount`，然后在超出范围时递减。不要在`ABILITYLIST_SCOPE_LOCK();`的范围内尝试移除能力（清除能力函数在内部检查`AbilityScopeLockCount`以防止在列表被锁定时移除能力）。

## 4.1.1 Replication Mode （复制模式）

The `ASC` defines three different replication modes for replicating `GameplayEffects`, `GameplayTags`, and `GameplayCues` - `Full`, `Mixed`, and `Minimal`. `Attributes` are replicated by their `AttributeSet`.

`ASC`定义了三种不同的复制模式来复制`GameplayEffects`、`GameplayTags`和`GameplayCues` - `Full`、`Mixed`和`Minimal`。`Attributes`由其`AttributeSet`复制。

| Replication Mode   | When to Use                             | Description                                                                                                                    |
| ------------------ | --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `Full`             | Single Player（单人游戏）                           | Every `GameplayEffect` is replicated to every client. （每个`GameplayEffect`都复制到每个客户端）                                                                          |
| `Mixed`            | Multiplayer, player controlled `Actors` （多人游戏，玩家控制的`Actor`） | `GameplayEffects` are only replicated to the owning client. Only `GameplayTags` and `GameplayCues` are replicated to everyone. （`GameplayEffects`只复制到拥有者客户端。只有`GameplayTags`和`GameplayCues`复制给所有人） |
| `Minimal`          | Multiplayer, AI controlled `Actors`     (多人游戏，AI控制的`Actor`) | `GameplayEffects` are never replicated to anyone. Only `GameplayTags` and `GameplayCues` are replicated to everyone. （`GameplayEffects`从不复制给任何人。只有`GameplayTags`和`GameplayCues`复制给所有人）           |

**Note:** `Mixed` replication mode expects the `OwnerActor's` `Owner` to be the `Controller`. `PlayerState's` `Owner` is the `Controller` by default but `Character's` is not. If using `Mixed` replication mode with the `OwnerActor` not the `PlayerState`, then you need to call `SetOwner()` on the `OwnerActor` with a valid `Controller`.

**注意：**`Mixed`复制模式期望`OwnerActor`的`Owner`是`Controller`。`PlayerState`的`Owner`默认是`Controller`，但`Character`的不是。如果在`OwnerActor`不是`PlayerState`的情况下使用`Mixed`复制模式，那么你需要在`OwnerActor`上调用`SetOwner()`并传入有效的`Controller`。

Starting with 4.24, `PossessedBy()` now sets the owner of the `Pawn` to the new `Controller`.

从4.24开始，`PossessedBy()`现在将`Pawn`的所有者设置为新的`Controller`。

## 4.1.2 Setup and Initialization （设置和初始化）

`ASCs` are typically constructed in the `OwnerActor's` constructor and explicitly marked replicated. **This must be done in C++**.

`ASC`通常在`OwnerActor`的构造函数中构造并明确标记为复制。**这必须在C++中完成**。

```c++
AGDPlayerState::AGDPlayerState()
{
	// Create ability system component, and set it to be explicitly replicated
	AbilitySystemComponent = CreateDefaultSubobject<UGDAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	AbilitySystemComponent->SetIsReplicated(true);
	//...
}
```

The `ASC` needs to be initialized with its `OwnerActor` and `AvatarActor` on both the server and the client. You want to initialize after the `Pawn's` `Controller` has been set (after possession). Single player games only need to worry about the server path.

`ASC`需要在服务器和客户端上都用其`OwnerActor`和`AvatarActor`进行初始化。你希望在`Pawn`的`Controller`被设置后（在拥有后）进行初始化。单人游戏只需要担心服务器路径。

For player controlled characters where the `ASC` lives on the `Pawn`, I typically initialize on the server in the `Pawn's` `PossessedBy()` function and initialize on the client in the `PlayerController's` `AcknowledgePossession()` function.

对于`ASC`存在于`Pawn`上的玩家控制的`Character`，在服务器上，我通常在`Pawn`的`PossessedBy()`函数中初始化，在客户端上，在`PlayerController`的`AcknowledgePossession()`函数中初始化。

```c++
void APACharacterBase::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->InitAbilityActorInfo(this, this);
	}

	// ASC MixedMode replication requires that the ASC Owner's Owner be the Controller.
	SetOwner(NewController);
}
```

```c++
void APAPlayerControllerBase::AcknowledgePossession(APawn* P)
{
	Super::AcknowledgePossession(P);

	APACharacterBase* CharacterBase = Cast<APACharacterBase>(P);
	if (CharacterBase)
	{
		CharacterBase->GetAbilitySystemComponent()->InitAbilityActorInfo(CharacterBase, CharacterBase);
	}

	//...
}
```

For player controlled characters where the `ASC` lives on the `PlayerState`, I typically initialize the server in the `Pawn's` `PossessedBy()` function and initialize on the client in the `Pawn's` `OnRep_PlayerState()` function. This ensures that the `PlayerState` exists on the client.

对于`ASC`存在于`PlayerState`上的玩家控制的`Character`，在服务器上，我通常在`Pawn`的`PossessedBy()`函数中初始化，在客户端上，在`Pawn`的`OnRep_PlayerState()`函数中初始化。这确保了`PlayerState`存在于客户端。

```c++
// Server only
void AGDHeroCharacter::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// Set the ASC on the Server. Clients do this in OnRep_PlayerState()
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// AI won't have PlayerControllers so we can init again here just to be sure. No harm in initing twice for heroes that have PlayerControllers.
		PS->GetAbilitySystemComponent()->InitAbilityActorInfo(PS, this);
	}
	
	//...
}
```

```c++
// Client only
void AGDHeroCharacter::OnRep_PlayerState()
{
	Super::OnRep_PlayerState();

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// Set the ASC for clients. Server does this in PossessedBy.
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// Init ASC Actor Info for clients. Server will init its ASC when it possesses a new Actor.
		AbilitySystemComponent->InitAbilityActorInfo(PS, this);
	}

	// ...
}
```

If you get the error message `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!` then you did not initialize your `ASC` on the client.

如果你收到错误消息`LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`，那么你没有在客户端上初始化你的`ASC`。

**[⬆ Back to Top](../README.md#table-of-contents)**
