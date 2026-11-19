# Level 1 and 2 expansion

Approved scope: reuse Lvl_FirstPerson ModSci sci-fi interior language; enlarge Level_01 to 2400 x 2400 cm and Level_02 to 4000 x 3200 cm. Preserve reference map and Level_03 internal content. Keep Q charge, gravity, grapple and freeze controls.

## Reference evidence
Interior captured from eye (-6700,1550,180) toward (-4600,1550,200). Layered gray metal panels, black structural ribs, exposed overhead pipes/trays, orange-red machinery and caution accents, white task lighting, framed industrial machinery bays and ceiling. The initial exterior capture was not interior evidence. Existing MegaLights warning not addressed.

## Shared seams in local cm
- Level 1 nominal bounds X 0..2400, Y 0..2400. Entry (0,220,0), exit (2400,2120,400). Ceiling about Z1000. Preserve startup player (250,220,140).
- Level 2 nominal bounds X 0..4000, Y 0..3200. Entry (0,220,0), exit (4000,2920,700). Ceiling about Z1200.
- Streaming offsets: Level1 (0,0,0); Level2 (2800,1900,400); Level3 (7200,4600,1100).
- Bridge1 X2400..2800, center Y2120, surface Z400; Bridge2 X6800..7200, center Y4820, surface Z1100.
- Unchanged Level3 content receives offset delta (3200,2640,400); move its existing finish landing/rails by same delta.

## Difficulty and Float population
Level1: broad recovery floor and intermediate landings, explicit gravity control, eight Float props total: two of each Float1/2/3/4 (including existing placed actors).
Level2: indirect multi-stage route, elevated alternating staging and tighter landings, twelve Float props total: three of each (including existing actors). Required anchor segments under existing 1500cm acquisition range. Do not invent a timer or completion gate.
Float1 tank; Float2 Standable pallet; Float3 toolbox; Float4 water drum. Preserve float motion; add Float3/4 synchronization to character.

## Delivered build
- Level1 layout: nominal 2400 x 2400cm, ceiling Z1000; 12 recovery-floor panels, 16 ceiling panels, four broad elevated Standable landings, layered wall bays, machinery, trays/braces and pipes. Eight task-fixture meshes; four added mid-height route fill lights. Added three caution decals and one voltage decal.
- Level2 layout: nominal 4000 x 3200cm, ceiling Z1200; 32 floor panels, 16 ceiling panels, four solid recovery underdeck sections, five intermediate 320x300cm Standable landings and a 460x440cm exit landing. Repeated machinery bays, tanks, pipes/trays/braces, task fixtures, six mid-height route fills. Added three caution decals and one voltage decal.
- Both existing layout Blueprint instances remain at location zero, rotation zero, unit scale. Original separate Float actors were retained at their valid existing positions, not duplicated or replaced. ChildActorComponents in the layouts supply the additional props. Live deduplicated count: Level1 two of each Float1..4; Level2 three of each.
- Float3/4 now receive SetFloatingMode(bZeroGravity) from the character's existing SyncFloatingProps path. Q charge and other mechanics were not rewritten.
- Level1 exit gravity control now sits at (2160,2260,400), on its landing, not at its earlier floating Z528. Entry plinth/control retained.
- Level3 internal layout and reference Lvl_FirstPerson were not edited. Only its streaming translation and matching finish walkway were relocated.

## Static and visual checks observed
- Live Blueprint status BS_UP_TO_DATE for Level01Layout, Level02Layout, SequenceStreaming, SequenceWalkways and FirstPersonCharacter; clean compiles reported.
- Native package saves confirmed both modified layout Blueprints, both room maps, character and sequence Blueprint packages. Earlier commit-render discrepancies were reconciled through fresh live component observations followed by successful native saves; no duplicate components were added to retry them.
- Level1 recovery floor: 48/48 downward collision samples supported. Center simple-collision surfaces on the first three landings are about 10.6cm below their mesh-rim design heights; arrival center about Z389.38 versus rim Z400. This is existing mesh relief, not a missing floor.
- Level2 recovery floor: 64/64 samples supported; six landing centers sampled successfully at Z180,330,480,570,650,700.
- Level1 door checks: 50 bidirectional grid rays per door across 240cm width and Z50..440 found only the closed moving leaf, no frame/wall obstruction. Threshold surface about 27.03cm above landing.
- Level2 checks: 50 bidirectional rays per door across 240cm width and Z40..430 found only the closed moving leaf. At the exit Z440 reaches the header/ceiling boundary, so clearance claims exclude that boundary.
- These are sampled static checks, not a capsule walk-through or exhaustive collision proof. Closed door leaves are expected until runtime interaction opens them.
- Intended route center segments Level1 approximately 1064/839/1048cm, Level2 up to1141.3cm, under existing 1500cm grapple acquisition range. This does not prove line-of-sight acquisition or puzzle solvability in play.
- Live walkway components at origin/unit scale. Bridge1 visible bounds approximately X2400.06..2800.00, center Y2120, top Z400; Bridge2 X6800.06..7200.00, center Y4820, top Z1100. Both about320cm wide. Finish landing follows Level3's translation delta.
- Interior evidence: reference 70d5eb47-e092-402b-8027-8f61af2b20e9; improved Level1 reverse view157a759c-6ff7-4242-b834-6361124340ed; improved Level2 view c433f5c3-acb2-4f10-b103-9d9207f49f95. Blueprint component previews also captured for both room layouts.
- Read-only image review PASS: all three surface categories detailed, routes readable, no obvious checkerboard materials or sky gaps. P2 deferred: further localized machinery density and subtle rear-area fill could bring them closer to the reference; no P0/P1 visual defects reported.

## Remaining test limits
No Play session was started. Dynamic streaming startup, door opening, gravity restoration, child-actor physics, tether acquisition, difficulty/balance, Q feel, and an end-to-end traversal are untested. Existing MegaLights warning/render settings unchanged.
Standalone walkway preview was too dark to establish appearance; that editor does not support the requested unlit capture mode. Its geometry and seam coordinates were checked numerically, not visually in the streamed running level.
Actor instance records initially returned containing-World data; no such records were edited as actor bodies. Future actor edits must use a reliable fresh actor view. Current editor map returned to ModSci_Sequence.

