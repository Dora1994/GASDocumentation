# 4.2 Gameplay Tags（游戏标签）

[`FGameplayTags`](https://docs.unrealengine.com/en-US/API/Runtime/GameplayTags/FGameplayTag/index.html) are hierarchical names in the form of `Parent.Child.Grandchild...` that are registered with the `GameplayTagManager`. These tags are incredibly useful for classifying and describing the state of an object. For example, if a character is stunned, we could give it a `State.Debuff.Stun` `GameplayTag` for the duration of the stun.

[`FGameplayTags`](https://docs.unrealengine.com/en-US/API/Runtime/GameplayTags/FGameplayTag/index.html)是以 `Parent.Child.Grandchild...`形式的分层名称，在 `GameplayTagManager`中注册。这些标签对于分类和描述对象的状态非常有用。例如，如果一个角色被眩晕，我们可以在眩晕期间给它一个 `State.Debuff.Stun` ``GameplayTags``。

You will find yourself replacing things that you used to handle with booleans or enums with `GameplayTags` and doing boolean logic on whether or not objects have certain `GameplayTags`.

你会发现自己在用 `GameplayTags` 替换你过去用布尔值或枚举处理的东西，并对对象是否具有某些 `GameplayTags`进行布尔逻辑。

When giving tags to an object, we typically add them to its `ASC` if it has one so that GAS can interact with them. `UAbilitySystemComponent` implements the `IGameplayTagAssetInterface` giving functions to access its owned `GameplayTags`.

当给对象添加标签时，我们通常将它们添加到其 `ASC`（如果它有的话），这样GAS就可以与它们交互。`UAbilitySystemComponent`实现了 `IGameplayTagAssetInterface`，提供访问其拥有的 `GameplayTags`的函数。

Multiple `GameplayTags` can be stored in an `FGameplayTagContainer`. It is preferable to use a `GameplayTagContainer` over a `TArray<FGameplayTag>` since the `GameplayTagContainers` add some efficiency magic. While tags are standard `FNames`, they can be efficiently packed together in `FGameplayTagContainers` for replication if `Fast Replication` is enabled in the project settings. `Fast Replication` requires that the server and the clients have the same list of `GameplayTags`. This generally shouldn't be a problem so you should enable this option. `GameplayTagContainers` can also return a `TArray<FGameplayTag>` for iteration.

多个 `GameplayTags`可以存储在 `FGameplayTagContainer`中。使用 `GameplayTagContainer`比 `TArray<FGameplayTag>`更可取，因为 `GameplayTagContainers`添加了一些效率魔法。虽然标签是标准的 `FNames`，但如果项目设置中启用了 `Fast Replication`，它们可以在 `FGameplayTagContainers`中高效地打包在一起进行复制。`Fast Replication`要求服务器和客户端具有相同的 `GameplayTags`列表。这通常不应该是个问题，所以你应该启用这个选项。`GameplayTagContainers`也可以返回 `TArray<FGameplayTag>`用于迭代。

`GameplayTags` stored in `FGameplayTagCountContainer` have a `TagMap` that stores the number of instances of that `GameplayTag`. A `FGameplayTagCountContainer` may still have the `GameplayTag` in it but its `TagMapCount` is zero. You may encounter this while debugging if an `ASC` still has a `GameplayTag`. Any of the `HasTag()` or `HasMatchingTag()` or similar functions will check the `TagMapCount` and return false if the `GameplayTag` is not present or its `TagMapCount` is zero.

存储在 `FGameplayTagCountContainer`中的 `GameplayTags`有一个 `TagMap`，存储该 `GameplayTags`的实例数量。`FGameplayTagCountContainer`可能仍然包含 `GameplayTags`，但其 `TagMapCount`为零。如果你在调试时遇到 `ASC`仍然有 `GameplayTags`的情况，你可能会遇到这种情况。任何 `HasTag()`或 `HasMatchingTag()`或类似函数都会检查 `TagMapCount`，如果 `GameplayTags`不存在或其 `TagMapCount`为零，则返回false。

`GameplayTags` must be defined ahead of time in the `DefaultGameplayTags.ini`. The Unreal Engine Editor provides an interface in the project settings to let developers manage `GameplayTags` without needing to manually edit the `DefaultGameplayTags.ini`. The `GameplayTag` editor can create, rename, search for references, and delete `GameplayTags`.

`GameplayTags`必须事先在 `DefaultGameplayTags.ini`中定义。虚幻引擎编辑器在项目设置中提供了一个界面，让开发者管理 `GameplayTags`，而无需手动编辑 `DefaultGameplayTags.ini`。`GameplayTags`编辑器可以创建、重命名、搜索引用和删除 `GameplayTags`。

![GameplayTag Editor in Project Settings](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaytageditor.png)

Searching for `GameplayTag` references will bring up the familiar `Reference Viewer` graph in the Editor showing all the assets that reference the `GameplayTag`. This will not however show any C++ classes that reference the `GameplayTag`.

搜索 `GameplayTags`引用将在编辑器中显示熟悉的 `Reference Viewer`图，显示所有引用该 `GameplayTags`的资源。但这不会显示任何引用该 `GameplayTags`的C++类。

Renaming `GameplayTags` creates a redirect so that assets still referencing the original `GameplayTag` can redirect to the new `GameplayTag`. I prefer if possible to instead create a new `GameplayTag`, update all the references manually to the new `GameplayTag`, and then delete the old `GameplayTag` to avoid creating a redirect.

重命名 `GameplayTags`会创建一个重定向，这样仍然引用原始 `GameplayTags`的资源可以重定向到新的 `GameplayTags`。如果可能的话，我更喜欢创建一个新的 `GameplayTags`，手动将所有引用更新到新的 `GameplayTags`，然后删除旧的 `GameplayTags`以避免创建重定向。

In addition to `Fast Replication`, the `GameplayTag` editor has an option to fill in commonly replicated `GameplayTags` to optimize them further.

除了 `Fast Replication`之外，`GameplayTags`编辑器还有一个选项来填充常用的复制 `GameplayTags`以进一步优化它们。

`GameplayTags` are replicated if they're added from a `GameplayEffect`. The `ASC` allows you to add `LooseGameplayTags` that are not replicated and must be managed manually. The Sample Project uses a `LooseGameplayTag` for `State.Dead` so that the owning clients can immediately respond to when their health drops to zero. Respawning manually sets the `TagMapCount` back to zero. Only manually adjust the `TagMapCount` when working with `LooseGameplayTags`. It is preferable to use the `UAbilitySystemComponent::AddLooseGameplayTag()` and `UAbilitySystemComponent::RemoveLooseGameplayTag()` functions than manually adjusting the `TagMapCount`.

如果 `GameplayTags`是从 `GameplayEffect`添加的，它们会被复制。`ASC`允许你添加不复制且必须手动管理的 `LooseGameplayTags`。示例项目为 `State.Dead`使用 `LooseGameplayTags`，这样拥有客户端可以立即响应其生命值降至零的情况。重生时手动将 `TagMapCount`设置回零。只有在使用 `LooseGameplayTags`时才手动调整 `TagMapCount`。使用 `UAbilitySystemComponent::AddLooseGameplayTag()`和 `UAbilitySystemComponent::RemoveLooseGameplayTag()`函数比手动调整 `TagMapCount`更可取。

Getting a reference to a `GameplayTag` in C++:

```c++
FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))
```

在C++中获取 `GameplayTags`的引用：

```c++
FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))
```

For advanced `GameplayTag` manipulation like getting the parent or children `GameplayTags`, look at the functions offered by the `GameplayTagManager`. To access the `GameplayTagManager`, include `GameplayTagManager.h` and call it with `UGameplayTagManager::Get().FunctionName`. The `GameplayTagManager` actually stores the `GameplayTags` as relational nodes (parent, child, etc) for faster processing than constant string manipulation and comparisons.

对于高级的 `GameplayTags`操作，如获取父级或子级 `GameplayTags`，请查看 `GameplayTagManager`提供的函数。要访问 `GameplayTagManager`，包含 `GameplayTagManager.h`并使用 `UGameplayTagManager::Get().FunctionName`调用它。`GameplayTagManager`实际上将 `GameplayTags`存储为关系节点（父级、子级等），以便比常量字符串操作和比较更快地处理。

`GameplayTags` and `GameplayTagContainers` can have the optional `UPROPERTY` specifier `Meta = (Categories = "GameplayCue")` that filters the tags in the Blueprint to show only `GameplayTags` that have the parent tag of `GameplayCue`. This is useful when you know the `GameplayTag` or `GameplayTagContainer` variable should only be used for `GameplayCues`.

`GameplayTags`和 `GameplayTagContainers`可以有可选的 `UPROPERTY`说明符 `Meta = (Categories = "GameplayCue")`，它在蓝图中过滤标签，只显示具有 `GameplayCue`父标签的 `GameplayTags`。当你知道 `GameplayTags`或 `GameplayTagContainers`变量应该只用于 `GameplayCues`时，这很有用。

Alternatively, there's a separate structure called `FGameplayCueTag` that encapsulates a `FGameplayTag` and also automatically filters `GameplayTags` in Blueprint to only show those tags with the parent tag of `GameplayCue`.

或者，有一个名为 `FGameplayCueTag`的单独结构，它封装了一个 `FGameplayTag`，并且还在蓝图中自动过滤 `GameplayTags`，只显示具有 `GameplayCue`父标签的标签。

If you want to filter a `GameplayTag` parameter in a function, use the `UFUNCTION` specifier `Meta = (GameplayTagFilter = "GameplayCue")`. `GameplayTagContainer` parameters in functions can not be filtered. If you would like to edit your engine to allow this, look at how `SGameplayTagGraphPin::ParseDefaultValueData()` from `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagGraphPin.cpp` calls `FilterString = UGameplayTagsManager::Get().GetCategoriesMetaFromField(PinStructType);` and passes `FilterString` to `SGameplayTagWidget` in `SGameplayTagGraphPin::GetListContent()`. The `GameplayTagContainer` version of these functions in `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagContainerGraphPin.cpp` do not check for the meta field properties and pass along the filter.

如果你想在函数中过滤 `GameplayTags`参数，使用 `UFUNCTION`说明符 `Meta = (GameplayTagFilter = "GameplayCue")`。函数中的 `GameplayTagContainers`参数无法过滤。如果你想编辑你的引擎以允许这样做，请查看 `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagGraphPin.cpp`中的 `SGameplayTagGraphPin::ParseDefaultValueData()`如何调用 `FilterString = UGameplayTagsManager::Get().GetCategoriesMetaFromField(PinStructType);`并在 `SGameplayTagGraphPin::GetListContent()`中将 `FilterString`传递给 `SGameplayTagWidget`。`Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagContainerGraphPin.cpp`中这些函数的 `GameplayTagContainers`版本不检查元字段属性并传递过滤器。

The Sample Project extensively uses `GameplayTags`.

示例项目广泛使用 `GameplayTags`。

## 4.2.1 Responding to Changes in Gameplay Tags （响应游戏标签的变化）

The `ASC` provides a delegate for when `GameplayTags` are added or removed. It takes in a `EGameplayTagEventType` that can specify only to fire when the `GameplayTag` is added/removed or for any change in the `GameplayTag's` `TagMapCount`.

`ASC`为 `GameplayTags`被添加或移除时提供一个委托。它接受一个 `EGameplayTagEventType`，可以指定只在 `GameplayTags`被添加/移除时触发，或者对 `GameplayTags`的 `TagMapCount`的任何变化触发。

```c++
AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("State.Debuff.Stun")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);
```

The callback function has a parameter for the `GameplayTag` and the new `TagCount`.

```c++
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);
```

回调函数有一个 `GameplayTags`参数和新的 `TagCount`。

```c++
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);
```

## 4.2.2 Loading Gameplay Tags from Plugin .ini Files （从插件.ini文件加载游戏标签）

If you create a plugin with its own .ini files with `GameplayTags`, you can load that plugin's `GameplayTag` .ini directory in your plugin's `StartupModule()` function.

如果你创建一个带有自己的包含 `GameplayTags`的.ini文件的插件，你可以在插件的 `StartupModule()`函数中加载该插件的 `GameplayTags`.ini目录。

For example, this is how the CommonConversation plugin that comes with Unreal Engine does it:

例如，这是虚幻引擎附带的CommonConversation插件的方式：

```c++
void FCommonConversationRuntimeModule::StartupModule()
{
	TSharedPtr<IPlugin> ThisPlugin = IPluginManager::Get().FindPlugin(TEXT("CommonConversation"));
	check(ThisPlugin.IsValid());

	UGameplayTagsManager::Get().AddTagIniSearchPath(ThisPlugin->GetBaseDir() / TEXT("Config") / TEXT("Tags"));

	//...
}
```

This would look for the directory `Plugins\CommonConversation\Config\Tags` and load any .ini files with `GameplayTags` in them into your project when the Engine starts up if the plugin is enabled.

这将查找目录 `Plugins\CommonConversation\Config\Tags`，并在引擎启动时（如果插件被启用）将其中包含 `GameplayTags`的任何.ini文件加载到你的项目中。

**[⬆ Back to Top](../README.md#table-of-contents)**
