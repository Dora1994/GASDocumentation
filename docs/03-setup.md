# 3. Setting Up a Project Using GAS

## 3. 使用GAS设置项目

Basic steps to set up a project using GAS:
1. Enable GameplayAbilitySystem plugin in the Editor
1. Edit `YourProjectName.Build.cs` to add `"GameplayAbilities", "GameplayTags", "GameplayTasks"` to your `PrivateDependencyModuleNames`
1. Refresh/Regenerate your Visual Studio project files
1. Starting with 4.24 up to 5.2, it is mandatory to call `UAbilitySystemGlobals::Get().InitGlobalData()` to use [`TargetData`](../04-concepts/04-11-targeting.md). The Sample Project does this in `UAssetManager::StartInitialLoading()`. This is called automatically starting in 5.3. See [`InitGlobalData()`](../04-concepts/04-9-ability-system-globals.md) for more information.

使用GAS设置项目的基本步骤：
1. 在编辑器中启用GameplayAbilitySystem插件
1. 编辑`YourProjectName.Build.cs`，将`"GameplayAbilities", "GameplayTags", "GameplayTasks"`添加到你的`PrivateDependencyModuleNames`中
1. 刷新/重新生成你的Visual Studio项目文件
1. 从4.24到5.2版本，必须调用`UAbilitySystemGlobals::Get().InitGlobalData()`来使用[`目标数据`](../04-concepts/04-11-targeting.md)。示例项目在`UAssetManager::StartInitialLoading()`中这样做。从5.3开始会自动调用。更多信息请参见[`InitGlobalData()`](../04-concepts/04-9-ability-system-globals.md)。

That's all that you have to do to enable GAS. From here, add an [`ASC`](../04-concepts/04-1-ability-system-component.md) and [`AttributeSet`](../04-concepts/04-4-attribute-set.md) to your `Character` or `PlayerState` and start making [`GameplayAbilities`](../04-concepts/04-6-gameplay-abilities.md) and [`GameplayEffects`](../04-concepts/04-5-gameplay-effects.md)!

这就是启用GAS所需要做的全部工作。从这里开始，向你的`Character`或`PlayerState`添加一个[`ASC`](../04-concepts/04-1-ability-system-component.md)和[`属性集`](../04-concepts/04-4-attribute-set.md)，然后开始制作[`游戏能力`](../04-concepts/04-6-gameplay-abilities.md)和[`游戏效果`](../04-concepts/04-5-gameplay-effects.md)！

**[⬆ Back to Top](../README.md#table-of-contents)**
