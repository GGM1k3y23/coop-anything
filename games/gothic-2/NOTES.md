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

## Existing work (phase 1, coop-research pass 2026-10-30)

### Story co-op lineage (all AGPL-3.0 Union plugins with ENet)

```
Colder, gitlab.com/gothiccoop/gothiccoop (2023, tags up to 55)
  -> Gothic-Workshop/GothicCoop (main 2025-11, broken-coop 2025-12-30) = DamianQualshy/GothicCoop
       Steam Workshop "Gothic 2 Cooperative" 2982365243 (DamianQ, maintained by "Lee")
       -> corshaegar/Gothic-I-II-Universal-Coop (0.3.1, 2026-10-30)   <- most advanced
       -> DustyCK-png/gothic2-coop-steam (v0.2.1, 2026-10-23)
```

Observed (read from the repos and pages on 2026-10-30, nothing run yet):

- **GothicCoop** ([GitLab](https://gitlab.com/gothiccoop/gothiccoop), last commit 2023-09-15 by
  Colder; [Gothic-Workshop](https://github.com/Gothic-Workshop/GothicCoop), 5 stars, no GitHub
  releases; README on `broken-coop` points to releases at DamianQualshy/GothicCoop).
  - About 4300 lines of own C++, ENet 1.3.17, nlohmann-json (later toml++), Union SDK 1.0l.
  - Syncs player bodies and animations, nearby NPCs (`VisibleNpcsUpdater`, `LocalNpc`,
    `RemoteNpc`), damage, spells, equipment, revive (`RX_Mult_ReviveHero`), world time
    (`broken-coop`). No quest, journal or save sync found in a grep for `quest`, `LOG_`, `Mis_`.
  - Remote players are an NPC instance (default `CH`), needs zFriendlyFire. F1 host, F2 join,
    port 1234. Its own README: "the mod is poorly written".
  - Workshop comments ([page](https://steamcommunity.com/sharedfiles/filedetails/comments/2982365243)),
    player reports: XP for magic kills sometimes missing for the host (2026-09); characters stuck
    in casting, sleeping or dancing animations (fix: F3 or reload); summons not shown to the
    partner; weapon sheathing drops a copy for the partner; no reconnect after a server crash;
    scripted skeleton spawns in a crypt not synced on 1.74; client spell damage higher than
    host. Error strings: `connection to the server failed (port 443)`, `invalid npc instance id`.
    Players also report finishing Guilds 1.6, Archolos and Zombie Valley in co-op "poorly".
    On 2026-10-30 the page showed a "removed for violating Steam Community guidelines" notice.
- **[Gothic-I-II-Universal-Coop](https://github.com/corshaegar/Gothic-I-II-Universal-Coop)**
  (corshaegar, AGPL-3.0, German docs, 0 stars, commits 2026-10-26 and 2026-10-30, version 0.3.1,
  successor of their [Othello-Coop](https://github.com/corshaegar/Othello-Coop)). Targets G2 NotR
  2.6 (main target), G1 1.08k and Othello; Union 1.0m bundled, optional GD3D11 v17.9.7 renderer.
  README claims: host-led world; shared dialogs (one talks, the other listens), journal,
  chapters, quest XP, cutscenes; guild-aware dialogs; host-kept chests, ground items and
  looting; shared co-op chest; shared mission items; players in different worlds with a hidden
  second Gothic process per world; shared save and load with backup at the partner; joint new
  game; IP (UDP 9955) or Steam with friend invites; in-game self test (Shift+F11).
  Its test table: one PC, host plus client, 28/28 guilds and chests, separate worlds Host 48/48,
  Client 45 OK; long runs "occasionally single mismatches"; no two-person test over a real
  network yet. Bug reports: GitHub issues plus `system\GothicCoop.log` from both PCs.
  Code in `Kern/` (`CoopDialogSync`, `CoopSaveSync`, `CoopFarSync`, `CoopHitAuthority`,
  `CoopAutoTest`, `CoopSteam`, ...).
- **[gothic2-coop-steam](https://github.com/DustyCK-png/gothic2-coop-steam)** (DustyCK-png,
  AGPL-3.0, Russian docs, commits 2026-10-21 to 2026-10-23, release v0.2.1-dialogue-fix). A patch
  on "GothicCoop v60" plus a Steam relay (AppID 480). Host-owned saves, shared XP, dialog lease,
  partial quest flag and journal sync. [BUGS.md](https://github.com/DustyCK-png/gothic2-coop-steam/blob/main/BUGS.md):
  quest items, NPC spawns and routines, attitudes and world triggers not synced; latest
  two-client run stopped at a host start timeout.

### Not story co-op

- **Gothic Multiplayer (GMP)**: [skejt23/GothicMultiplayer](https://github.com/skejt23/GothicMultiplayer)
  (MIT, Lua server, 2026-01) and [DamianQualshy/GothicMultiplayer2011](https://github.com/DamianQualshy/GothicMultiplayer2011)
  (active, last commit 2026-10-29). Server worlds, not the campaign. Same author group as
  GothicCoop, so a source of ZenGin networking code.
- **Gothic 2 Online (G2O)**, closed platform for RP servers ([docs](https://docs.gothic-online.com/0.3.4),
  [Nexus](https://www.nexusmods.com/gothic2/mods/14)); Gothic Online: Untold Chapters runs on it.
- **OpenGMP** ([ErrorFlexXx/OpenGMP](https://github.com/ErrorFlexXx/OpenGMP)), older open GMP rewrite, not checked further.
- **Local-Splitscreen** for G2 NotR on World of Gothic (download 725 / 723): pages blocked by bot
  protection, contents unknown.

### searched:

Web (standard and extended) in German, English, Polish and Russian: "Gothic 2 Die Nacht des
Raben Koop Mod Multiplayer Kampagne", "Gothic 2 Night of the Raven co-op mod campaign multiplayer
GothicCoop", "Gothic 2 kooperacja mod multiplayer Noc Kruka coop", "Готика 2 кооператив мод
Ночь Ворона мультиплеер", "Gothic Together coop Union plugin Steam Workshop", "Gothic 2 Local
Splitscreen", "reddit Gothic 2 co-op mod 2026", GitHub repos via web search. Cloned and read:
the five repos above plus the GitLab original, all branches. Not checked: Nexus and ModDB file
tabs, Reddit threads, World of Players and Gothic Workshop Discord, upstream coop-anything issues
and PRs (GitHub pages blocked for fetching, API not available in this session), GitHub forks lists.

Derived:

- Nobody has to start from zero. Gothic-I-II-Universal-Coop already covers most of what G3
  Together had to build (dialogs, journal, host-kept items and chests, shared saves, Steam),
  with an in-game self test. It is four days old and untested by real players over a network.
- Best path is path 1 (finish what exists), on top of Gothic-I-II-Universal-Coop: test it with
  two real clients, harden what breaks, add bot suites in the G3 Together style, and offer
  fixes upstream to corshaegar (contact needs the owner's approval). AGPL-3.0 means our
  changes stay AGPL and public.
- gothic2-coop-steam is a smaller parallel attempt on the same base; its BUGS.md is a useful
  list of hard spots (dialog dispatch in `oCNpc::OnMessage`, XP claims racing damage).

Unknown:

- Which build of each project runs on the owner's G2 NotR (store, 2.6 or 2.6 fix, Union version).
- Whether the corshaegar release assets exist (the GitHub page showed no releases).
- Whether Workshop Union and a separately installed Union conflict.

## Next steps

1. Record the owner's install: store, build, SHA-256 of `System\Gothic2.exe`, Union version,
   other mods. Copy the install to a test folder, never touch the original.
2. Install Gothic-I-II-Universal-Coop 0.3.1 in two test copies, run its Shift+F11 self test,
   then a two-client session on one PC; write down what really syncs.
3. Read corshaegar's `Kern/` code and CHANGELOG; map it against the G3 Together feature list.
4. Decide: contribute to corshaegar, or fork. Write the decision here with the evidence.
5. Open a claim issue upstream so others know Gothic II is being worked on.
