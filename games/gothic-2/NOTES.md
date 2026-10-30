# Gothic II: Night of the Raven: working notes

Started 2026-10-30. Goal: campaign co-op for Gothic II with the Night of the Raven addon,
following [AGENTS.md](../../AGENTS.md). Sister project: [G3 Together](https://github.com/GGM1k3y23/g3-together)
for Gothic 3 ([write-up](../gothic-3/README.md)).

## Triage (phase 0)

| Question | Answer |
|---|---|
| Game | Gothic II: Night of the Raven (2003). Store, version and SHA-256 of `Gothic2.exe`: unknown, to record from the owner's install |
| Engine | ZenGin (shared with Gothic 1). Game logic in Daedalus scripts, run by the engine's parser |
| Source | No engine source. The reverse engineered Union SDK with Gothic UserAPI headers (`ZenGin/Gothic_II_Addon/API`, shipped inside GothicCoop) covers the exe. Union plugins load from `<game dir>\Autorun` or the Steam Workshop |
| Engine multiplayer | None in the game. Existing mods add their own (ENet) |
| Shipping | A Union plugin DLL ships without game files. Steam Workshop already carries Union plugins for this game |

## Existing work (phase 1, first pass)

Observed (read from the repos on 2026-10-30):

- **GothicCoop (Union plugin)**, AGPL-3.0. Lineage: Colder's GitLab project, then
  [Gothic-Workshop/GothicCoop](https://github.com/Gothic-Workshop/GothicCoop) (`main`:
  2025-11-10 "old coop code"; branch `broken-coop`: 2025-12-30, TOML config, world time sync,
  startup guard, builds for G1, G1A, G2, G2A). Releases at
  [DamianQualshy/GothicCoop](https://github.com/DamianQualshy/GothicCoop/releases), Steam
  Workshop item for G2 NotR: https://steamcommunity.com/sharedfiles/filedetails/?id=2982365243.
  - C++, about 4300 lines of own code, ENet 1.3.17, nlohmann-json, Union SDK 1.0l.
  - Syncs: player bodies and animations, nearby NPCs (`VisibleNpcsUpdater`, `LocalNpc`,
    `RemoteNpc`), damage, spell casts, equipment, revive (`RX_Mult_ReviveHero`), world time.
  - Remote players are spawned as an NPC instance (default `CH`) and need the
    zFriendlyFire plugin.
  - Its own README calls it "poorly written". No tests. No quest, dialog, item ledger or
    save sync found in a grep for `quest`, `LOG_`, `Mis_`.
  - Host and client start in game with F1 and F2, direct IP on port 1234.
- **Gothic Multiplayer (GMP) / Gothic Online**, MIT,
  [skejt23/GothicMultiplayer](https://github.com/skejt23/GothicMultiplayer). Persistent
  server world for RPG servers, not the single-player story (see `data/directory.yaml`).

Derived:

- Same starting position as Gothic 3 had: a mod that shows players and fights, but no
  story, quest, item or save sync. GothicCoop is the closest base (path 1, revive and
  extend), and its license (AGPL-3.0) allows that as long as the result stays AGPL.
- The G3 Together design (host-kept ledger for story, items and save; regional NPC
  ownership; bot suites with two real clients) carries over as a plan, not as code.

Unknown:

- Which GothicCoop build actually runs on the current Steam and GOG builds of G2 NotR.
- Whether Workshop Union and a separately installed Union conflict.
- Whether two copies of `Gothic2.exe` start on one PC (needed for bot tests).

## Next steps

1. Record the owner's install: store, build, SHA-256 of `System\Gothic2.exe`, Union version,
   other mods. Copy the install to a test folder, never touch the original.
2. Mine GothicCoop bug reports: GitHub issues of both repos, Workshop comments, the
   World of Players and Gothic Workshop Discord threads.
3. Build `broken-coop` and `main` from source, run two copies, write down what syncs.
4. Decide: extend GothicCoop or start a fresh Union plugin with the G3 Together
   architecture. Write the decision here with the evidence.
5. Open a claim issue upstream so others know Gothic II is being worked on.
