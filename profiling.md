# Ghost Flag Database

How the ghost identification system works. Covers all 25 ghosts. Nothing here is work-in-progress or planned.

The system has two independent layers that run side by side:

1. Evidence matching. Standard game evidence (EMF 5, Ghost Orb, and so on). This narrows the ghost list the normal way.
2. Flag detection. Ghost-specific behaviors and traits that the game exposes to the client. This flags likely ghosts on top of the evidence match.

You read them together. Evidence tells you which ghosts are possible. Flags tell you which of those is likely, and sometimes confirm one outright.

## Evidence layer

Eight evidence types drive the standard match.

| Key | Display name |
|-----|--------------|
| EMF5 | EMF 5 |
| GhostOrbs | Ghost Orb |
| GhostWriting | Inscription |
| FreezingTemps | Freezing Temps |
| Handprints | Prints |
| LaserProjector | Laser Projector |
| Wither | Wither |
| SpiritBox | Spirit Box |

Each of the 25 ghosts has three evidence types. The matcher collects evidence you've found, then walks every ghost and scores it. A ghost with no missing evidence and at least one real match is an exact match. If exactly one ghost is an exact match, it locks.

### Ghost evidence table

| Ghost | Evidence |
|-------|----------|
| Aswang | Inscription, Wither, EMF 5 |
| Banshee | Ghost Orb, Prints, Freezing Temps |
| Demon | Prints, Freezing Temps, EMF 5 |
| Dullahan | Wither, Freezing Temps, Laser Projector |
| Dybbuk | Wither, Freezing Temps, Prints |
| Entity | Laser Projector, Prints, Spirit Box |
| Ghoul | Ghost Orb, Spirit Box, Freezing Temps |
| Keres | Wither, Spirit Box, Prints |
| Leviathan | Ghost Orb, Prints, Inscription |
| Nightmare | Spirit Box, Ghost Orb, EMF 5 |
| Oni | Spirit Box, Freezing Temps, Laser Projector |
| Phantom | Ghost Orb, EMF 5, Prints |
| Revenant | Freezing Temps, EMF 5, Inscription |
| Shadow | EMF 5, Inscription, Laser Projector |
| Siren | Spirit Box, Wither, EMF 5 |
| Skinwalker | Spirit Box, Freezing Temps, Inscription |
| Specter | EMF 5, Freezing Temps, Laser Projector |
| Spirit | Inscription, Spirit Box, Prints |
| Wisp | Wither, Ghost Orb, Laser Projector |
| Umbra | Ghost Orb, Laser Projector, Prints |
| Vex | Wither, Ghost Orb, Freezing Temps |
| Wendigo | Ghost Orb, Laser Projector, Inscription |
| Wraith | Spirit Box, Laser Projector, EMF 5 |
| Ravager | EMF 5, Inscription, Spirit Box |
| Vesper | Wither, Inscription, Prints |

### No-Evidence mode

When No-Evidence mode is on, the evidence match is bypassed. Every ghost is returned as a candidate, and identification leans entirely on the flag layer plus behavioral tells. This is the mode for custom runs that hide evidence.

## Flag layer

Flags are traits and behaviors tied to specific ghosts. Each ghost carries zero or more flags. When a flag gets detected during play, every ghost that owns that flag gets marked as flagged in the results.

### Ghost flag table

Only flags with a confirmed, client-detectable tell are listed. Every other ghost carries `(none)` and relies on the evidence layer alone until its tell is proven live.

| Ghost | Flags |
|-------|-------|
| Aswang | (none) |
| Banshee | BansheeWail |
| Demon | (none) |
| Dullahan | Headless |
| Dybbuk | (none) |
| Entity | (none) |
| Ghoul | CantDisableElectronics |
| Keres | (none) |
| Leviathan | (none) |
| Nightmare | (none) |
| Oni | (none) |
| Phantom | (none) |
| Revenant | (none) |
| Shadow | (none) |
| Siren | SirenHum |
| Skinwalker | SkinwalkerFakeOrb |
| Specter | (none) |
| Spirit | (none) |
| Wisp | Burning |
| Umbra | NoFootStepSounds |
| Vex | InvisibleOnLIDAR |
| Wendigo | (none) |
| Wraith | (none) |
| Ravager | (none) |
| Vesper | (none) |

Only Vex, Ghoul, Dullahan, Wisp, Umbra, Siren, Banshee, and Skinwalker carry flags. All 17 other ghosts rely on evidence alone.

### Flag display names

The flags map to short readable labels for the UI.

| Flag | Label |
|------|-------|
| CantDisableElectronics | Can't Disable Electronics |
| InvisibleOnLIDAR | Invisible on LIDAR |
| Headless | Headless |
| Burning | Burning (Holy Oil) |
| NoFootStepSounds | No Footstep Sounds |
| BansheeWail | Banshee Wail |
| SirenHum | Siren Hum |
| SkinwalkerFakeOrb | Fake Orb (Skinwalker) |

