# Level 2 power puzzle

## Current installed state — revision 3
The cable puzzle and room-lighting power idea are installed in /Game/Levels/BP_Level02Layout.BP_Level02Layout, used by the MaintenanceLoop actor in /Game/Maps/Level_02. Earlier blocker notes below are historical, not the current installation state.

### Controls and behavior
- R: pick up/drop a nearby cable plug; R at a socket while carrying a plug connects it.
- Four plugs start at the entrance prototype power bank. Their cable sources are four layout components, sampled in world space at startup so the existing sequence streaming offset is respected.
- Four sockets have unique IDs0–3 and are distributed on supported route sections: local (850,780,210), (1380,1300,360), (2690,2360,600), (3290,2750,680) cm. Socket mounts and entrance bank are intentionally greybox/prototype geometry; plugs and sockets use existing industrial meshes.
- One power manager displays0/4–4/4 progress and prevents duplicate connections. Connected cables remain visible and connected plugs cannot be picked up again.
- At BeginPlay, the layout switches off8 main lights,6 route fills,4 bay accents and the exit light. It also sets the8 fixture materials' local Intensity parameter to0 without altering shared materials. The entrance light and4 blue socket guides remain available.
- On the fourth connection the manager's bPowered drives layout bRoomPowered; all those lights restore, and fixture intensity returns to5.5. Gravity does not erase connections or turn off restored room lighting.
- bFansPermitted is exposed as room powered AND normal gravity. NO fan controller, fan rotation or airflow is installed in this revision; that earlier work is deferred.

### Integration and interaction
The room initializes all9 puzzle child actors once, with validity checks and retry while children are unavailable. The layout owns a socket-first/plug-second R-press interaction path using strongly typed puzzle references, overlap checks and existing actor eligibility functions. Each plug ticks after the layout so a deferred drop cannot finish before the room has had a chance to connect at a socket. No global per-frame actor scans are used. The existing character Blueprint was reserved, so it was not edited. The attempted manager interaction rewrite was rolled back in full; the original manager remains in use. Runtime ordering with existing character dispatch still needs gameplay validation.

### Observed verification
- Layout component installation, initialization functions, power/lighting functions and typed interaction helpers compiled cleanly.
- Live editor enumeration confirmed1 Level2 layout,1 manager,4 plugs and4 sockets. It also confirmed1 gravity switch,1 each Float1–4,1 start door and1 end door among its child actors.
- Existing level-root transform was (0,0,0). Door, wall and platform component transforms were not edited for this puzzle.
- Static visual inspection captured the layout preview, entrance instructions/progress display and a socket on its prototype mount. Editor socket labels show NOT INITIALIZED because label IDs, enlarged socket meshes, cable anchors and power visibility are set at BeginPlay; these views do not verify runtime behavior.
- Native saves succeeded for Level_02, BP_Level02Layout and all3 puzzle producer packages. Final label-orientation adjustment compiled cleanly.
- No Play session was started. Carry/drop, snap alignment, cable simulation, fourth-connection lighting, streaming behavior and input ordering are not runtime-tested. Current controls assume local player0 and R, matching the existing mapping.

## Historical design and checkpoint notes

## Canonical brief v1 — approved design
New gameplay scenario from explicit user description; no footage/reference motion was supplied or generated. Unreal centimetres, seconds, +X forward, +Y right, +Z up. Room layout-relative coordinates; streaming offset must remain supported.

### State contract
- 0–3 unique socket connections: main room lights OFF, fan rotors STOPPED, airflow OFF.
- 4 unique socket connections, normal gravity: main lights ON, rotors RUNNING, airflow ON.
- 4 unique socket connections, zero gravity: main lights ON, rotors STOPPED, airflow OFF.
- Frozen or otherwise non-simulating item components receive no fan force; fan logic never enables physics.
- Three wall fans; four physical plugs carried with existing interaction and connected to four distributed sockets. Persistent visible cable per plug. Independent small guidance lighting before power restoration.

