# Spellbound Core architecture roadmap

Status: proposal for the studio to accept or amend. Prepared October 2026 against `main` at `a77a8c1` and the
`dev` branch that upstream PR #74 ("1.1 Release") would merge. This roadmap changes no code.

## Executive summary

Spellbound Core already holds the right building blocks for a long-lived foundation. It has a tagged binary
serializer, hashed asset identity, a compile-time-stripped logger, a console, and a preset-driven object model with
an ECS representation. The `dev` branch adds the first test suite and an editor assembly. The problem is not the
building blocks. It is that they all live in one assembly that depends on eight Unity packages, and the folders inside
it depend on each other in circles. Today a game that wants only the logger also pulls in Entities, Entities Graphics,
Unity Physics, the Input System, and uGUI.

### Recommended package layout

| Package | Holds | Depends on |
| --- | --- | --- |
| `com.spellboundstudios.core` | Logging, hashing, packing, registries, hashed assets, pooling, sampling, small editor tools | The Unity engine only. No Unity packages at all. |
| `com.spellboundstudios.console` | Command registry, parser, built-in commands, the uGUI console view and its prefab | Core, uGUI (which includes TextMeshPro), Input System |
| `com.spellboundstudios.objects` | Presets, modules and their contracts, instance data, event surfaces, the ECS spawning and proximity systems | Core, Entities (with Collections, Mathematics, Burst) |

Every arrow points toward Core, and Core points at nothing outside the engine. Other packages wait until a real trigger
justifies them. [Package layout](02-package-layout.md) lists those triggers.

### The five most important changes

1. **Add a safety net before moving anything.** Land PR #74, then add CI that runs the existing tests and makes a
   player build on every pull request. Today nothing checks that the package compiles outside the editor, and one
   runtime file (`Runtime/Tooling/ImmutableAttribute.cs`) has an editor-only `using` that we expect to break player
   builds.
2. **Draw the boundaries with assemblies first, then with packages.** Split the single `Spellbound.Core` assembly into
   per-area assemblies inside the current package. The compiler then enforces the dependency rules, and the later
   package split becomes a file move.
3. **Shrink Core to the engine.** Core's code that is not about consoles or world objects uses no Unity package. Move
   Entities, Entities Graphics, Input System, and uGUI to the packages that use them, and drop Unity Physics, which no
   file uses.
4. **Move to Unity 6.3 LTS now and plan for 6.7 LTS.** Unity lists 6.0 LTS support through October 2026. Unity 6.7 is in
   beta, is planned as the next LTS, and is the last release on Mono. Unity 7 removes domain reload. The package needs
   to reset its static state explicitly and stop using `AppDomain` assembly scans.
5. **Keep multiplayer and MMO games possible without writing networking.** Keep data separate from presentation, make the
   byte format explicit and versioned, and pass worlds, time, randomness, and instance-id allocation in as interfaces.
   A game's own networking layer then plugs into named seams.

### Timeline

| Phase | When (quarters from kickoff) | Outcome |
| --- | --- | --- |
| 0. Safety net | Q1, first six weeks | CI, tests in CI, player build check, asmdef and dependency hygiene, Unity 6.3 LTS minimum for the 2.0 candidate |
| 1. Boundaries inside one package | Q1 to Q2 | Per-area assemblies, no folder cycles, `[MovedFrom]` on moved types |
| 2. Package split | Q2 to Q3 | Core 2.0, Console 1.0, Objects 1.0 on a scoped registry; Core 1.x kept for in-flight games |
| 3. Unity 6.7 LTS alignment | Q3, once 6.7 LTS has shipped | Domain-reload-free, CoreCLR-ready, `EntityId`-ready |
| 4. Objects refactor and multiplayer seams | Q3 to Q5 | `ObjectParent` broken up, injected world, versioned byte format |
| 5. Scale-out | Year 2 onward | Unity 7, later packages when their triggers fire, team process |

### Decisions the studio must make

- Accept or amend the three-package layout and the package names.
- Choose a package registry (recommended: a scoped npm registry) to replace git URLs. Unity does not support git
  dependencies between packages, so without a registry every game must list and align each package by hand.
- Choose the CI service and fund its Unity licence and runner minutes.
- Set the minimum Unity version (recommended: 6000.3 LTS now, 6000.7 LTS once it ships).
- Decide how long Core 1.x receives fixes while in-flight games migrate (recommended: two quarters after Core 2.0).
- Decide whether the game-specific types listed in [Current state](01-current-state.md) move into the games.

## How to read this roadmap

1. [Current state](01-current-state.md). What exists today, what each folder depends on, and where it strains.
2. [Package layout](02-package-layout.md). The proposed packages and assemblies, the dependency graph, packages to
   create later, and how the packages stay ready for multiplayer without networking code.
3. [Unity 6.7 and later](03-unity-6.md). What Unity has shipped and announced, with sources, and the actions that
   follow for this code.
4. [Engineering standards](04-engineering-standards.md). Conventions for a large team over 10 to 15 years, and which to
   adopt now.
5. [Phased roadmap](05-roadmap.md). What, why, and when, with entry and exit criteria, effort, risks, the migration
   strategy for in-flight games, and the open questions for the studio.