## How flags get detected

Flags come from two sources: attribute probes and behavioral detectors.

### Attribute probes

The game sets attributes on the ghost model. The script reads them by name. Matcha's bulk attribute read returns empty, so the script probes each known attribute name individually with `GetAttribute`, and also tries a bulk read as a catch-all. It probes the ghost model, its primary part, its parent, and its descendants.

The probe list (confirmed to replicate to the client):

```
CantDisableElectronics, InvisibleOnLIDAR, Headless, Burning
```

Plus the state attributes read the same way: `LaserVisible`, `Hunting`, `EventActive`, `Transparency`, and the ghost's `LastEMFLevel5Time`.

A probe counts as a hit when the attribute reads `true` or `1`.

Note: only attributes confirmed to replicate to the client are probed. Server-only flag names are not listed here because they never reach the client under Matcha's polling model.

## Detection status: what currently works

These detectors are confirmed working against the live game (field-tested via the recon cycle, or marked confirmed in the build plan). Everything below reads on the client under Matcha's polling model. Ghosts not listed here identify by evidence alone or are still being worked out — they are not documented as working until they are.

### Confirmed-working client attributes

These attribute probes are verified to replicate to the client and fire reliably:

| Ghost | Attribute | Result |
|-------|-----------|--------|
| Vex | InvisibleOnLIDAR | Deterministic lock. |
| Ghoul | CantDisableElectronics | Deterministic lock. |
| Dullahan | Headless | Deterministic lock (also reads as no Head part). |
| Wisp | Burning | Set true when the ghost walks through burning Holy Oil during a hunt. A non-Wisp is blocked and never gains it. Reliable. |

State attributes that also read live: LaserVisible, Hunting, EventActive, Transparency, and the ghost's LastEMFLevel5Time (skin-proof EMF 5 signal).

### Confirmed-working behavioral and instance detectors

| Ghost | Tell | Notes |
|-------|------|-------|
| Umbra | No GhostFootsteps child script during a hunt | Ships. |
| Aswang | PhotoRewardType == "WitheredFlowers" on the pot (Wither) | Confirmed live. Skin-proof (read the attribute, not the petal part name). Also shows a per-run speed step up after a kill (direction confirmed, needs 3+ players and the stuck-ghost guard). |
| Dybbuk | A corpse in workspace.Ragdolls moves 5+ studs with no thrown projectile near it | Confirmed live at 7 to 7.5 studs. |
| Siren | "- HUMMING -" subtitle with isHunt false | Confirmed live. Delivered via the ShowSubtitle path. |
| Banshee | "> Ghost Wail <" on the subtitle TextLabel | Confirmed live. Positive-only, one wail per hunt, sparse. Narrows to Banshee or Skinwalker (only those two own the wail); split by evidence. |
| Skinwalker | A Ghost Orb inside the ghost's FavoriteRoom in No-Evidence mode | Confirmed live. The orb's camera-only visibility is a render trick; the instance is fully present in the workspace tree and pollable, no camera item needed. |

Inscription via PhotoRewardType == "Inscription" on the Spirit Book is confirmed live for the book path and feeds the evidence layer for the ghosts that carry Inscription.

### Behavioral detectors

The rest come from watching the game over time. Each has its own function:

- No-footstep detection (Umbra): the ghost moves without footstep sounds during a hunt.
- Hunt subtitle detection: reads the subtitle text during a hunt. "wail" or "screech" sets BansheeWail. "humming" or "singing" sets SirenHum.
- No-Evidence orb (Skinwalker): a ghost orb showing up in No-Evidence mode is a fake orb.

Wisp's Burning tell comes from the attribute probe above, not a behavioral watch.

## How results combine

The solver returns a single result each tick with several parts:

- LockedGhost: set when exactly one ghost is an exact evidence match.
- ExactGhosts / PartialGhosts: the evidence match lists.
- FlaggedGhosts / FlaggedDisplayNames: ghosts marked by a detected flag.
- TotalFound: how many evidence types are in.
- HasNegativeEvidence: whether any negative evidence rules a ghost out.

### Skinwalker handling

The Skinwalker is a special case because it mimics other ghosts. These rules handle it:

- In normal (evidence) mode, a Banshee wail also adds Skinwalker as a candidate, since a Skinwalker can fake the wail.
- In normal mode, a Siren hum combined with a ghost orb points to Skinwalker, and Siren gets dropped from that flag match.
- In No-Evidence mode, when a wail or hum is detected, the flagged label shows as "Skinwalker/[Ghost]" to signal the ambiguity.

## Reading the output

Put the two layers together like this:

- One exact evidence match, no conflicting flags: that's your ghost.
- Several evidence candidates, one of them flagged: the flag is your tiebreaker.
- No-Evidence mode: ignore the evidence list, work the flags and behaviors.
- A flag on a ghost the evidence already ruled out: treat it as a possible Skinwalker mimic, not the real ghost.
