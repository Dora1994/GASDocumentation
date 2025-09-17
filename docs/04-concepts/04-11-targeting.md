# 4.11 Targeting （目标选择）

## 4.11.1 Target Data （目标数据）

[`FGameplayAbilityTargetData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetData/index.html) is a generic structure for targeting data meant to be passed across the network. `TargetData` will typically hold `AActor`/`UObject` references, `FHitResults`, and other generic location/direction/origin information. However, you can subclass it to put essentially anything that you want inside of them as a simple means to [pass data between the client and server in `GameplayAbilities`](04-6-gameplay-abilities.md). The base struct `FGameplayAbilityTargetData` is not meant to be used directly but instead subclassed. `GAS` comes with a few subclassed `FGameplayAbilityTargetData` structs out of the box located in `GameplayAbilityTargetTypes.h`.

[`FGameplayAbilityTargetData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetData/index.html)是用于目标数据的通用结构，旨在跨网络传递。`TargetData`通常会保存`AActor`/`UObject`引用、`FHitResults`和其他通用位置/方向/原点信息。但是，你可以将其子类化以将你想要的任何内容放入其中，作为在[`GameplayAbilities`中在客户端和服务器之间传递数据](04-6-gameplay-abilities.md)的简单方法。基础结构`FGameplayAbilityTargetData`不应该直接使用，而是应该被子类化。`GAS`开箱即用地提供了一些子类化的`FGameplayAbilityTargetData`结构，位于`GameplayAbilityTargetTypes.h`中。

`TargetData` is typically produced by [Target Actors](#concepts-targeting-actors) or **created manually** and consumed by [AbilityTasks](04-7-ability-tasks.md) and [GameplayEffects](04-5-gameplay-effects.md) via the [EffectContext](04-5-gameplay-effects.md). As a result of being in the `EffectContext`, [Executions](04-5-gameplay-effects.md), [MMCs](04-5-gameplay-effects.md), [GameplayCues](04-8-gameplay-cues.md), and the functions on the backend of the [AttributeSet](04-4-attribute-set.md) can access the `TargetData`.

`TargetData`通常由[Target Actors](#concepts-targeting-actors)产生或**手动创建**，并通过[EffectContext](04-5-gameplay-effects.md)被[AbilityTasks](04-7-ability-tasks.md)和[GameplayEffects](04-5-gameplay-effects.md)消费。由于存在于`EffectContext`中，[Executions](04-5-gameplay-effects.md)、[MMCs](04-5-gameplay-effects.md)、[GameplayCues](04-8-gameplay-cues.md)和[AttributeSet](04-4-attribute-set.md)后端的函数都可以访问`TargetData`。

We don't typically pass around the `FGameplayAbilityTargetData` directly, instead we use a [`FGameplayAbilityTargetDataHandle`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetDataHandle/index.html) which has an internal TArray of pointers to `FGameplayAbilityTargetData`. This intermediate struct provides support for polymorphism of the `TargetData`.

我们通常不直接传递`FGameplayAbilityTargetData`，而是使用[`FGameplayAbilityTargetDataHandle`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetDataHandle/index.html)，它有一个指向`FGameplayAbilityTargetData`的指针的内部TArray。这个中间结构为`TargetData`提供多态性支持。

An example of inheritting from `FGameplayAbilityTargetData` (继承`FGameplayAbilityTargetData`的示例):

```c++
USTRUCT(BlueprintType)
struct MYGAME_API FGameplayAbilityTargetData_CustomData : public FGameplayAbilityTargetData
{
    GENERATED_BODY()
public:

    FGameplayAbilityTargetData_CustomData()
    { }

    UPROPERTY()
    FName CoolName = NAME_None;

    UPROPERTY()
    FPredictionKey MyCoolPredictionKey;

    // This is required for all child structs of FGameplayAbilityTargetData
    // 这对于FGameplayAbilityTargetData的所有子结构都是必需的
    virtual UScriptStruct* GetScriptStruct() const override
    {
    	return FGameplayAbilityTargetData_CustomData::StaticStruct();
    }

	// This is required for all child structs of FGameplayAbilityTargetData
	// 这对于FGameplayAbilityTargetData的所有子结构都是必需的
    bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
    {
	    // The engine already defined NetSerialize for FName & FPredictionKey, thanks Epic!
	    // 引擎已经为FName和FPredictionKey定义了NetSerialize，感谢Epic！
        CoolName.NetSerialize(Ar, Map, bOutSuccess);
        MyCoolPredictionKey.NetSerialize(Ar, Map, bOutSuccess);
        bOutSuccess = true;
        return true;
    }
}

template<>
struct TStructOpsTypeTraits<FGameplayAbilityTargetData_CustomData> : public TStructOpsTypeTraitsBase2<FGameplayAbilityTargetData_CustomData>
{
	enum
	{
        WithNetSerializer = true // This is REQUIRED for FGameplayAbilityTargetDataHandle net serialization to work
                                // 这对于FGameplayAbilityTargetDataHandle网络序列化工作是必需的
    };
};
```

**[⬆ Back to Top](../README.md#table-of-contents)**
