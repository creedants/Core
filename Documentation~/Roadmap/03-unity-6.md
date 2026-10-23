# Unity 6.7 and later

[Back to the summary](README.md)

This section records what Unity has shipped and announced as of 22 October 2026, then turns it into actions for this
package. Each item carries its source. Items are labelled.

- **Confirmed.** Stated in Unity documentation, release notes, the Unity website, or a Unity staff post on Unity
  Discussions about something already released.
- **Announced.** A Unity plan or statement about future work. Plans move. The CoreCLR schedule below already has.
- **Unconfirmed.** From a third party, or not found in a primary source.

## The Unity 6 release line

| Release | Status in October 2026 | Label | Source |
| --- | --- | --- | --- |
| 6.0 LTS | Support ended October 2026 | Confirmed | [Unity 6 support][support] |
| 6.0 LTS exact dates | Support ended 16 October 2026. Enterprise and Industry support runs to 16 October 2027. | Unconfirmed | [endoflife.date][eol] |
| 6.1, 6.2 | Update releases, now out of support | Unconfirmed | [endoflife.date][eol] |
| 6.3 LTS | Released 4 December 2025, supported until December 2027 | Confirmed | [6.3 LTS post][u63], [Unity 6 support][support] |
| 6.4 | Update release, reached end of life | Confirmed | [6.4 post][u64] |
| 6.5 | Update release, out of support since 31 August 2026 | Unconfirmed | [endoflife.date][eol] |
| 6.6 | Current supported release, called the final supported release of Unity 6 before 6.7 LTS | Confirmed | [6.6 post][u66] |
| 6.7 | Public beta since 17 September 2026, no release date | Confirmed | [6.7 beta post][u67b] |
| 6.7 as LTS | Planned as the next LTS | Announced | [6.6 post][u66] |
| 7.0 | First alpha 5 October 2026 | Confirmed | [CoreCLR update, October 2026][clr-oct] |
| 7.0 schedule | Beta in December 2026, release in the first quarter of 2027 | Announced, reported by a third party | [Inven Global][inven] |

Unity's support policy gives an LTS release two years of support (three for Enterprise and Industry customers). An
update release is supported until the next one ships ([Unity 6 support][support], confirmed). For a studio that
ships games over many years, that means building on LTS releases only.

## What Unity 6.7 changes

Unity has not published final 6.7 notes. The 6.7 manual says more will be published when 6.7 leaves beta
([6.7 manual][m67], confirmed). What is known:

- 6.7 is "our final release built upon Mono" ([CoreCLR update, June 2026][clr-jun], announced).
- 6.7 has an experimental CoreCLR player for Windows, macOS, and Linux, not for production use. The editor still runs
  on Mono ([CoreCLR scripting backend][m67-clr], confirmed). The experimental players are limited to C# 9 and the
  .NET Standard 2.1 API ([CoreCLR update, October 2026][clr-oct], confirmed).
- Planned breaking changes ([planned 6.7 changes][u67-break], announced):
  - Serialization warnings UAC1005 to UAC1008, for self-referencing or cyclic by-value types, become compile errors.
    The fix is `[SerializeReference]`.
  - More than 160 long-obsolete APIs in Mecanim, Physics, Audio, NavMesh, and Timeline move from warning to error,
    or from error to removed.
  - The Input System will become a core package, but not in 6.7.
- The Built-in Render Pipeline stays available in 6.7 LTS ([render pipeline strategy][rp], confirmed).

## Unity's direction after 6.7

### CoreCLR, .NET, and domain reload

- The schedule moved. In March 2026 Unity put the CoreCLR editor and the end of Mono in 6.8
  ([CoreCLR and ECS update, March 2026][clr-mar], announced). In October 2026 Unity moved them to 7.0. The 7.0 alpha
  is the first CoreCLR editor without Mono. It still targets C# 9 and .NET Standard 2.1
  ([CoreCLR update, October 2026][clr-oct], confirmed).
- .NET 10 and C# 14 are planned, with no committed date ([CoreCLR update, October 2026][clr-oct], announced).
- Domain reload goes away in 7.0 and is replaced by code reload ([CoreCLR update, June 2026][clr-jun], announced).
  Unity staff confirmed the 7.0 alpha has no domain reload ([CoreCLR update, October 2026][clr-oct], confirmed).
- From 6.6, new projects enter play mode without a domain reload by default, so static fields are not reset. Existing
  projects keep their setting ([6.6 post][u66], confirmed).
