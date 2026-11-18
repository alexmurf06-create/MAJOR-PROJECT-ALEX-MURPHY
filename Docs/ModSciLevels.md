# ModSci gravity puzzle levels

## Delivery status

Three independently openable starter game levels were created using existing ModSci floor/wall meshes and their original materials. They have distinct layouts, increasing usable interior dimensions, practical lighting, selected Standable wall/platform components, elevated destinations, and physics props placed above the floor for an initial fall. These are simple scaled-kit prototype layouts, not a fully dressed environment or a runtime-validated puzzle campaign.

The original Tester_level layout was preserved. Project startup/default maps were not changed. The three rooms are now connected through a persistent dynamic-streaming master; there is no full-map travel, level-selection UI, or scripted win condition.

## Combined streaming sequence

Open @/Game/Maps/ModSci_Sequence.ModSci_Sequence for the linked route: Level_01 → Level_02 → Level_03 → finish landing. The existing three maps remain independently openable, now with real entrance and exit openings and door child actors.

The master contains @/Game/Levels/BP_SequenceStreaming.BP_SequenceStreaming and @/Game/Levels/BP_SequenceWalkways.BP_SequenceWalkways. At gameplay startup the coordinator creates real LevelStreamingDynamic instances of all three source maps at origins (0,0,0), (1600,805,300), and (4000,1960,700) cm. It retains them for smooth walkways and backtracking; this small demo preloads all three rather than unloading rooms behind the player. There is no OpenLevel call or per-door teleport.

Rooms are loaded at runtime, not placed as duplicate layout actors in the master editor viewport. Thus the unsimulated master shows its connecting walkways and persistent PlayerStart, not the three streamed interiors. Individual source maps show their full interiors while editing.

The player is movement-locked until all three streaming handles report visible. Then the coordinator initializes normal gravity, synchronizes loaded Float props, places the player once at (250,220,140), stops residual velocity, and restores the existing movement mode. A failed streaming request prints an explicit error and leaves startup locked rather than falsely marking the sequence ready. Gameplay state otherwise stays with the same persistent player; do not expect saved physics state across closing/reopening the master.

### Door rules

@/Game/Levels/BP_StartDoor.BP_StartDoor inherits the shared @/Game/Levels/BP_EndDoor.BP_EndDoor logic with its normal-gravity requirement disabled. Start doors open within 320 cm of a valid player in either gravity mode. End doors require both that proximity and authoritative character bZeroGravity=false; zero gravity closes/locks them. Each update instantly raises the solid leaf by 340 cm to open it or restores its closed position. These are functional prototype doors, not an animated/polished door pass.

Every entrance threshold is at local Z0. Exit thresholds are always higher: Level_01 Z300, Level_02 Z400, Level_03 Z550 cm. Door gaps are nominally 240 cm wide by 390 cm high, with ModSci frames and blocking movable leaves. The original perimeter wall spans were shortened and sill/header sections added so hidden full walls do not obstruct the doors.

The streamed world exit heights are Z300, Z700, and Z1250 cm. Walkway 1 connects X1200–1600, centred Y1025; walkway 2 connects X3600–4000, centred Y2180; the finish landing spans X6800–7200, centred Y4060. Their surfaces sit 0.5 cm below the adjoining platform/floor to avoid coplanar overlap. Side rails and a finish-end rail are included.

### Streaming acceptance checks still required

- [ ] Launch the master and confirm all rooms become visible, startup unlocks, and the initial character/view are correct.
- [ ] Walk through every entrance and exit and across both walkway joins without capsule snagging or falling through.
- [ ] Approach every end door in zero gravity: verify the leaf stays closed and blocks passage; normal gravity opens it on approach.
- [ ] Switch into zero gravity while an exit is open and confirm it closes; check contact behavior if standing in the leaf path.
- [ ] Confirm start doors work in both gravity modes and backtracking stays available.
- [ ] Check the final door leads onto its finish landing.
- [ ] Recheck gravity, floating props, grappling, wall anchors and T-freeze across streamed rooms.

Static verification completed: all seven door/streaming/walkway/layout Blueprints report BS_UP_TO_DATE; all three doorway child pairs and wall cuts were read back; real component previews captured the closed door, all three updated layouts and connecting walkways. A real master-map collision probe found the connector/landing surfaces at Z299.5, Z699.5 and Z1249.5. Live editor inspection confirmed both master actors and its single persistent PlayerStart at (250,220,140).

Static graph/component inspection and compile results are not proof of runtime streaming or door behavior; these checks were not run in Play.

## Maps

| Map | Usable interior | Route | Float1 / Float2 instances | Platform heights |
| --- | --- | --- | --- | --- |
| @/Game/Maps/Level_01.Level_01 | approximately 12 x 12 m | Compact L-route around a divider | 1 / 1 | 130 and 300 cm |
| @/Game/Maps/Level_02.Level_02 | approximately 20 x 16 m | Maintenance rooms and a two-ended loop around a central divider | 2 / 2 | 200 and 400 cm |
| @/Game/Maps/Level_03.Level_03 | approximately 28 x 24 m | Central cargo bay with branching side rooms and staged elevated destination | 3 / 3 | 220, 380 and 550 cm |

Usable dimensions exclude decorative mesh overhang outside the enclosure. These maps intentionally have open tops.

## Layout ownership

- @/Game/Levels/BP_Level01Layout.BP_Level01Layout
- @/Game/Levels/BP_Level02Layout.BP_Level02Layout
- @/Game/Levels/BP_Level03Layout.BP_Level03Layout

