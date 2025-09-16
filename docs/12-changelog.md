# 12. GAS Changelog

## 12. GAS更新日志

## 5.3

## 5.3

* Crash Fix: Fixed a crash when trying to apply Gameplay Cues after a seamless travel.
* Crash Fix: Fixed a crash caused by GlobalAbilityTaskCount when using Live Coding.
* Crash Fix: Fixed UAbilityTask::OnDestroy to not crash if called recursively for cases like UAbilityTask_StartAbilityState.
* Bug Fix: It is now safe to call `Super::ActivateAbility` in a child class. Previously, it would call `CommitAbility`.
* Bug Fix: Added support for properly replicating different types of FGameplayEffectContext.
* Bug Fix: FGameplayEffectContextHandle will now check if data is valid before retrieving "Actors".
* Bug Fix: Retain rotation for Gameplay Ability System Target Data LocationInfo.
* Bug Fix: Gameplay Ability System now stops searching for PC only if a valid PC is found.
* Bug Fix: Use existing GameplayCueParameters if it exists instead of default parameters object in RemoveGameplayCue_Internal.
* Bug Fix: GameplayAbilityWorldReticle now faces towards the source Actor instead of the TargetingActor.
* Bug Fix: Cache trigger event data if it was passed in with GiveAbilityAndActivateOnce and the ability list was locked.
* Bug Fix: Support has been added for the FInheritedGameplayTags to update its CombinedTags immediately rather than waiting until a Save.
* Bug Fix: Moved ShouldAbilityRespondToEvent from client-only code path to both server and client.
* Bug Fix: Fixed FAttributeSetInitterDiscreteLevels from not working in Cooked Builds due to Curve Simplification.
* Bug Fix: Set CurrentEventData in GameplayAbility.
* Bug Fix: Ensure MinimalReplicationTags are set up correctly before potentially executing callbacks.
* Bug Fix: Fixed ShouldAbilityRespondToEvent from not getting called on the instanced GameplayAbility.
* Bug Fix: Gameplay Cue Notify Actors executing on Child Actors no longer leak memory when gc.PendingKill is disabled.
* Bug Fix: Fixed an issue in GameplayCueManager where GameplayCueNotify_Actors could be 'lost' due to hash collisions.
* Bug Fix: WaitGameplayTagQuery will now respect its Query even if we have no Gameplay Tags on the Actor.
* Bug Fix: PostAttributeChange and AttributeValueChangeDelegates will now have the correct OldValue.
* Bug Fix: Fixed FGameplayTagQuery from not showing a proper Query Description if the struct was created by native code.
* Bug Fix: Ensure that the UAbilitySystemGlobals::InitGlobalData is called if the Ability System is in use. Previously if the user did not call it, the Gameplay Ability System did not function correctly.
* Bug Fix: Fixed issue when linking/unlinking anim layers from UGameplayAbility::EndAbility.
* Bug Fix: Updated Ability System Component function to check the Spec's ability pointer before use.
* New: Added a GameplayTagQuery field to FGameplayTagRequirements to enable more complex requirements to be specified.
* New: Introduced FGameplayEffectQuery::SourceAggregateTagQuery to augment SourceTagQuery.
* New: Extended the functonality to execute and cancel Gameplay Abilities & Gameplay Effects from a console command.
* New: Added the ability to perform an "Audit" on Gameplay Ability Blueprints that will show information on how they're developed and intended to be used.
* Change: OnAvatarSet is now called on the primary instance instead of the CDO for instanced per Actor Gameplay Abilities.
* Change: Allow both Activate Ability and Activate Ability From Event in the same Gameplay Ability Graph.
* Change: AnimTask_PlayMontageAndWait now has a toggle to allow Completed and Interrupted after a BlendOut event.
* Change: ModMagnitudeCalc wrapper functions have been declared const.
* Change: FGameplayTagQuery::Matches now returns false for empty queries.
* Change: Updated FGameplayAttribute::PostSerialize to mark the contained attribute as a searchable name.
* Change: Updated GetAbilitySystemComponent to default parameter to Self.
* Change: Marked functions as virtual in AbilityTask_WaitTargetData.
* Change: Removed unused function FGameplayAbilityTargetData::AddTargetDataToGameplayCueParameters.
* Change: Removed vestigial GameplayAbility::SetMovementSyncPoint.
* Change: Removed unused replication flag from Gameplay tasks & Ability system components.
* Change: Moved some gameplay effect functionality into optional components. All existing content will automatically update to use components during PostCDOCompiled, if necessary.

