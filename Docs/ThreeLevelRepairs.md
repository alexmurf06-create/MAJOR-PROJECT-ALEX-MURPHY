# Three-level repairs

Date: 2026-11-19

## Scope
Updated Level_01, Level_02 and Level_03 through their BP_Level01Layout, BP_Level02Layout and BP_Level03Layout Blueprints plus separately placed map actors.

## Delivered
- Replaced unintended perimeter doorway-wall sections with dimension-fitted solid wall panels while preserving entrance and exit openings.
- Added local two-sided material instances under /Game/Levels/WallFix and assigned wall slots across 34, 20 and 15 wall components respectively. Shared source materials and meshes were not changed.
- Removed continuous floor bypass sections and arranged separated elevated platforms approaching each final-door landing. Broken means physically separated sections, not collapsing behavior.
- Retained one entrance GravityChanger per level; removed exit-side duplicates.
- Retained exactly one child actor of Float1 (tank), Float2 (pallet), Float3 (toolbox) and Float4 (drum) in each layout.
- Deleted 12 separately placed map duplicates: Level_01 two, Level_02 four, Level_03 six. Each deletion reported the actor absent afterward.

## Measurements
Layout-local centimetres; intended platform geometry measured from source mesh bounds.
- Level_01: entrance top 0; successive platform tops 160, 240, 320, 400. Horizontal gaps approximately 200, 150, 180, 240.
- Level_02: successive elevated platform tops 180, 330, 480, 570, 650; exit top 700. Minimum edge gaps approximately 50, 180, 320, 255, 200, 140 along the route.
- Level_03: entrance top 0; successive platform tops 220, 380, 550. Horizontal gaps approximately 200, 223.6, 250.
- Level_02 and Level_03 include explicit local collision boxes for supports and walls. Full collision/traversal validation was not performed.

## Sequence integration
BP_SequenceStreaming and BP_SequenceWalkways were not modified. Existing door transforms and room roots were preserved.
- Level_01 exit: (2400,2120,400).
- Level_02 offset: (2800,1900,400); entrance local (0,220,0), exit local (4000,2920,700).
- Level_03 offset: (7200,4600,1100); entrance local (0,220,0), exit local (2800,2100,550).
- Existing first bridge extends approximately X=2400..2800 at top Z=400; second extends approximately X=6800..7200 at top Z=1100. Door endpoints and bridge assets remain unchanged. Runtime streaming/traversal was not tested.

## Verification
- Edited Blueprints reported clean compilation.
- Read-only live actor counts were taken separately in all three loaded editor maps after duplicate cleanup: GravityChanger=1 and Float1=Float2=Float3=Float4=1 in each map. These include layout child actors and standalone map actors.
- Three interior scene captures showed final-door wall coverage and approach platforms.
- Exterior layout captures initially exposed backface visibility; the local two-sided wall material correction was applied afterward. Final captures of each layout showed filled wall backs rather than the earlier open framing. These are sampled-view checks, not an exhaustive all-angle proof.
- Eight still images delivered in total, including before/after views. One additional capture request failed due to an unsupported asset-editor view mode; the corrected standard capture succeeded.
- Some component edit receipts were PARTIAL despite the live component templates containing the edits. Subsequent native saves succeeded for all three Blueprints, all five local material instances and all three maps; final save reported zero dirty map/content packages.

## Limitations
No Play In Editor session was run. Player jump/grapple reachability, floating-object motion, collision at every platform edge, runtime streaming and all-angle wall coverage still need gameplay validation. Scene indexing remained partial; live actor enumeration was used for exact prop counts. Ceiling and platform underside backfaces were outside the wall-material correction.
