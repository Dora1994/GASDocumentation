# 4.9 Ability System Globals （能力系统全局）

The [`AbilitySystemGlobals`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemGlobals/index.html) class holds global information about GAS. Most of the variables can be set from the `DefaultGame.ini`. Generally you won't have to interact with this class, but you should be aware of its existence. If you need to subclass things like the [GameplayCueManager](04-8-gameplay-cues.md) or the [GameplayEffectContext](04-5-gameplay-effects.md), you have to do that through the `AbilitySystemGlobals`.

[`AbilitySystemGlobals`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemGlobals/index.html)类保存有关GAS的全局信息。大多数变量可以从`DefaultGame.ini`设置。通常你不需要与这个类交互，但你应该知道它的存在。如果你需要子类化如[GameplayCueManager](04-8-gameplay-cues.md)或[GameplayEffectContext](04-5-gameplay-effects.md)之类的东西，你必须通过`AbilitySystemGlobals`来做。

To subclass `AbilitySystemGlobals`, set the class name in the `DefaultGame.ini`:

要子类化`AbilitySystemGlobals`，在`DefaultGame.ini`中设置类名：

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
AbilitySystemGlobalsClassName="/Script/ParagonAssets.PAAbilitySystemGlobals"
```

## 4.9.1 InitGlobalData()

Between UE 4.24 and 5.2, it is necessary to call `UAbilitySystemGlobals::Get().InitGlobalData()` to use [TargetData](04-11-targeting.md), otherwise you will get errors related to `ScriptStructCache` and clients will be disconnected from the server. This function only needs to be called once in a project. Fortnite calls it from `UAssetManager::StartInitialLoading()` and Paragon called it from `UEngine::Init()`. I find that putting it in `UAssetManager::StartInitialLoading()` is a good place as shown in the Sample Project. I would consider this boilerplate code that you should copy into your project to avoid issues with `TargetData`. Starting in 5.3 it is called automatically.

在UE 4.24和5.2之间，需要调用`UAbilitySystemGlobals::Get().InitGlobalData()`来使用[TargetData](04-11-targeting.md)，否则你会得到与`ScriptStructCache`相关的错误，客户端将与服务器断开连接。这个函数在项目中只需要调用一次。Fortnite从`UAssetManager::StartInitialLoading()`调用它，Paragon从`UEngine::Init()`调用它。我发现将它放在`UAssetManager::StartInitialLoading()`中是一个好地方，如示例项目所示。我认为这是你应该复制到项目中的样板代码，以避免`TargetData`的问题。从5.3开始它会自动调用。

If you run into a crash while using the `AbilitySystemGlobals` `GlobalAttributeSetDefaultsTableNames`, you may need to call `UAbilitySystemGlobals::Get().InitGlobalData()` later like Fortnite in the `AssetManager` or in the `GameInstance`.

如果在使用`AbilitySystemGlobals` `GlobalAttributeSetDefaultsTableNames`时遇到崩溃，你可能需要像Fortnite那样在`AssetManager`或`GameInstance`中稍后调用`UAbilitySystemGlobals::Get().InitGlobalData()`。

**[⬆ Back to Top](../README.md#table-of-contents)**
