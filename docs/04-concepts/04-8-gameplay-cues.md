# 4.8 Gameplay Cues （游戏提示）

## 4.8.1 Gameplay Cue Definition （游戏提示定义）

`GameplayCues` (`GC`) execute non-gameplay related things like sound effects, particle effects, camera shakes, etc. `GameplayCues` are typically replicated (unless explicitly `Executed`, `Added`, or `Removed` locally) and predicted.

`GameplayCues` (`GC`) 执行与游戏玩法无关的事物，如音效、粒子效果、摄像机震动等。`GameplayCues`通常被复制（除非明确在本地`执行`、`添加`或`移除`）和预测。

We trigger `GameplayCues` by sending a corresponding `GameplayTag` with the **mandatory parent name of `GameplayCue.`** and an event type (`Execute`, `Add`, or `Remove`) to the `GameplayCueManager` via the `ASC`. `GameplayCueNotify` objects and other `Actors` that implement the `IGameplayCueInterface` can subscribe to these events based on the `GameplayCue's` `GameplayTag` (`GameplayCueTag`).

我们通过向`GameplayCueManager`发送带有**强制父名称`GameplayCue.`**的相应`GameplayTag`和事件类型（`Execute`、`Add`或`Remove`）来触发`GameplayCues`，通过`ASC`。`GameplayCueNotify`对象和实现`IGameplayCueInterface`的其他`Actors`可以基于`GameplayCue`的`GameplayTag`（`GameplayCueTag`）订阅这些事件。

**Note:** Just to reiterate, `GameplayCue` `GameplayTags` need to start with the parent `GameplayTag` of `GameplayCue`. So for example, a valid `GameplayCue` `GameplayTag` might be `GameplayCue.A.B.C`.

**注意：**重申一下，`GameplayCue` `GameplayTags`需要以`GameplayCue`的父`GameplayTag`开头。例如，一个有效的`GameplayCue` `GameplayTag`可能是`GameplayCue.A.B.C`。

There are two classes of `GameplayCueNotifies`, `Static` and `Actor`. They respond to different events and different types of `GameplayEffects` can trigger them. Override the corresponding event with your logic.

有两类`GameplayCueNotifies`，`Static`和`Actor`。它们响应不同的事件，不同类型的`GameplayEffects`可以触发它们。用你的逻辑重写相应的事件。

| `GameplayCue` Class                                                                                                                  | Event             | `GameplayEffect` Type    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`GameplayCueNotify_Static`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayCueNotify_Static/index.html) | `Execute`         | `Instant` or `Periodic`  | Static `GameplayCueNotifies` operate on the `ClassDefaultObject` (meaning no instances) and are perfect for one-off effects like hit impacts. （ 静态`GameplayCueNotifies`在`ClassDefaultObject`上操作（意味着没有实例），非常适合一次性效果，如击中冲击。）                                                                                                                                                                                                                                                                                                                                                                       |
| [`GameplayCueNotify_Actor`](https://docs.unrealengine.com/en-US/BlueprintAPI/GameplayCueNotify/index.html)                           | `Add` or `Remove` | `Duration` or `Infinite` | Actor `GameplayCueNotifies` spawn a new instance when `Added`. Because these are instanced, they can do actions over time until they are `Removed`. These are good for looping sounds and particle effects that will be removed when the backing `Duration` or `Infinite` `GameplayEffect` is removed or by manually calling remove. These also come with options to manage how many are allowed to be `Added` at the same time so that multiple applications of the same effect only start the sounds or particles once. （ Actor `GameplayCueNotifies`在`添加`时生成新实例。因为这些是实例化的，它们可以随时间执行动作直到被`移除`。这些适用于循环声音和粒子效果，当支持的`Duration`或`Infinite` `GameplayEffect`被移除或手动调用移除时将被移除。这些还带有选项来管理同时允许`添加`多少个，以便同一效果的多次应用只启动一次声音或粒子。） |

`GameplayCueNotifies` technically can respond to any of the events but this is typically how we use them.

`GameplayCueNotifies`技术上可以响应任何事件，但这通常是我们使用它们的方式。

**Note:** When using `GameplayCueNotify_Actor`, check `Auto Destroy on Remove` otherwise subsequent calls to `Add` that `GameplayCueTag` won't work.

**注意：**使用`GameplayCueNotify_Actor`时，检查`移除时自动销毁`，否则后续调用`添加`该`GameplayCueTag`将不起作用。

When using an `ASC` [Replication Mode](04-1-ability-system-component.md) other than `Full`, `Add` and `Remove` `GC` events will fire twice on Server players (listen server) - once for applying the `GE` and again from the "Minimal" `NetMultiCast` to the clients. However, `WhileActive` events will still only fire once. All events will only fire once on clients.

当使用除`Full`之外的`ASC` [复制模式](04-1-ability-system-component.md)时，`Add`和`Remove` `GC`事件将在服务器玩家（监听服务器）上触发两次 - 一次用于应用`GE`，再次从"Minimal"`NetMultiCast`到客户端。然而，`WhileActive`事件仍然只触发一次。所有事件在客户端上只触发一次。

The Sample Project includes a `GameplayCueNotify_Actor` for stun and sprint effects. It also has a `GameplayCueNotify_Static` for the FireGun's projectile impact. These `GCs` can be optimized further by [triggering them locally](#concepts-gc-local) instead of replicating them through a `GE`. I opted for showing the beginner way of using them in the Sample Project.

示例项目包含用于眩晕和冲刺效果的`GameplayCueNotify_Actor`。它还有用于FireGun弹道撞击的`GameplayCueNotify_Static`。这些`GCs`可以通过[本地触发](#concepts-gc-local)而不是通过`GE`复制来进一步优化。我选择在示例项目中展示使用它们的初学者方法。

## 4.8.2 Triggering Gameplay Cues （触发游戏提示）

From inside of a `GameplayEffect` when it is successfully applied (not blocked by tags or immunity), fill in the `GameplayTags` of all the `GameplayCues` that should be triggered.

当`GameplayEffect`成功应用时（未被标签或免疫阻止），从其内部填写所有应该被触发的`GameplayCues`的`GameplayTags`。

![GameplayCue Triggered from a GameplayEffect](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromge.png)

`UGameplayAbility` offers Blueprint nodes to `Execute`, `Add`, or `Remove` `GameplayCues`.

`UGameplayAbility`提供蓝图节点来`Execute`、`Add`或`Remove` `GameplayCues`。

![GameplayCue Triggered from a GameplayAbility](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromga.png)

In C++, you can call functions directly on the `ASC` (or expose them to Blueprint in your `ASC` subclass):

在C++中，你可以直接在`ASC`上调用函数（或在你的`ASC`子类中将它们暴露给蓝图）：

```c++
/** GameplayCues can also come on their own. These take an optional effect context to pass through hit result, etc */
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** Add a persistent gameplay cue */
void AddGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void AddGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** Remove a persistent gameplay cue */
void RemoveGameplayCue(const FGameplayTag GameplayCueTag);
	
/** Removes any GameplayCue added on its own, i.e. not as part of a GameplayEffect. */
void RemoveAllGameplayCues();
```

**[⬆ Back to Top](../README.md#table-of-contents)**