- Unity's upgrade guide for library authors ([CoreCLR upgrade guide][clr-guide], announced, a living document) says:
  - Statics are not reset for you. Use `[AutoStaticsCleanup]` or `[BeforeCodeUnloading]`.
  - `AppDomain.GetAssemblies()` raises warnings. Use `UnityEngine.Assemblies.CurrentAssemblies.GetLoadedAssemblies()`.
  - `AppDomain.DomainUnload` no longer fires.
  - `BinaryFormatter` is obsolete, and `string.GetHashCode` is not stable across runs.
  - Burst function-pointer delegate types must be public or internal, and managed exceptions do not cross Burst frames.
- We found no Unity statement about changes to `RuntimeInitializeOnLoadMethod` or `SubsystemRegistration`
  (unconfirmed either way).

### ECS for all and EntityId

- From 6.4, Entities, Collections, Mathematics, and Entities Graphics ship with the editor as core packages
  ([6.4 post][u64], confirmed). Unity's core package table for 6.6 lists Entities, Entities Graphics, Collections,
  Unity Physics, and uGUI. It does not list Mathematics ([core packages][core], confirmed). We could not resolve that
  difference (unconfirmed). A project cannot choose a different version of a core package
  ([core packages][core], confirmed).
- Unity plans for every GameObject to carry ECS component data and for `EntityId` to identify both GameObjects and
  entities ([ECS status, December 2025][ecs-dec], announced).
- `EntityId` replaces instance IDs. `Object.GetInstanceID()` is marked removed in the 6.6 scripting reference
  ([GetInstanceID][getid], confirmed). Unity advises against casting `EntityId` to `int` or sorting by it
  ([ECS status, December 2025][ecs-dec], confirmed).
- The `FindObjectsByType` overloads that take a sort mode are deprecated in 6.6 ([FindObjectsByType][find], confirmed).
- Entities 1.4 marks `Entities.ForEach` and `IAspect` obsolete ([Entities changelog][ent-cl], confirmed).

### Package Manager

- From 6.3, the Package Manager checks signatures on tarball and registry packages and shows a package as signed,
  invalid, or unsigned. Builds before 6000.3.5f2 can wrongly report valid packages as invalid
  ([package signatures][sign], confirmed).
- The Package Manager does not support git dependencies between packages. A package's own `package.json` cannot
  name a git URL. Only the project manifest can ([git dependencies][git], confirmed).
- Unity is moving toward an MSBuild and `.csproj` build pipeline ([CoreCLR update, October 2026][clr-oct], announced).

### Render pipelines, input, and UI

- Built-in Render Pipeline deprecation starts in 6.5, with support at least until the end of 2028. No new features are
  planned for HDRP, and Unity is investing in URP ([render pipeline strategy][rp], confirmed).
- The legacy Input Manager "will be removed in future versions of Unity" ([legacy input][input], confirmed).
- uGUI remains a core package, version 2.6 in 6.6 ([core packages][core], confirmed). We found no deprecation of uGUI
  (unconfirmed).
- `Awaitable` is documented in 6.6 as usually preferable to .NET `Task`. Instances are pooled, so code must not await
  one twice ([Awaitable][await], confirmed).

## Actions for this package

Each action names the fact above that drives it and the phase in the [roadmap](05-roadmap.md) that does it.

1. **Raise the minimum Unity version to 6000.3 LTS (Phase 0).** 6.0 LTS is out of support. Set `"unity": "6000.3"`
   and a `unityRelease` the studio's games already use, no earlier than `5f2` if the packages will be signed. Add
   6000.7 when it ships as LTS, and drop 6000.3 when its support ends in December 2027.
2. **Update package versions (Phase 0).** Entities 1.3.14, Collections 2.5.7, and Burst 1.8.19 are well behind.
   Unity's Burst changelog lists 1.8.31 ([Burst changelog][burst-cl], confirmed), and the Input System changelog lists
   1.20.1 ([Input System changelog][input-cl], confirmed). The latest 1.x Entities line we found is 1.5.0, from a mirror
   of Unity's packages (unconfirmed). Confirm each version in the Package Manager for 6000.3 before bumping.
3. **Stop relying on version pins for core packages (Phase 3).** On 6.4 and later the editor fixes the versions of
   Entities and its siblings. Use `versionDefines` in each asmdef when code must differ by version, and keep
   `package.json` entries as minimums.
4. **Make every static reset explicit (Phase 0, finished in Phase 3).** Domain reload is off by default for new 6.6
   projects and gone in 7.0. Give each static registry the `SubsystemRegistration` reset that `PresetRegistry` already
   has. This applies to `CommandRegistry`, `AttributeCommandRegistry`, `PresetResolver`, `DeltaResolver`, and
   `SingletonManager`. Move to `[AutoStaticsCleanup]` when it ships. Add a PlayMode test that enters play mode twice
   with domain reload off.