### Motion card FAN-01
Source: explicit gameplay scenario, proposed construction values rather than measured footage. Fixed housing, independent rotor pivot at fan hub, local +X airflow/rotation axis. Running angular speed 720 degrees/second; activation/deactivation within 0.25 seconds; repeated loop while enabled. No housing rotation, camera animation, particle effect or generated audio required. Fan force is continuous acceleration applied only to registered Level2 loose-prop components within the bounded airflow lane, with range/strength tunable; initial proposed acceleration 900 cm/s², lane length 4000 cm and half-width/height 250 cm. Calibrate after static size inspection; runtime balancing remains unverified unless actually tested. Frozen items retain physics-disabled state. Fan tick never scans world actors.

### Interaction card CABLE-01
Four identifiable sockets along supported route sections. Use existing interact control to pick up/drop a plug; interact at a socket while carrying a compatible unconnected plug to snap it into place. Repeated interaction cannot count twice. Power restores immediately after the fourth unique connection, no imposed cutscene or timer. Connected plugs remain attached and cannot be counted again. Cable endpoints follow plug motion; cable source stays fixed. No invented motion evidence; carry offset and cable slack are authored gameplay values.

### Work ownership/interfaces — resumed revision 2
User supplied /Game/Fab/Fans_For_Metro/fans_for_metro/StaticMeshes/fans_for_metro.fans_for_metro and authorized completion. It is a combined twin-fan static mesh with integrated blades; keep its housing fixed and overlay separate prototype moving rotors at the hubs. Fan actor in /Game/Levels/Level02Power/BP_WallFan exposes SetFanEnabled(Enabled bool), InitializeTargets(TargetA PrimitiveComponent, TargetB PrimitiveComponent, TargetC PrimitiveComponent, TargetD PrimitiveComponent). Four explicit bindings replace the array-based implementation that failed authoring earlier; the requested gameplay behavior is unchanged. Puzzle actors expose manager bPowered and current carried plug; layout owns explicit child references, initial state and gravity/power synchronization. Producers compile before layout integration. Shared character interaction may receive a minimal socket-priority fix while preserving existing grapple/freeze and legacy interaction behavior.

## Implementation checkpoint — incomplete
Created and compiled cleanly:
- /Game/Levels/Level02Power/BP_PowerManager.BP_PowerManager
- /Game/Levels/Level02Power/BP_CablePlug.BP_CablePlug
- /Game/Levels/Level02Power/BP_PowerSocket.BP_PowerSocket

Manager exposes bPowered, CarriedPlug and RegisterConnection(SocketId). Plug exposes InitializePlug(Manager, SourceLocation world vector), ConnectTo(SocketRoot), and Interact. Socket exposes InitializeSocket(Manager, SocketId) and Interact. Unique IDs0..3 count once. Cable start follows plug; cable end is the fixed world source converted to local coordinates each tick, with100cm slack. Current input handling assumes local player0 and the R key; remapping/touch require adaptation.

Blocked: BP_WallFan was not created. Three bounded authoring attempts failed conversion/compilation on generic/wildcard array-element typing while caching and using physics components. Rejected creation shells were removed; no usable fan asset exists. No further retry was attempted after the bounded failure limit.

Nothing has been integrated into BP_Level02Layout or placed in Level_02 for this request. Existing Level2 lights, gravity, props and layout behavior remain unchanged. No character or legacy puzzle assets were edited. No captures, play tests, fan motion validation or airflow/freeze tests were performed for this request. The three puzzle producers are saved building blocks, not a working installed puzzle.

## Resumption after user-added fan asset
The user authorized full completion after importing the twin-fan static mesh named in revision2. Its thumbnail was inspected. New fan-controller and socket-interaction jobs paused, but resuming either returned: 'The paused sub-agent’s staged project changes are unavailable. Do not continue from this checkpoint.' A fresh fan job then failed because a declared file was already in use. No lock bypass was attempted.

Current observed state: the Level02Power folder still contains only the three saved puzzle producers; no BP_WallFan exists. A fresh character read found no Level02Power/socket-priority integration. No Level02 layout edits were dispatched in this resumption. The supplied fan asset was not changed. Progress is blocked by unavailable staged jobs and an in-use asset lock, not by a diagnosed defect in the supplied mesh. The editing-session blocker must be resolved before authoring can continue safely.
