# 4.7 Ability Tasks （能力任务）

## 4.7.1 Ability Task Definition （能力任务定义）

`GameplayAbilities` only execute in one frame. This does not allow for much flexibility on its own. To do actions that happen over time or require responding to delegates fired at some point later in time we use latent actions called `AbilityTasks`.

`GameplayAbilities`只在一帧中执行。这本身不允许太多灵活性。要执行随时间发生或需要响应在稍后某个时间点触发的委托的动作，我们使用称为`AbilityTasks`的潜在动作。

GAS comes with many `AbilityTasks` out of the box (GAS提供了很多开箱即用的`AbilityTasks`):
* Tasks for moving Characters with `RootMotionSource` (使用`RootMotionSource`移动角色的任务)
* A task for playing animation montages (播放动画蒙太奇的任务)
* Tasks for responding to `Attribute` changes (响应`Attribute`变化的任务)
* Tasks for responding to `GameplayEffect` changes (响应`GameplayEffect`变化的任务)
* Tasks for responding to player input (响应玩家输入的任务)
* and more (以及更多)

The `UAbilityTask` constructor enforces a hardcoded game-wide maximum of 1000 concurrent `AbilityTasks` running at the same time. Keep this in mind when designing `GameplayAbilities` for games that can have hundreds of characters in the world at the same time like RTS games.

`UAbilityTask`构造函数中强制的以硬编码的方式设置并发`AbilityTasks`的最大值：1000个。在为可以同时在世界中拥有数百个角色的游戏（如RTS游戏）设计`GameplayAbilities`时，请记住这一点。

## 4.7.2 Custom Ability Tasks （自定义能力任务）

Often you will be creating your own custom `AbilityTasks` (in C++). The Sample Project comes with two custom `AbilityTasks`:
1. `PlayMontageAndWaitForEvent` is a combination of the default `PlayMontageAndWait` and `WaitGameplayEvent` `AbilityTasks`. This allows animation montages to send gameplay events from `AnimNotifies` back to the `GameplayAbility` that started them. Use this to trigger actions at specific times during animation montages.
1. `WaitReceiveDamage` listens for the `OwnerActor` to receive damage. The passive armor stacks `GameplayAbility` removes a stack of armor when the hero receives an instance of damage.

通常你会创建自己的自定义`AbilityTasks`（在C++中）。示例项目带有两个自定义`AbilityTasks`：
1. `PlayMontageAndWaitForEvent`是默认`PlayMontageAndWait`和`WaitGameplayEvent` `AbilityTasks`的组合。这允许动画蒙太奇从`AnimNotifies`发送游戏事件回到启动它们的`GameplayAbility`。使用这个在动画蒙太奇的特定时间触发动作。
1. `WaitReceiveDamage`监听`OwnerActor`接收伤害。被动护甲堆叠`GameplayAbility`在英雄接收伤害实例时移除一层护甲。

`AbilityTasks` are composed of （`AbilityTasks`由以下组成）:
* A static function that creates new instances of the `AbilityTask` (创建`AbilityTask`新实例的静态函数)
* Delegates that are broadcasted on when the `AbilityTask` completes its purpose (当`AbilityTask`完成其目的时广播的委托) 
* An `Activate()` function to start its main job, bind to external delegates, etc. (启动其主要工作、绑定到外部委托等的`Activate()`函数)
* An `OnDestroy()` function for cleanup, including external delegates that it bound to (用于清理的`OnDestroy()`函数，包括它绑定的外部委托)
* Callback functions for any external delegates that it bound to (它绑定的任何外部委托的回调函数)
* Member variables and any internal helper functions (成员变量和任何内部辅助函数)

**Note:** `AbilityTasks` can only declare one type of output delegate. All of your output delegates must be of this type, regardless if they use the parameters or not. Pass default values for unused delegate parameters.

**注意：**`AbilityTasks`只能声明一种类型的输出委托。你所有的输出委托必须是这种类型，无论它们是否使用参数。为未使用的委托参数传递默认值。

`AbilityTasks` only run on the Client or Server that is running the owning `GameplayAbility`; however, `AbilityTasks` can be set to run on simulated clients by setting `bSimulatedTask = true;` in the `AbilityTask` constructor, overriding `virtual void InitSimulatedTask(UGameplayTasksComponent& InGameplayTasksComponent);`, and setting any member variables to be replicated. This is only useful in rare situations like movement `AbilityTasks` where you don't want to replicate every movement change but instead simulate the entire movement `AbilityTask`. All of the `RootMotionSource` `AbilityTasks` do this. See `AbilityTask_MoveToLocation.h/.cpp` as an example.

`AbilityTasks`只在运行拥有`GameplayAbility`的客户端或服务器上运行；但是，`AbilityTasks`可以通过在`AbilityTask`构造函数中设置`bSimulatedTask = true;`、重写`virtual void InitSimulatedTask(UGameplayTasksComponent& InGameplayTasksComponent);`，并设置任何成员变量为复制来设置在模拟客户端上运行。这只在罕见情况下有用，如移动`AbilityTasks`，你不想复制每个移动变化，而是模拟整个移动`AbilityTask`。所有`RootMotionSource` `AbilityTasks`都这样做。参见`AbilityTask_MoveToLocation.h/.cpp`作为示例。

`AbilityTasks` can `Tick` if you set `bTickingTask = true;` in the `AbilityTask` constructor and override `virtual void TickTask(float DeltaTime);`. This is useful when you need to lerp values smoothly across frames. See `AbilityTask_MoveToLocation.h/.cpp` as an example.

如果你在`AbilityTask`构造函数中设置`bTickingTask = true;`并重写`virtual void TickTask(float DeltaTime);`，`AbilityTasks`可以`Tick`。当你需要跨帧平滑插值时这很有用。参见`AbilityTask_MoveToLocation.h/.cpp`作为示例。

## 4.7.3 Using Ability Tasks （使用能力任务）

To create and activate an `AbilityTask` in C++ (From `GDGA_FireGun.cpp`):

在C++中创建和激活`AbilityTask`（来自`GDGA_FireGun.cpp`）：

```c++
UGDAT_PlayMontageAndWaitForEvent* Task = UGDAT_PlayMontageAndWaitForEvent::PlayMontageAndWaitForEvent(this, NAME_None, MontageToPlay, FGameplayTagContainer(), 1.0f, NAME_None, false, 1.0f);
Task->OnBlendOut.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnCompleted.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnInterrupted.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->OnCancelled.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->EventReceived.AddDynamic(this, &UGDGA_FireGun::EventReceived);
Task->ReadyForActivation();
```

## 4.7.4 Root Motion Source Ability Tasks （根运动源能力任务）

`AbilityTasks` that use `RootMotionSource` for movement are special. They can be locally predicted and will automatically handle rollback if the prediction fails. The Sample Project uses `AbilityTask_MoveToLocation` for the dash ability.

使用`RootMotionSource`进行移动的`AbilityTasks`是特殊的。它们可以被本地预测，如果预测失败会自动处理回滚。示例项目使用`AbilityTask_MoveToLocation`进行冲刺能力。

**[⬆ Back to Top](../README.md#table-of-contents)**
