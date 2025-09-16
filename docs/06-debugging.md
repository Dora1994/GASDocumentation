# 6. Debugging GAS

## 6. 调试GAS

## 6.1 showdebug abilitysystem

## 6.1 showdebug abilitysystem

Type `showdebug abilitysystem` in the in-game console. This feature is split into three "pages". All three pages will show the `GameplayTags` that you currently have. Type `AbilitySystem.Debug.NextCategory` into the console to cycle between the pages.

在游戏内控制台中输入`showdebug abilitysystem`。此功能分为三个"页面"。所有三个页面都会显示你当前拥有的`游戏标签`。在控制台中输入`AbilitySystem.Debug.NextCategory`来在页面之间循环。

The first page shows the `CurrentValue` of all of your `Attributes`:
![First Page of showdebug abilitysystem](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage1.png)

第一页显示你所有`属性`的`当前值`：
![showdebug abilitysystem的第一页](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage1.png)

The second page shows all of the `Duration` and `Infinite` `GameplayEffects` on you, their number of stacks, what `GameplayTags` they give, and what `Modifiers` they give.
![Second Page of showdebug abilitysystem](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage2.png)

第二页显示你身上所有的`持续`和`无限` `游戏效果`，它们的堆叠数量，它们给予的`游戏标签`，以及它们给予的`修饰符`。
![showdebug abilitysystem的第二页](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage2.png)

The third page shows all of the `GameplayAbilities` that have been granted to you, whether they are currently running, whether they are blocked from activating, and the status of currently running `AbilityTasks`.
![Third Page of showdebug abilitysystem](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage3.png)

第三页显示已授予你的所有`游戏能力`，它们是否正在运行，是否被阻止激活，以及当前运行的`能力任务`的状态。
![showdebug abilitysystem的第三页](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage3.png)

To cycle between targets (denoted by a green rectangular prism around the Actor), use the `PageUp` key or `NextDebugTarget` console command to go to the next target and the `PageDown` key or `PreviousDebugTarget` console command to go to the previous target.

要在目标之间循环（由Actor周围的绿色矩形棱镜表示），使用`PageUp`键或`NextDebugTarget`控制台命令转到下一个目标，使用`PageDown`键或`PreviousDebugTarget`控制台命令转到上一个目标。

**Note:** In order for the ability system information to update based on the currently selected debug Actor, you need to set `bUseDebugTargetFromHud=true` in the `AbilitySystemGlobals` like so in the `DefaultGame.ini`:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
bUseDebugTargetFromHud=true
```

**注意：**为了让能力系统信息根据当前选择的调试Actor更新，你需要在`AbilitySystemGlobals`中设置`bUseDebugTargetFromHud=true`，在`DefaultGame.ini`中如下：
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
bUseDebugTargetFromHud=true
```

**Note:** For `showdebug abilitysystem` to work an actual HUD class must be selected in the GameMode. Otherwise the command is not found and "Unknown Command" is returned.

**注意：**为了让`showdebug abilitysystem`工作，必须在GameMode中选择实际的HUD类。否则找不到命令并返回"Unknown Command"。

## 6.2 Gameplay Debugger

## 6.2 游戏调试器

GAS adds functionality to the Gameplay Debugger. Access the Gameplay Debugger with the Apostrophe (') key. Enable the Abilities category by pressing 3 on your numpad. The category may be different depending on what plugins you have. If your keyboard doesn't have a numpad like a laptop, then you can change the keybindings in the project settings.

GAS为游戏调试器添加了功能。使用撇号(')键访问游戏调试器。通过按小键盘上的3启用能力类别。类别可能因你拥有的插件而异。如果你的键盘没有小键盘（如笔记本电脑），那么你可以在项目设置中更改键绑定。

Use the Gameplay Debugger when you want to see the `GameplayTags`, `GameplayEffects`, and `GameplayAbilities` on **other** `Characters`. Unfortunately it does not show the `CurrentValue` of the target's `Attributes`. It will target whatever `Character` is in the center of your screen. You can change targets by selecting them in the World Outliner in the Editor or by looking at a different `Character` and press Apostrophe (') again. The currently inspected `Character` has the largest red circle above it.

当你想查看**其他**`角色`的`游戏标签`、`游戏效果`和`游戏能力`时使用游戏调试器。不幸的是，它不显示目标`属性`的`当前值`。它会瞄准你屏幕中心的任何`角色`。你可以通过在编辑器的世界大纲中选择它们或通过看向不同的`角色`并再次按撇号(')来更改目标。当前检查的`角色`上方有最大的红色圆圈。

![Gameplay Debugger](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaydebugger.png)

## 6.3 GAS Logging

## 6.3 GAS日志记录

The GAS source code contains a lot of logging statements produced at varying verbosity levels. You will most likely see these as `ABILITY_LOG()` statements. The default verbosity level is `Display`. Anything higher will not be displayed in the console by default.

GAS源代码包含大量以不同详细级别产生的日志语句。你最可能看到这些作为`ABILITY_LOG()`语句。默认详细级别是`Display`。默认情况下，任何更高的级别都不会在控制台中显示。

To change the verbosity level of a log category, type into your console:

要更改日志类别的详细级别，在控制台中输入：

```
log [category] [verbosity]
```

For example, to turn on `ABILITY_LOG()` statements, you would type into your console:
```
log LogAbilitySystem VeryVerbose
```

例如，要开启`ABILITY_LOG()`语句，你会在控制台中输入：
```
log LogAbilitySystem VeryVerbose
```

To reset it back to default, type:
```
log LogAbilitySystem Display
```

要重置回默认值，输入：
```
log LogAbilitySystem Display
```

**[⬆ Back to Top](../README.md#table-of-contents)**