Each map contains its matching layout Blueprint at the origin. Edit its components for architecture, lights, Standable tags, the PlayerSpawn child actor, the Scalecube child actor, and the switch support plinth. Physics props are separate actors in the PhysicsProps Outliner folder.

PlayerSpawn is a real PlayerStart child actor, placed 140 cm above the main floor facing the switch. Live editor inspection confirmed an instantiated PlayerStart and GravityChanger child in each map and exactly one PlayerStart in each world. PlayerStart X is 250 cm in Level_01 and 300 cm in Level_02/03; Y is 220 cm. Switch position is X=250/300, Y=500, Z=128 cm, supported by a narrow ModSci plinth.

The temporary standalone PlayerStart and Scalecube in Level_01 were removed after the corresponding child actors were committed. The resulting map directory was checked to confirm removal.

## Gravity and floating

- Each new map explicitly overrides world gravity to -980 cm/s².
- Character NormalGravityScale is now 1 rather than 2. The existing SetZeroGravityEnabled function remains authoritative and is called by GravityChanger.
- @/Game/LevelPrototyping/Float1.Float1 and @/Game/LevelPrototyping/Float2.Float2 have engine gravity enabled by default and explicitly enable gravity/wake their mesh on BeginPlay.
- Their SetFloatingMode(Enabled) function is called for every instance by the character's SyncFloatingProps helper when gravity changes. The actor scans happen on mode changes/startup, not on Tick.
- Entering floating mode disables body gravity, captures its current world position, and applies a transition-only upward velocity-change impulse. Repeated calls with the same mode do not launch it again.
- While floating and simulating physics, each prop applies capped spring-damped acceleration toward a slowly moving target approximately 200 cm above its captured position. Small sinusoidal XY drift and vertical bobbing give autonomous motion without teleporting.
- Restoring normal gravity disables the floating update, enables engine gravity, and wakes the mesh. Existing velocity is not forcibly zeroed, so settling remains physical.
- No floating function enables simulation. T-frozen objects therefore remain frozen until explicitly released. Upon release, they follow the current gravity mode.
- MOVE-BOXES retains its single custom-gravity force of direction x 980 cm/s², with engine gravity disabled to avoid double gravity. Its default direction is now (0,0,-1) rather than zero.
- Static architectural Cube actors remain static; this work does not turn walls or other non-physics geometry into falling props.

The elevated props seen in the editor are intentional authored starting positions. Their falling/settling begins during gameplay, not while viewing an unsimulated editor level.

## Controls

Use the existing interact action near Scalecube to toggle gravity. IA_Interact already has an InputTriggerPressed trigger, so it is configured for one activation per press.

Existing grapple controls remain: hold RMB for eligible object tethering in zero gravity, wheel forward for slack/backward to reel, RMB on Standable surfaces for persistent anchoring, G to unhook, and T to freeze/unfreeze multiple components. See Docs/GrappleControls.md.

Float1 already carried the Standable component tag before this work; that tag is preserved. Direct RMB acquisition on it prioritizes wall anchoring under the existing grapple rules. Float2 has only its Float component tag and remains the straightforward object-tether test prop.

## Floating tuning defaults

Both prop classes use height 200 cm, horizontal drift amplitude 40 cm, bob amplitude 30 cm, phase speed 0.5, spring 2, damping 3, acceleration cap 300 cm/s², and entry impulse speed 120 cm/s. Float1 names the horizontal amplitude FloatHorizontalAmplitude and launch speed FloatLaunchSpeed. Float2 uses FloatRadius and FloatEntryImpulse. These values are initial tunings, not playtested values.

## Verification performed

- Float1, Float2, character gravity integration, MOVE-BOXES, and all three layout Blueprints compiled cleanly after final commits.
- Initial layout-create partial reports were reconciled using their committed component transforms in a subsequent clean batch commit.
- Real Blueprint component previews were captured for all three final layouts.
- Real top-down scene captures were inspected for all three populated maps.
- Real collision surface probes found the main floor at Z=0 and the listed platform heights in every map.
- Placement receipts for all 12 physics props reported intentional elevation above their corresponding floor and no placement overlaps. Cargo's initially failed sixth placement was retried successfully under the indexed PhysicsProps folder.
- Read-only live editor inspection confirmed the three PlayerSpawn/Scalecube child pairs and the expected map prop counts (2, 4, 6; Level_01 count also established by placement receipts/directory).
- Gameplay motion, input feel, player spawn at runtime, puzzle completion, falling/settling, freeze/grapple interaction, and repeated gravity cycles were not tested in Play.

## Runtime acceptance checks still required

- [ ] Open each map independently and confirm the intended first-person character spawns with clear capsule space.
- [ ] At startup all unfrozen simulating props fall and settle instead of remaining suspended.
- [ ] Press interact near Scalecube once: gravity turns off and Float1/Float2 rise and drift without held input.
- [ ] Interact again: gravity returns and those props fall, collide, and settle naturally.
- [ ] Freeze several props with T in both modes; toggle gravity while frozen, then release individual props.
- [ ] Test the preserved independent wall anchor/object tether controls and all wheel directions.
- [ ] Traverse the L-route, the full maintenance loop, each cargo side room, and every elevated destination; tune difficulty/clearance as necessary.
- [ ] Check lighting from the first-person camera and adjust for the project's exposure settings if necessary.
