# 4.11 Targeting

## 4.11.1 Target Data

[`FGameplayAbilityTargetData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetData/index.html) is a generic structure for targeting data meant to be passed across the network. `TargetData` will typically hold `AActor`/`UObject` references, `FHitResults`, and other generic location/direction/origin information. However, you can subclass it to put essentially anything that you want inside of them as a simple means to [pass data between the client and server in `GameplayAbilities`](04-6-gameplay-abilities.md). The base struct `FGameplayAbilityTargetData` is not meant to be used directly but instead subclassed. `GAS` comes with a few subclassed `FGameplayAbilityTargetData` structs out of the box located in `GameplayAbilityTargetTypes.h`.

`TargetData` is typically produced by [Target Actors](#concepts-targeting-actors) or **created manually** and consumed by [AbilityTasks](04-7-ability-tasks.md) and [GameplayEffects](04-5-gameplay-effects.md) via the [EffectContext](04-5-gameplay-effects.md). As a result of being in the `EffectContext`, [Executions](04-5-gameplay-effects.md), [MMCs](04-5-gameplay-effects.md), [GameplayCues](04-8-gameplay-cues.md), and the functions on the backend of the [AttributeSet](04-4-attribute-set.md) can access the `TargetData`.

We don't typically pass around the `FGameplayAbilityTargetData` directly, instead we use a [`FGameplayAbilityTargetDataHandle`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetDataHandle/index.html) which has an internal TArray of pointers to `FGameplayAbilityTargetData`. This intermediate struct provides support for polymorphism of the `TargetData`.

An example of inheritting from `FGameplayAbilityTargetData`:
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
    virtual UScriptStruct* GetScriptStruct() const override
    {
    	return FGameplayAbilityTargetData_CustomData::StaticStruct();
    }

	// This is required for all child structs of FGameplayAbilityTargetData
    bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
    {
	    // The engine already defined NetSerialize for FName & FPredictionKey, thanks Epic!
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
    };
};
```

**[⬆ Back to Top](../README.md#table-of-contents)**