* 崩溃修复：修复了在无缝旅行后尝试应用游戏提示时的崩溃。
* 崩溃修复：修复了使用实时编码时由GlobalAbilityTaskCount引起的崩溃。
* 崩溃修复：修复了UAbilityTask::OnDestroy在递归调用时不会崩溃的问题，如UAbilityTask_StartAbilityState的情况。
* 错误修复：现在在子类中调用`Super::ActivateAbility`是安全的。以前，它会调用`CommitAbility`。
* 错误修复：添加了对正确复制不同类型的FGameplayEffectContext的支持。
* 错误修复：FGameplayEffectContextHandle现在会在检索"Actors"之前检查数据是否有效。
* 错误修复：为游戏能力系统目标数据LocationInfo保留旋转。
* 错误修复：游戏能力系统现在只有在找到有效PC时才停止搜索PC。
* 错误修复：在RemoveGameplayCue_Internal中使用现有的GameplayCueParameters（如果存在）而不是默认参数对象。
* 错误修复：GameplayAbilityWorldReticle现在面向源Actor而不是TargetingActor。
* 错误修复：如果触发事件数据与GiveAbilityAndActivateOnce一起传入且能力列表被锁定，则缓存触发事件数据。
* 错误修复：添加了对FInheritedGameplayTags立即更新其CombinedTags而不是等到保存的支持。
* 错误修复：将ShouldAbilityRespondToEvent从仅客户端代码路径移动到服务器和客户端。
* 错误修复：修复了FAttributeSetInitterDiscreteLevels由于曲线简化在Cooked Builds中不工作的问题。
* 错误修复：在GameplayAbility中设置CurrentEventData。
* 错误修复：确保MinimalReplicationTags在可能执行回调之前正确设置。
* 错误修复：修复了ShouldAbilityRespondToEvent在实例化GameplayAbility上不被调用的问题。
* 错误修复：当gc.PendingKill被禁用时，在子Actor上执行的游戏提示通知Actor不再泄漏内存。
* 错误修复：修复了GameplayCueManager中由于哈希冲突导致GameplayCueNotify_Actors可能"丢失"的问题。
* 错误修复：WaitGameplayTagQuery现在会尊重其查询，即使Actor上没有游戏标签。
* 错误修复：PostAttributeChange和AttributeValueChangeDelegates现在将有正确的OldValue。
* 错误修复：修复了如果结构体由原生代码创建时FGameplayTagQuery不显示正确查询描述的问题。
* 错误修复：确保如果能力系统在使用中，则调用UAbilitySystemGlobals::InitGlobalData。以前如果用户不调用它，游戏能力系统无法正常工作。
* 错误修复：修复了从UGameplayAbility::EndAbility链接/取消链接动画层时的问题。
* 错误修复：更新了能力系统组件函数，在使用前检查Spec的能力指针。
* 新功能：向FGameplayTagRequirements添加了GameplayTagQuery字段，以启用更复杂的要求。
* 新功能：引入了FGameplayEffectQuery::SourceAggregateTagQuery来增强SourceTagQuery。
* 新功能：扩展了从控制台命令执行和取消游戏能力和游戏效果的功能。
* 新功能：添加了对游戏能力蓝图执行"审计"的能力，将显示有关它们如何开发和预期使用的信息。
* 更改：OnAvatarSet现在在主要实例上调用，而不是在每Actor实例化游戏能力的CDO上。
* 更改：允许在同一游戏能力图中激活能力和从事件激活能力。
* 更改：AnimTask_PlayMontageAndWait现在有一个切换，允许在BlendOut事件后完成和中断。
* 更改：ModMagnitudeCalc包装函数已声明为const。
* 更改：FGameplayTagQuery::Matches现在对空查询返回false。
* 更改：更新了FGameplayAttribute::PostSerialize以将包含的属性标记为可搜索名称。
* 更改：更新了GetAbilitySystemComponent以将默认参数设置为Self。
* 更改：在AbilityTask_WaitTargetData中将函数标记为virtual。
* 更改：移除了未使用的函数FGameplayAbilityTargetData::AddTargetDataToGameplayCueParameters。
* 更改：移除了残留的GameplayAbility::SetMovementSyncPoint。
* 更改：从游戏任务和能力系统组件中移除了未使用的复制标志。
* 更改：将一些游戏效果功能移动到可选组件中。所有现有内容将在PostCDOCompiled期间自动更新以使用组件（如果需要）。

https://docs.unrealengine.com/5.3/en-US/unreal-engine-5.3-release-notes/

## 5.2

## 5.2

* Bug Fix: Fixed a crash in the `UAbilitySystemBlueprintLibrary::MakeSpecHandle` function.
* Bug Fix: Fixed logic in the Gameplay Ability System where a non-Controlled Pawn would be considered remote, even if it was spawned locally on the server (e.g. Vehicles).
* Bug Fix: Correctly set activation info on predicted instanced abilities that were rejected by the server.

* 错误修复：修复了`UAbilitySystemBlueprintLibrary::MakeSpecHandle`函数中的崩溃。
* 错误修复：修复了游戏能力系统中的逻辑，其中非控制的Pawn会被视为远程，即使它在服务器上本地生成（例如车辆）。
* 错误修复：正确设置被服务器拒绝的预测实例化能力的激活信息。

**[⬆ Back to Top](../README.md#table-of-contents)**
