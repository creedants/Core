# Engineering standards

[Back to the summary](README.md)

These standards are for a codebase that many engineers will change over 10 to 15 years, most of whom have not joined
yet. Each rule exists so that a new engineer can tell where code goes and what a change will break. The table at the
end says which to adopt now and which can wait.

## Namespaces and folders

- A package's root namespace matches its assembly name. Core keeps `Spellbound.Core`. Other packages drop `Core`, for
  example `Spellbound.Console` and `Spellbound.Objects`, so the namespace says which package a type comes from.
- One folder is one namespace. Today `Runtime/ObjectData/IAwardReciever.cs` declares `Spellbound.Core.Objects`,
  `Runtime/ObjectHandling/TransformData.cs` declares `Spellbound.Core`, `Runtime/Registries/PackerIdAttribute.cs`
  declares `Spellbound.Core.Packing`, and `Editor/ObjectPresetEditor.cs` declares `Spellbound.Core`. Each mismatch makes
  a reader search for a type. Folder names follow namespace names too, so `Logger/` becomes `Logging/` and
  `EntityPrefabRegistry/` becomes `EntityPrefabs/`.
- Every package uses the same layout: `Runtime/`, `Editor/`, `Tests/Editor/`, `Tests/Runtime/`, `Samples~/`,
  `Documentation~/`, `package.json`, `README.md`, `CHANGELOG.md`, and `LICENSE.md`. This follows Unity's
  [package layout](https://docs.unity3d.com/Manual/cus-layout.html).
- Editor-only code lives in an editor assembly, not behind `#if UNITY_EDITOR` in a runtime file. `#if UNITY_EDITOR`
  stays for the rare case where a runtime type needs an editor hook, such as `OnValidate` in
  `Runtime/Registries/HashedScriptableObject.cs`. Every `using UnityEditor;` sits inside the guard.
- Fix misspelled public names in the same release that moves them, for example `IAwardReciever` to
  `IAwardReceiver`. That release breaks callers anyway.

## Assembly definition rules

1. One assembly per package layer: runtime, editor, EditMode tests, PlayMode tests. Add an assembly inside a layer
   only to isolate an optional dependency, as the objects package does for Entities Graphics and the console.
2. `autoReferenced: false` for every library assembly. Games and other packages reference what they use explicitly.
   That keeps a game's own assembly from silently depending on every Spellbound type. It is a breaking change for games
   whose scripts live in Unity's default `Assembly-CSharp` with no asmdef, so it ships with the package split in
   Phase 2, not before.
3. Explicit references only, by GUID. GUID references survive renames. A CI step maps each GUID to its assembly name
   and fails on an unknown or unused one. That check would have caught the two unused Unity Physics references in
   `Runtime/SpellBound.Core.asmdef`.
4. `allowUnsafeCode` stays off unless a file in that assembly uses `unsafe`. `overrideReferences` stays off unless the
   assembly links a precompiled DLL.
5. Editor assemblies set `includePlatforms` to `Editor`. Test assemblies use `defineConstraints:
   ["UNITY_INCLUDE_TESTS"]`, as `dev`'s `Tests/Spellbound.Core.Tests.asmdef` already does.
6. Optional integrations use `versionDefines` keyed on the other package plus a matching `defineConstraints` entry,
   so the assembly compiles only when that package is installed.

## Public API and breaking changes

- **Internal by default.** A type is public only when a game or another package needs it. Tests reach internals
  through `InternalsVisibleTo`. Most of Core's public surface today is public by habit, so expect Phase 1 to narrow it.
- **Semantic versioning.** A major version may break callers. A minor version adds features. A patch version only
  fixes defects. Serialized data counts as API: renaming a `[PackerId]`, a serialized field, or a
  `[SerializeReference]` type's namespace or assembly is a breaking change.
- **Deprecation window.** A public member is first marked `[Obsolete("Use X instead. Removed in 3.0.")]` as a warning
  for at least one minor release and at least one quarter. It then becomes an error, and the next major version
  removes it.
- **Moving types.** When a type moves namespace or assembly, add `[MovedFrom]` from `UnityEngine.Scripting.APIUpdating`.
  Unity's script updater then rewrites game code, and `[SerializeReference]` data such as the `PresetModule` lists in
  `Runtime/Objects/PresetSurface.cs` keeps loading. MonoBehaviour and ScriptableObject references survive a move as
  long as the `.cs` file keeps its `.meta` file, because Unity stores them by script GUID.
- **Changelog.** Every pull request that changes behavior adds a `CHANGELOG.md` line under an Unreleased heading, in
  the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format. A release turns that heading into a version and
  date. The `dev` changelog already lists versions this way, but it places 2.0.0 (2025) below 1.1.x (2026), which
  should be corrected so versions only increase.
- **API review.** A change to a package's public API needs approval from that package's owner. Once the team grows,
  add a CI step that writes the public API to a text file, so every API change shows up in review.

## Testing

- Every package has EditMode tests, run with the Unity Test Framework, for logic that needs no scene: packing round
  trips, registries, parsing, deltas, and command dispatch. `dev` has fifteen such files for Core today.
- Every package that touches play mode or ECS has PlayMode tests: entering play mode twice with domain reload off,
  spawning and destroying through the ECS systems, and proximity promotion.
- Test behavior that callers rely on, not implementation details. Each defect from
  [Current state](01-current-state.md#defects-found-while-reading) gets a test that fails before its fix.
- **Coverage target.** Measure line coverage with Unity's Code Coverage package. Start at 70% for Core and 50% for the
  other packages. Raise the targets by 10 points a year until Core reaches 85%. Use the number to find untested code.
  Chasing it with shallow tests defeats the point.
- **Determinism tests.** For every `IApplyDelta` implementation in a package, a test applies the same delta to the same
  bytes twice and requires identical output.

## Continuous integration

There is no CI today. Two options fit.

| | GameCI on GitHub Actions | Unity Build Automation |
| --- | --- | --- |
| What it is | Open-source actions that run Unity in containers. Third party, not Unity. | Unity's hosted build service |
| Licence | A Personal licence needs the licence file contents, email, and password as secrets. Plus and Pro need the serial, email, and password ([GameCI activation](https://game.ci/docs/github/activation)). | Uses the studio's Unity organization |
| Package tests | `unity-test-runner` has a package mode on Linux runners. The Unity version must be set explicitly ([GameCI test runner](https://game.ci/docs/github/test-runner)). | Builds projects. Package tests need a host project. |
| Cost | GitHub Actions runner minutes, or a self-hosted runner | 200 Windows, 100 Mac, and 100 Linux build minutes free each month, then per-minute rates. Two concurrent builds free ([Unity DevOps charges](https://support.unity.com/hc/en-us/articles/34748492914964-Understanding-Unity-DevOps-charges)). |
| Fit | The code is on GitHub, the checks sit on the pull request, and the configuration is in the repository | Strong for building and distributing game players, weaker for library checks |

**Recommendation.** Use GameCI on GitHub Actions for the packages now. Consider Unity Build Automation later for the
games' player builds, where it is strongest. Move to a self-hosted runner when hosted minutes or the editor download
time become the bottleneck.

The pipeline for each pull request:

1. Lint the package: `package.json` fields, asmdef GUID references, and namespace-to-folder agreement. A short script
   covers all three.
2. EditMode and PlayMode tests on the minimum supported LTS (6000.3), with coverage.
3. A player build of a minimal host project with the package installed the way a game installs it. This is the check
   that catches editor-only code in runtime assemblies.
4. The same tests on the next LTS (6000.7 beta now), allowed to fail until it ships.

## Code ownership

- A `CODEOWNERS` file names an owner for each package directory. Owners approve changes to public API and to
  `package.json`.
- Each package's `README.md` names its owner and its allowed dependencies. A pull request that adds a dependency
  needs the architecture owner's approval.
- Larger decisions, such as a new package, a dependency, or a breaking change, go in a short decision record under
  `Documentation~/Decisions/`, one file per decision. The record says what was decided, why, and what was rejected.

## Documentation

- Each package has a `README.md` (purpose, install, quick start, owner, dependencies), a `CHANGELOG.md`, and
  `Documentation~/` for longer guides.
- Public types and members carry XML documentation comments. Those comments say what a member promises, not how it
  works.
- Samples live in `Samples~/` and are listed under `samples` in `package.json`, so games import them on request. Move
  `dev`'s `Samples/FastPackerBenchmark.cs` there.
- The external documentation site (linked from `README.md`) is generated from or checked against these files, so it
  cannot drift from the code as `README.md` has drifted (see [Current state](01-current-state.md#documentation-drift)).

## Package distribution

Today games install Core by git URL. The `author.url` in `package.json` still points at the old `Spell-Bound-Studios`
organization, which upstream issue #79 tracks.

Git URLs work for one package. They do not work for several, because the Package Manager does not support git
dependencies between packages ([git dependencies](https://docs.unity3d.com/Manual/upm-git.html)). With git URLs, every
game would have to list Core, Console, and Objects in its own manifest and keep their commits compatible by hand.

**Recommendation.** Publish to a private scoped npm registry. Options include a self-hosted Verdaccio, GitHub Packages,
or a hosted service. Each game adds one `scopedRegistries` entry for `com.spellboundstudios`, and `package.json`
dependencies then resolve by version. Keep git URLs for testing a branch before release. Decide on package signing at
the same time (see [Unity 6.7 and later](03-unity-6.md#package-manager)).

## Adopt now or later

| Standard | When | Why then |
| --- | --- | --- |
| CI with EditMode tests and a player build | Now (Phase 0) | Everything else depends on being able to check it |
| Asmdef hygiene (remove unused references, add missing ones, unsafe off) | Now (Phase 0) | Small, safe, and removes a dependency from every game |
| Changelog per pull request, semantic versioning | Now (Phase 0) | Free, and needed before any breaking change |
| `CODEOWNERS` | Now (Phase 0) | One line per package, and it grows with the team |
| Namespace and folder agreement | Phase 1 | Done while moving types into assemblies, with `[MovedFrom]` |
| PlayMode tests, domain-reload test | Phase 1 | Needed before the split, to prove nothing changed |
| `autoReferenced: false` | Phase 2 | Breaks games without asmdefs, so it ships with the major version |
| Scoped registry | Phase 2 | Required once there is more than one package |
| Public API snapshot in CI, decision records | Phase 2 | Worth it once more than a handful of engineers change the packages |
| Coverage targets enforced in CI | Phase 3 | Report coverage first, enforce once the baseline is known |
| Analyzers for banned APIs | Phase 3 | Lands with the Unity 6.7 work that motivates it |
