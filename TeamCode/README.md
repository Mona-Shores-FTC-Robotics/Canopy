# TeamCode — external tooling

This module builds on the stock FTC SDK (`11.2.1`) with a small set of external
libraries. This doc records what's included, what was deliberately left out,
and why — so the reasoning survives past whoever added it.

## Included

| Library | Artifact | Purpose |
|---|---|---|
| Sloth | `dev.frozenmilk.sinister:Sloth:0.2.4` (+ `dev.frozenmilk:Load:0.2.4` Gradle plugin) | Fast dex scanning at OpMode discovery time — roughly 50% faster init than the SDK's plain reflective scan. |
| Pedro Pathing | `com.pedropathing:ftc:2.1.2` | Autonomous path following (Bézier curves, continuous path correction). |
| Ivy | `com.pedropathing:ivy:1.0.1` | Command-based control flow (scheduler, `Command`/`CommandBuilder`, subsystem requirements/priority). Pedro Pathing's own command framework — used in place of NextFTC. |
| CachingHardware | `dev.frozenmilk.dairy:CachingHardware:1.0.0` | Wraps motor/servo writes to skip redundant `setPower`/`setPosition` calls when the new value is within tolerance of the cached one. |
| Panels | `com.bylazar.sloth:fullpanels:0.2.4+1.0.12` | Live dashboard: real-time tuning of constants (including Pedro path constants, which plain FTC Dashboard can't tune), field/pose overlay, wireless Limelight pipeline tuning, telemetry graphs. Sloth-compatible build variant of `com.bylazar:fullpanels`. |
| FTC Dashboard | `com.acmerobotics.slothboard:dashboard:0.2.4+0.5.1` | Passive telemetry/field monitoring, run alongside Panels. Sloth-compatible build variant of `com.acmerobotics.dashboard:dashboard`. Kept in case we resume AdvantageScope integration (it piggybacks on FTC Dashboard's packet stream). |

Limelight3A support (`com.qualcomm.hardware.limelightvision`) needs no separate
dependency — it ships as part of the SDK's `Hardware` artifact in
`build.dependencies.gradle`.

## Deliberately excluded

- **NextFTC** — the previous command framework. Migrated off it onto Ivy;
  reintroducing it would mean maintaining two overlapping command schedulers.
- **`com.pedropathing:telemetry`** — only existed as a crutch for one
  unmigrated file in the source project this was based on. Don't carry it
  forward; if a file needs it, port that file to the current telemetry API
  instead of re-adding the dependency.
- **Road Runner / `maven.brott.dev`** — not in use; the maven repo isn't
  declared here to avoid an unused, unexplained entry.
- **AdvantageScope Lite** — not currently wired in. Add it back only if that
  debugging workflow is actually resumed.
- **Marrow** (`io.github.skeleton-army.marrow`) — a newer reactive-behavior
  library that layers on NextFTC/FTCLib/SolversLib. No evidence yet of use by
  competitive teams; worth revisiting, not a default include.
- **`exportPaths` Gradle task** — a nice-to-have that exports autonomous
  paths as `.pp` files for the Pedro Pathing visualizer. Not added here
  because it depends on a `util.ExportAutoPaths` utility class that doesn't
  exist in this project yet. Add both together when autonomous path code
  exists to export.

## SDK 11.2.1 compatibility notes

FTC SDK 11.1 → 11.2.1 has no breaking Java API changes for third-party code —
the only breaking change is a build-tooling bump (Gradle 9.1 + AGP 8.13.2),
which requires **Android Studio Narwhal 3 Feature Drop or later** to sync the
project at all.

- **Sloth** is a Gradle plugin (not just a runtime library), so it's the
  dependency most exposed to that bump. Confirmed safe: Sloth added explicit
  Gradle 9 / configuration-cache support in a commit merged 2026-05-06 —
  two months before the SDK itself required Gradle 9.1. `0.2.4` (pinned
  above) is that fix.
- **Panels, FTC Dashboard, and CachingHardware** are plain runtime Android
  libraries, not build plugins, so they're much less exposed to a Gradle/AGP
  bump. That said, their source repos have had no commits since before the
  SDK's Gradle 9 requirement landed — no reported breakage, but also no
  explicit re-validation. Do a real build-and-deploy-to-robot smoke test
  before relying on them mid-season.
- **Pedro Pathing / Ivy** are actively maintained (Ivy had a commit as
  recently as 2026-08-07) with no reported compatibility issues.
