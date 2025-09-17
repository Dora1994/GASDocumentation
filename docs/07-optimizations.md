# 7. Optimizations (优化)

## 7.1 Ability Batching (能力批处理)


[GameplayAbilities](04-concepts/04-6-gameplay-abilities.md) that activate, optionally send `TargetData` to the server, and end all in one frame can be [batched to condense two-three RPCs into one RPC](04-concepts/04-6-gameplay-abilities.md). These types of abilities are commonly used for hitscan guns.

在一帧内激活、可选地向服务器发送`TargetData`并结束的[GameplayAbilities](04-concepts/04-6-gameplay-abilities.md)可以[批处理以将两到三个RPC压缩为一个RPC](04-concepts/04-6-gameplay-abilities.md)。这些类型的能力通常用于命中扫描枪。

## 7.2 Gameplay Cue Batching (游戏提示批处理)

If you're sending many [GameplayCues](04-concepts/04-8-gameplay-cues.md) at the same time, consider [batching them into one RPC](04-concepts/04-8-gameplay-cues.md). The goal is to reduce the number of RPCs (`GameplayCues` are unreliable NetMulticasts) and send as little data as possible.

如果你同时发送许多[GameplayCues](04-concepts/04-8-gameplay-cues.md)，考虑[将它们批处理为一个RPC](04-concepts/04-8-gameplay-cues.md)。目标是减少RPC的数量（`GameplayCues`是不可靠的NetMulticasts）并发送尽可能少的数据。

## 7.3 AbilitySystemComponent Replication Mode (能力系统组件复制模式)

By default, the [ASC](04-concepts/04-1-ability-system-component.md) is in [Full Replication Mode](04-concepts/04-1-ability-system-component.md). This will replicate all [GameplayEffects](04-concepts/04-5-gameplay-effects.md) to every client (which is fine for a single player game). In a multiplayer game, set the player owned `ASCs` to `Mixed Replication Mode` and AI controlled characters to `Minimal Replication Mode`. This will replicate `GEs` applied on a player character to only replicate to the owner of that character and `GEs` applied on AI controlled characters will never replicate `GEs` to clients. [GameplayTags](04-concepts/04-2-gameplay-tags.md) will still replicate and [GameplayCues](04-concepts/04-8-gameplay-cues.md) will still be unreliable NetMulticast to all clients, regardless of the `Replication Mode`. This will cut down on network data from `GEs` being replicated when all clients don't need to see them.

默认情况下，[ASC](04-concepts/04-1-ability-system-component.md)处于[Full Replication Mode](04-concepts/04-1-ability-system-component.md)。这会将所有[GameplayEffects](04-concepts/04-5-gameplay-effects.md)复制到每个客户端（这对单人游戏来说很好）。在多人游戏中，将玩家拥有的`ASC`设置为`Mixed Replication Mode`，将AI控制的角色设置为`Minimal Replication Mode`。这将使应用于玩家角色的`GE`只复制到该角色的拥有者，而应用于AI控制角色的`GE`永远不会将`GE`复制到客户端。[GameplayTags](04-concepts/04-2-gameplay-tags.md)仍会复制，[GameplayCues](04-concepts/04-8-gameplay-cues.md)仍会向所有客户端进行不可靠的NetMulticast，无论`Replication Mode`如何。这将减少当所有客户端不需要看到它们时从`GE`复制的网络数据。

## 7.4 Attribute Proxy Replication (属性代理复制)

In large games with many players like Fortnite Battle Royale (FNBR), there will be a lot of [ASCs](04-concepts/04-1-ability-system-component.md) living on always-relevant `PlayerStates` replicating a lot of [Attributes](04-concepts/04-3-attributes.md). To optimize this bottleneck, Fortnite disables the `ASC` and its [AttributeSets](04-concepts/04-4-attribute-set.md) from replicating altogether on **simulated player-controlled proxies** in the `PlayerState::ReplicateSubobjects()`. Autonomous proxies and AI controlled `Pawns` still fully replicate according to their [Replication Mode](04-concepts/04-1-ability-system-component.md). Instead of replicating `Attributes` on the `ASC` on the always-relevant `PlayerStates`, FNBR uses a replicated proxy structure on the player's `Pawn`. When `Attributes` change on the server's `ASC`, they are changed on the proxy struct too. The client receives the replicated `Attributes` from the proxy struct and pushes the changes back into its local `ASC`. This allows `Attribute` replication to use the `Pawn`'s relevancy and `NetUpdateFrequency`. This proxy struct also replicates a small white-listed set of `GameplayTags` in a bitmask. This optimization reduces the amount of data over the network and allows us to take advantage of pawn relevancy. AI controlled `Pawns` have their `ASC` on the `Pawn` which already uses its relevancy so this optimization is not needed for them.

在像Fortnite Battle Royale (FNBR)这样有很多玩家的大型游戏中，会有很多[ASC](04-concepts/04-1-ability-system-component.md)生活在总是相关的`PlayerStates`上，复制大量[Attributes](04-concepts/04-3-attributes.md)。为了优化这个瓶颈，Fortnite在`PlayerState::ReplicateSubobjects()`中完全禁用**模拟玩家控制代理**上的`ASC`及其[AttributeSets](04-concepts/04-4-attribute-set.md)的复制。自主代理和AI控制的`Pawn`仍然根据其[Replication Mode](04-concepts/04-1-ability-system-component.md)完全复制。FNBR不是在总是相关的`PlayerStates`上的`ASC`上复制`Attributes`，而是在玩家的`Pawn`上使用复制的代理结构。当服务器上的`ASC`上的`Attributes`发生变化时，代理结构上的也会发生变化。客户端从代理结构接收复制的`Attributes`并将更改推回其本地`ASC`。这允许`Attributes`复制使用`Pawn`的相关性和`NetUpdateFrequency`。这个代理结构还在位掩码中复制一小部分白名单`GameplayTags`。这种优化减少了网络上的数据量，并允许我们利用pawn相关性。AI控制的`Pawn`在`Pawn`上有其`ASC`，这已经使用了其相关性，所以这种优化对它们来说是不需要的。

> I'm not sure if it is still necessary with other server side optimizations that have been done since then (Replication Graph, etc) and it is not the most maintainable pattern.

> 我不确定在自那以后完成的其他服务器端优化（复制图等）是否仍然需要，而且这不是最可维护的模式。

*Dave Ratti from Epic's answer to [community questions #3](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)*

*Dave Ratti来自Epic对[社区问题#3](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)的回答*

## 7.5 ASC Lazy Loading (ASC延迟加载)

Fortnite Battle Royale (FNBR) has a lot of damageable `AActors` (trees, buildings, etc) in the world, each with an [ASC](04-concepts/04-1-ability-system-component.md). This can add up in memory cost. FNBR optimizes this by lazily loading `ASCs` only when they're needed (when they first take damage by a player). This reduces overall memory usage since some `AActors` may never be damaged in a match.

Fortnite Battle Royale (FNBR)在世界上有很多可损坏的`AActors`（树木、建筑物等），每个都有一个[ASC](04-concepts/04-1-ability-system-component.md)。这可能会累积内存成本。FNBR通过仅在需要时（当它们第一次被玩家损坏时）延迟加载`ASC`来优化这一点。这减少了总体内存使用，因为一些`AActors`可能在一场比赛中永远不会被损坏。

**[⬆ Back to Top](../README.md#table-of-contents)**
