# Level 3 moving walls

## Intent and implementation (revision 1)
Edit to existing gameplay, based on the user's explicit scenario; no reference footage or generated motion reference required. Level 3 uses `/Game/Levels/BP_Level03Layout.BP_Level03Layout`.

### Motion contract
- Six existing interior sections move: LeftCrossWall, RightCrossWall, LeftRoomLower, LeftRoomUpper, RightRoomLower, RightRoomUpper. Their attached collision boxes move with them.
- Outer walls, doorway structure, floors and platforms stay fixed.
- Units: Unreal centimeters/seconds. Existing LayoutRoot is the local origin; X and Y are layout-local axes. Existing mesh pivots/origins are captured once at initialization.
- `WallTravelDistance = 100`: positive excursion from each original position. Crosswalls travel along local Y; room-wall sections along local X.
- `WallTravelSpeed = 40`: linear speed, hard reversal at endpoints. 2.5 seconds one way; 5 seconds full cycle when uninterrupted. All sections share phase and offset.
- Normal gravity permits motion. Zero gravity and button pause preserve current position and direction. No catch-up after gravity pause.
- Contact queries subdivide travel into steps no larger than 5 cm, sweeping the existing collision-box shape before moving it. Frozen target physics need not be enabled for the query.

### Stop button
- Prototype cylinder cap on the inner WestUpper boundary near the entry, layout-local position (3, 430, 145), diameter 32 cm and thickness 8 cm, pitch 90 degrees.
- Material `/Game/Levels/M_Level03StopRed.M_Level03StopRed`: steady opaque unlit red, linear emissive (0.7, 0, 0). Initial intensity 3 appeared orange under preview tone mapping and was reduced.
- Component tag `Level03StopButton`; projectile impact checks this tag and invokes `TryStopWalls(HitComponent)` on the component's owning Level 3 layout. Layout validates exact component identity.
- Pause duration is 5 seconds of game time. Repeat shots during the active pause are ignored, not stacked or restarted. Countdown continues through zero gravity; global game pause/time dilation follows Unreal game time.

### Frozen object contact
- Player helper `UnfreezeFromMovingWall(Target)` only acts during normal gravity and only for a component in the player's FrozenComponents array.
- It enables gravity, enables physics simulation, wakes the body, removes frozen membership, clears the locked component if it matches, and updates the lock-status boolean.
- Moving-wall queries pass actual contacts to this helper; no queries/thaw actions occur while walls are gravity-paused or button-paused.
- No actor-class scans run per tick. Player reference is cached and reacquired only if invalid.

### Integration
- Player: `/Game/FirstPerson/Blueprints/BP_FirstPersonCharacter.BP_FirstPersonCharacter`.
- Projectile: `/Game/Mechanics/Bp_Projectile.Bp_Projectile`.
- Existing projectile impulse/destruction behavior is retained outside the new tagged-button branch.

### Verification and limits
- Character helper, material, layout and projectile reported clean compilation after their final applied edits. A prior partial layout edit was repaired by the author before integration.
- Material appearance captured in its editor; layout/button appearance inspected separately by supervisor.
- Runtime movement, exact gameplay pause timing, zero-gravity transitions, frozen-body drop and projectile reachability remain untested in a running game. Editor component captures do not establish gameplay behavior.
- Suggested in-game checks: observe full 5-second travel cycle; shoot once and again during pause; toggle gravity mid-pause; freeze object in a wall path, restore gravity and observe contact-driven drop; ensure stationary paused walls do not thaw objects; check entry/exit accessibility and physics response.
