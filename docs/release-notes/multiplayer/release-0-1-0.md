---
id: mlapi-0-1-0
title: MLAPI 0.1.0 - 2021-03-23
description: Release notes for Multiplayer v0.1.0, including new features, updates, bug fixes, known issues, and information to help you upgrade.
label: 
---

The Multiplayer v0.1.0 Experimental release contains features, updates, bug fixes, and refactoring for the first release of MLAPI for Unity.

| Product | Version | Status | Release Date | Supported Unity Versions |
| -- | -- | -- | -- | -- |
| MLAPI | 0.1.0 | Experimental | March 23, 2021 | 2019.4 and later |

:::note
Unity MLAPI supports Windows and MacOS versions of Unity Editor and Player.
:::

## New features

This release provides the following new features and APIs:

* Refactored a new standard for Remote Procedure Call (RPC) in MLAPI which provides increased performance, significantly reduced boilerplate code, and extensibility for future-proofed code. MLAPI RPC includes `ServerRpc` and `ClientRpc` to execute logic on the server and client-side. This provides a single performant unified RPC solution, replacing MLAPI Convenience and Performance RPC (see [here](#removed-features)). For information on compatibility and deprecations, see [RPC Migration and Compatibility](../../advanced-topics/message-system/rpc-compatibility.md).
* Added standarized serialization types, including built-in and custom serialization flows. See [About Serialization](../../advanced-topics/serialization/serialization-intro.md) for details.
* `INetworkSerializable` interface replaces `IBitWritable`.
* Added `NetworkSerializer`..., which is the main aggregator that implements serialization code for built-in supported types and holds `NetworkReader` and `NetworkWriter` instances internally.
* Added a Network Update Loop infrastructure that aids Netcode systems to update (such as RPC queue and transport) outside of the standard `MonoBehaviour` event cycle. See [Network Update Loop](../../advanced-topics/network-update-loop-system/index.md) and the following details: <!-- MTT-498 RFC #8 -->
  * It uses Unity's [low-level Player Loop API](https://docs.unity3d.com/ScriptReference/LowLevel.PlayerLoop.html) and allows for registering `INetworkUpdateSystem`s with `NetworkUpdate` methods to be executed at specific `NetworkUpdateStage`s, which may also be before or after `MonoBehaviour`-driven game logic execution.
  * You will typically interact with `NetworkUpdateLoop` for registration and `INetworkUpdateSystem` for implementation.
  * `NetworkTickSystem` tracks time through network interactions and syncs `NetworkVariable`s, used in this update loop. <!-- MTT-241, RFC #12-->
<!--IN RFC - MAY COME BACK * Extended `Transport` to expose `NetworkAddress` and `NetworkPort` properties, used to change the address and port which an MLAPI client connects to at runtime or change the port on which a server gets hosted. This change promotes cleaner code and implementations, and makes it more interchangeable in both user code and library extensions.  -->
* Added message batching to handle consecutive RPC requests sent to the same client. `RpcBatcher` sends batches based on requests from the `RpcQueueProcessing`, by batch size threshold or immediately. <!-- add link to docs -->
* [GitHub 494](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/494): Added a constraint to allow one `NetworkObject` per `GameObject`, set through the `DisallowMultipleComponent` attribute.
* Integrated MLAPI with the Unity Profiler for versions 2020.2 and later:
  * Added new profiler modules for MLAPI that report important network data.
  * Attached the profiler to a remote player to view network data over the wire.
  * When installed you will see the following modules in the Unity UI:

    ![](/img/profiler-modules.png)

:::tip
A test project is available for building and experimenting with MLAPI features. This project is available in the MLAPI GitHub [testproject folder](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/tree/release/0.1.0/testproject). 

We also provide a new [Hello World example](../../tutorials/helloworldintro.md) to walk through installation to building your first networked game.
:::

[MLAPI Community Contributions](https://github.com/Unity-Technologies/mlapi-community-contributions/tree/master/com.mlapi.contrib.extensions) is a new GitHub repository open to the MLAPI community for extensions. Current extensions include moved MLAPI features for lag compensation (useful for Server Authoritative actions) and `TrackedObject`.

## Changes

This release includes the following updates:

* MLAPI now uses the Unity Package Manager for installation management. <!-- PR 520-->
* Added functionality and usability to [`NetworkVariable`](../../mlapi-basics/networkvariable.md), previously called `NetworkVar`. Updates enhance options and fully replace the need for `SyncedVar`s. 
* [GitHub 507](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/507): Reimplemented `NetworkAnimator`, which synchronizes animation states for networked objects. 

### Refactored API names

For users of previous versions of MLAPI, this release renames APIs due to refactoring. All obsolete marked APIs have been removed as per [GitHub 513](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/513). <!-- more coming from Fatih -->

| Previous MLAPI Versions | V 0.1.0 Name |
| -- | -- |
| `NetworkingManager` | `NetworkManager` |
| `NetworkedObject` | `NetworkObject` |
| `NetworkedBehaviour` | `NetworkBehaviour` |
| `NetworkedClient` | `NetworkClient` |
| `NetworkedPrefab` | `NetworkPrefab` |
| `NetworkedVar` | `NetworkVariable` |
| `NetworkedTransform` | `NetworkTransform` |
| `NetworkedAnimator` | `NetworkAnimator` |
| `NetworkedAnimatorEditor` | `NetworkAnimatorEditor` |
| `NetworkedNavMeshAgent` | `NetworkNavMeshAgent` |
| `SpawnManager` | `NetworkSpawnManager` |
| `BitStream` | `NetworkBuffer` |
| `BitReader` | `NetworkReader` |
| `BitWriter` | `NetworkWriter` |
| `NetEventType` | `NetworkEventType` |
| `ChannelType` | `NetworkDelivery` |
| `Channel` | `NetworkChannel` |
| `Transport` | `NetworkTransport` |
| `NetworkedDictionary` | `NetworkDictionary` |
| `NetworkedList` | `NetworkList` |
| `NetworkedSet` | `NetworkSet` |
| `MLAPIConstants` | `NetworkConstants` |

Refactoring includes the following changes:

* GitHub [444](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/444) and [455](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/455): Channels are now represented as bytes instead of strings.
* [GitHub 514](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/514): Refactored and updated code to use the following:

  * Apply Unity C# standards across the entire MLAPI framework.
  * Use string interpolation over string concatenation with + (plus) operator.
  * Use `var` instead of full types when it is obvious from the right side of the statement. For example: `var targetScript = scriptProperty.objectReferenceValue as MonoScript;` instead of `MonoScript targetScript`
  * Use `nameof` operator when possible. For example: `Debug.LogError($"{nameof(ILPostProcessor)} Error - {message.MessageData} {message.File}:{message.Line}");`
  * Use explicit access modifiers.

### Removed features

With a new release of MLAPI in Unity, some features have been removed:

* SyncVars have been removed from MLAPI. Use `NetworkVariable`s in place of this functionality. <!-- MTT54 -->
* [GitHub 527](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/527): Lag compensation systems and `TrackedObject` have moved to the new [MLAPI Community Contributions](https://github.com/Unity-Technologies/mlapi-community-contributions/tree/master/com.mlapi.contrib.extensions) repo.
* [GitHub 509](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/509): Encryption has been removed from MLAPI. The `Encryption` option in `NetworkConfig` on the `NetworkingManager` is not available in this release. This change will not block game creation or running. A current replacement for this functionality is not available, and may be developed in future releases. See the following changes: <!-- MTT-516-->

    * Removed `SecuritySendFlags` from all APIs.
    * Removed encryption, cryptography, and certificate configurations from APIs including `NetworkManager` and `NetworkConfig`.
    * Removed "hail handshake", including `NetworkManager` implementation and `NetworkConstants` entries.
    * Modified `RpcQueue` and `RpcBatcher` internals to remove encryption and authentication from reading and writing.

* Removed the previous MLAPI Profiler editor window from Unity versions 2020.2 and later.
* Removed previous MLAPI Convenience and Performance RPC APIs with the new standard RPC API. <!-- RFC#1 -->
* [GitHub 520](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/520): Removed the MLAPI Installer.
* Fixed an issue with NetworkSceneManager, where users received a “soft synch” exception typically followed by a `null reference` exception on the client side when the host or server invoked the `NetworkSceneManager.SwitchScene` method. Transitioning between in-game session scenes, where the scene being transitioned into contains one or more manually placed `GameObject`(s) with the `NetworkObject` component attached, should no longer cause MLAPI to throw exceptions. This fix should improve the overall stability of scene-to-scene transitions for the user.

## Fixes

This release includes the following issue fixes:

* [GitHub 460](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/460): Fixed an issue for RPC where the host-server was not receiving RPCs from the host-client and vice versa without the loopback flag set in `NetworkingManager`.  <!-- MTT466 -->
* Fixed an issue where data in the Profiler was incorrectly aggregated and drawn, which caused the profiler data to increment indefinitely instead of resetting each frame. <!-- MTT-526-->
* Fixed an issue the client soft-synced causing PlayMode client-only scene transition issues, caused when running the client in the editor and the host as a release build. Users may have encountered a soft sync of `NetworkedInstanceId` issues in the `NetworkSpawnManager.ClientCollectSoftSyncSceneObjectSweep` method. <!--MTT-505 -->
* [GitHub 458](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/458): Fixed serialization issues in `NetworkList` and `NetworkDictionary` when running in Server mode.
* [GitHub 498](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/498): Fixed numerical precision issues to prevent not a number (NaN) quaternions.
* [GitHub 438](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/438): Fixed booleans by reaching or writing bytes instead of bits.
* [GitHub 519](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi/pull/519): Fixed an issue where calling `Shutdown()` before making `NetworkManager.Singleton = null` is null on `NetworkManager.OnDestroy()`.
* Fixed an issue with `NetworkSceneManager`, where users received a “soft synch” exception typically followed by a `null reference` exception on the client side when the host or server invoked the `NetworkSceneManager.SwitchScene` method. Transitioning between in-game session scenes, where the scene being transitioned into contains one or more manually placed `GameObject`(s) with the `NetworkObject` component attached, should no longer cause MLAPI to throw exceptions. This fix should improve the overall stability of scene-to-scene transitions for the user.

## Known issues

Review the following known issues with this release:

* `NetworkNavMeshAgent` does not synchronize mesh data, Agent Size, Steering, Obstacle Avoidance, or Path Finding settings. It only synchronizes the destination and velocity, not the path to the destination.
* For `RPC`, methods with a `ClientRpc` or `ServerRpc` suffix which are not marked with [ServerRpc] or [ClientRpc] will cause a compiler error.
* For `NetworkAnimator`, Animator Overrides are not supported. Mechanism trigger parameters (such as auto resetting tools) do not work.
* For `NetworkVariable`, the `NetworkDictionary` `List` and `Set` must use the `reliableSequenced` channel.
* `NetworkObjects`s are supported but when spawning a prefab with nested child network objects you have to manually call spawn on them
* `NetworkTransform` has the following issues:
  * Replicated objects may have jitter. 
  * The owner is always authoritative about the object's position.
  * Scale is not synchronized.
* Connection Approval is not called on the host client.
* For `NamedMessages`, always use `NetworkBuffer` as the underlying stream for sending named and unnamed messages.
* `NetworkManager` have the following issues:
  * Connection management is limited. Use `IsServer`, `IsClient`, `IsConnectedClient`, or other code to check if MLAPI connected correctly.
  * Adding a `GameObject` with a `NetworkObject` component as a child to a `GameObject` that is assigned the `NetworkManager` component will cause a soft synchronization error. Avoid assigning `NetworkObject`s to a `GameObject` with the `NetworkManager`.

## Upgrade guide

If using UNet, see the [Migrating From UNet to MLAPI](../../migration/migratingtomlapi.md).

If using previous versions of MLAPI, see [Updating to the Unity Package](../../migration/migratingfrommlapi.md).

## Learn more

See the [Unity Multiplayer MLAPI](https://github.com/Unity-Technologies/com.unity.multiplayer.mlapi) repository to learn more about contributing, open issues, and in-progress development.

To provide feedback and content on documentation, see the links at the bottom of each page.
