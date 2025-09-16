# 4.9 Ability System Globals

The [`AbilitySystemGlobals`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemGlobals/index.html) class holds global information about GAS. Most of the variables can be set from the `DefaultGame.ini`. Generally you won't have to interact with this class, but you should be aware of its existence. If you need to subclass things like the [GameplayCueManager](04-8-gameplay-cues.md) or the [GameplayEffectContext](04-5-gameplay-effects.md), you have to do that through the `AbilitySystemGlobals`.

To subclass `AbilitySystemGlobals`, set the class name in the `DefaultGame.ini`:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
AbilitySystemGlobalsClassName="/Script/ParagonAssets.PAAbilitySystemGlobals"
```

## 4.9.1 InitGlobalData()

Between UE 4.24 and 5.2, it is necessary to call `UAbilitySystemGlobals::Get().InitGlobalData()` to use [TargetData](04-11-targeting.md), otherwise you will get errors related to `ScriptStructCache` and clients will be disconnected from the server. This function only needs to be called once in a project. Fortnite calls it from `UAssetManager::StartInitialLoading()` and Paragon called it from `UEngine::Init()`. I find that putting it in `UAssetManager::StartInitialLoading()` is a good place as shown in the Sample Project. I would consider this boilerplate code that you should copy into your project to avoid issues with `TargetData`. Starting in 5.3 it is called automatically.

If you run into a crash while using the `AbilitySystemGlobals` `GlobalAttributeSetDefaultsTableNames`, you may need to call `UAbilitySystemGlobals::Get().InitGlobalData()` later like Fortnite in the `AssetManager` or in the `GameInstance`.

**[⬆ Back to Top](../README.md#table-of-contents)**