5. **Put assembly scanning behind one call (Phase 0).** `dev`'s `Runtime/Tooling/AssemblyScanning.cs` is the single
   place that calls `AppDomain.GetAssemblies()` at runtime. Move `Editor/LogConfigEditor.cs` to `TypeCache`. Then the
   switch to `CurrentAssemblies.GetLoadedAssemblies()` is one edit.
6. **Keep to C# 9 and .NET Standard 2.1 (always, until Unity commits to .NET 10).** The CoreCLR players and the 7.0
   alpha target exactly that.
7. **Prefer `Awaitable` on the main thread (Phase 2).** On `dev`, `IAwardReciever.Award` and
   `IDeleteResolver.ResolveDelete` return `Task<bool>`. (`dev` already made `IObjectDataAccess.DeleteInstance`
   synchronous.) Change them to `Awaitable<bool>` when the objects package is first released, because a new package is
   the cheapest time for a signature change. Keep `Task` for work that
   truly runs off the main thread, such as `FileSink`'s writer.
8. **Track the EntityId and Find API changes (Phase 3).** The code calls no instance-ID API today. A search of `main`
   and `dev` finds no `GetInstanceID`, no `FindObjectsOfType`, no `Entities.ForEach`, and no `IAspect`.
   `AttributeCommandRegistry` uses `FindAnyObjectByType`, which is not deprecated. Add an analyzer or CI search so
   these do not creep in.
9. **Run CI on the next LTS early (Phase 0 onward).** Add 6000.7 beta to the CI matrix as allowed to fail. That
   surfaces the UAC1005 to UAC1008 serialization errors and the obsolete-API removals before 6.7 LTS ships.
10. **Keep samples free of a render pipeline (Phase 2).** Built-in RP is deprecated. The console prefab uses uGUI only,
    which works on every pipeline. Any future visual sample should target URP.
11. **Decide on package signing with the registry choice (Phase 2).** Unsigned packages show as unsigned from 6.3.
12. **Prepare for one simulation, many worlds (Phase 4).** "ECS for all" and Netcode for Entities both run more than
    one world. The world seam in [Package layout](02-package-layout.md#staying-ready-for-multiplayer-and-mmo-games)
    removes the `World.DefaultGameObjectInjectionWorld` lookups.

[support]: https://unity.com/releases/unity-6/support
[eol]: https://endoflife.date/unity
[u63]: https://discussions.unity.com/t/unity-6-3-lts-is-now-available/1697328
[u64]: https://discussions.unity.com/t/unity-6-4-is-now-available/1713245
[u66]: https://discussions.unity.com/t/unity-6-6-is-now-available/1735357
[u67b]: https://discussions.unity.com/t/unity-6-7-beta-is-now-available/1736830
[u67-break]: https://discussions.unity.com/t/planned-breaking-changes-in-unity-6-7/1723293
[m67]: https://docs.unity.com/en-us/engine/6000.7/manual
[m67-clr]: https://docs.unity.com/en-us/engine/6000.7/manual/programming-environment/scripting-backends/coreclr
[clr-mar]: https://discussions.unity.com/t/coreclr-scripting-and-ecs-status-update-march-2026/1711852
[clr-jun]: https://discussions.unity.com/t/coreclr-scripting-and-serialization-update-june-2026/1723299
[clr-oct]: https://discussions.unity.com/t/coreclr-scripting-and-net-update-october-2026/1738338
[clr-guide]: https://discussions.unity.com/t/path-to-coreclr-2026-upgrade-guide/1714279
[ecs-dec]: https://discussions.unity.com/t/ecs-development-status-december-2025/1699284
[core]: https://docs.unity3d.com/Manual/pack-core.html
[getid]: https://docs.unity.com/en-us/engine/6000.6/script-reference/unityengine/object/getinstanceid
[find]: https://docs.unity.com/en-us/engine/6000.6/script-reference/unityengine/object/findobjectsbytype
[ent-cl]: https://docs.unity3d.com/Packages/com.unity.entities@6.6/changelog/CHANGELOG.html
[sign]: https://docs.unity3d.com/6000.3/Documentation/Manual/upm-signature.html
[git]: https://docs.unity3d.com/Manual/upm-git.html
[rp]: https://unity.com/topics/render-pipelines-strategy-for-2026
[input]: https://docs.unity3d.com/6000.4/Documentation/Manual/InputLegacy.html
[await]: https://docs.unity.com/en-us/engine/6000.6/script-reference/unityengine/awaitable
[burst-cl]: https://docs.unity3d.com/Packages/com.unity.burst@1.8/changelog/CHANGELOG.html
[input-cl]: https://docs.unity3d.com/Packages/com.unity.inputsystem@1.20/changelog/CHANGELOG.html
[inven]: https://www.invenglobal.com/articles/24003/unity-engine-7-changing-the-development-paradigm-and-the-roadmap-ahead
