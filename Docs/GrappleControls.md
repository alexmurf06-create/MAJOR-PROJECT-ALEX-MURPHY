# Dual Grapple — Approved Target Behavior

## Status and scope

The controls, independent wall/object states, and gravity-switch integration are implemented. The character and gravity-switch Blueprints compiled cleanly, and committed logic, input mappings, and character tuning defaults were inspected. Runtime behavior and cable appearance have not been tested; all runtime acceptance checks below remain unchecked.

The original Tester_level was not automatically tagged. The new ModSci level-layout Blueprints deliberately tag selected walls and platforms `Standable` for anchoring.

## Project references

- Character: @/Game/FirstPerson/Blueprints/BP_FirstPersonCharacter.BP_FirstPersonCharacter
- Gravity switch: @/Game/LevelPrototyping/GravityChanger.GravityChanger
- Input mapping: @/Game/Input/IMC_Default.IMC_Default
- Grapple action: @/Game/Input/Actions/IA_Grapple.IA_Grapple
- Reel action: @/Game/Input/Actions/IA_GrappleReel.IA_GrappleReel
- Wall unhook action: @/Game/Input/Actions/IA_Unhook.IA_Unhook

## Approved controls

| Input | Intended behavior |
| --- | --- |
| Hold RMB | Acquire and maintain an eligible physics-object rope in zero gravity. RMB replaces the former E grapple mapping. |
| Release RMB | Drop the held object only. Do not release an established wall anchor. |
| Wheel forward/up, positive value | Add slack by increasing the object tether's target length. |
| Wheel backward/down, negative value | Shorten the object tether's target length. Pull the object only when the rope is taut and zero gravity is enabled. |
| RMB on a qualifying `Standable` surface | Anchor to the surface, approach it with the character, and hold there until G or target invalidation. This works independently of gravity mode. |
| G | Unhook the wall only; do not drop a held object. |
| T | Preserve existing multi-item freeze behavior. Frozen objects cannot be dragged. |

`IA_GrappleReel` is an Axis1D action mapped to `MouseWheelAxis`. `IA_Unhook` is a Boolean action mapped to G. `IA_Grapple` uses RMB rather than E.

## Independent object and wall channels

### Object rope

- Object acquisition and dragging require explicit zero-gravity state.
- Eligible props must simulate physics and block the Visibility trace. Frozen objects are ineligible for dragging.
- Holding RMB maintains the object rope; releasing RMB releases only the object channel.
- Wheel input changes target tether length, clamped to the configured minimum and maximum. Positive input adds slack; negative input shortens the tether.
- A slack rope applies no pull. Shortening is not an unconditional force or teleport: pull is allowed only when taut and in zero gravity, using the configured stiffness, damping, and acceleration limit.
- Invalid object targets release the object channel.

### Wall anchor

- A surface qualifies when its Actor Tags or the hit component's Component Tags contain the exact tag `Standable`. Substring or approximate matches do not qualify.
- The tagged surface does not need to simulate physics; it must be traceable for acquisition.
- Wall approach and holding are independent of gravity mode. Releasing RMB does not cancel the wall anchor.
- Approach must respect character capsule collision and obstacles rather than teleporting through geometry. Clearance and arrival tolerance govern the intended stopping position.
- Track moving anchors using a stored local hit point and local offset, transformed with the target, rather than retaining only a fixed world-space destination.
- Hold at the anchor until G or target invalidation. An invalid wall target releases the wall channel. For collision safety, an unrelated obstacle blocking the approach also releases the anchor; if following a moving anchor is blocked by more than 50 cm, the anchor releases rather than pushing the character through geometry.
- Do not automatically add `Standable` or other tags to level objects.

### Concurrent use

Once wall-anchored, the character can independently hold RMB on another eligible physics object in zero gravity. Wall holding and object dragging remain separate states: releasing RMB drops the object without unhooking the wall; G unhooks the wall without dropping the object. Invalidating one target releases its corresponding channel, not the other valid channel.

## Gravity integration

- `bZeroGravity` is the explicit gameplay state; do not infer it solely from a gravity-scale value.
- Expose `SetZeroGravityEnabled` on the character. The gravity switch calls this function to change modes.
- The approved normal-gravity baseline is now character gravity scale **1** for Earth-strength gravity. The three ModSci maps explicitly use world gravity -980 cm/s². Zero-gravity mode disables character gravity.
- The same setter calls `SetFloatingMode` on all Float1/Float2 instances. They start with engine gravity enabled, lift and drift through bounded physics forces in zero gravity, and fall naturally when normal gravity is restored. Frozen bodies are never force-enabled.
- Returning to normal gravity stops any active object rope but retains an existing wall anchor. Wall approach and holding continue independently of gravity.
- The same setter updates `MOVE-BOXES` custom gravity: zero in zero-gravity mode, downward in normal-gravity mode. MOVE-BOXES applies this direction times 980 cm/s² as acceleration, starts with a downward direction, and keeps engine gravity disabled to avoid double gravity.
- Preserve T multi-item freeze behavior across these changes; changing gravity must not make frozen objects draggable.

## Cable presentation versus tension

Cable components provide rope visuals only. Their visible shape, sag, length, or simulation must not be treated as the source of gameplay tension. Object pull is determined independently by target tether length, current endpoint separation, explicit zero-gravity state, and target eligibility. A visible cable does not itself imply active pull.

## Target authored defaults

These authored defaults were confirmed in the character's current class defaults and are consumed by the committed grapple logic. They have not been tuned through runtime testing.

| Parameter | Target default | Meaning |
| --- | --- | --- |
| `GrappleRange` | 1500 cm | Acquisition trace range. |
| `MinTetherLength` | 150 cm | Minimum object tether target length. |
| `MaxTetherLength` | 3000 cm | Maximum object tether target length. |
| `ReelStep` | 100 cm per wheel value | Signed target-length change; positive adds slack. |
| `RopeStiffness` | 20 | Object-rope spring tuning. |
| `RopeDamping` | 8 | Object-rope damping tuning. |
| `MaxPullAcceleration` | 6000 | Object-pull acceleration cap. |
| `WallReelSpeed` | 900 cm/s | Character wall-approach speed target, subject to collision. |
| `WallClearance` | 6 cm | Desired capsule clearance from the anchored surface. |
| `WallArrivalTolerance` | 8 cm | Wall-arrival positional tolerance. |

## Runtime acceptance checklist — not yet verified

- [ ] **Zero gravity off:** object dragging cannot begin; normal character gravity scale is 1; `MOVE-BOXES` custom gravity is downward. Wall acquisition, approach, and holding remain available.
- [ ] **Zero gravity on:** the switch calls `SetZeroGravityEnabled`, `bZeroGravity` reflects the mode, character gravity is disabled, and `MOVE-BOXES` custom gravity is zero.
- [ ] **Hold/release:** hold RMB on an eligible prop to maintain its rope; release RMB to drop only that object. Confirm the former E mapping no longer triggers grapple.
- [ ] **Wheel:** forward/up positive input adds slack; backward/down negative input shortens the target length. Verify both length clamps, no pull while slack, and pull only while taut and in zero gravity.
- [ ] **Surface eligibility:** exact Actor Tag or Component Tag `Standable` permits wall anchoring, including on a non-simulating surface. Untagged and near-match tags do not qualify. No level objects are automatically tagged.
- [ ] **Wall persistence:** release RMB during wall approach or holding; the anchor remains until G or invalidation, in either gravity mode.
- [ ] **Moving wall:** translate and rotate the anchored surface; approach/holding follows its stored local point and offset rather than the original world position.
- [ ] **Obstacles:** place geometry between the character and anchor; capsule-aware approach does not pass through it. Check clearance and arrival tolerance at the surface.
- [ ] **Concurrent channels:** while wall-anchored in zero gravity, hold RMB on another eligible physics object. Verify object manipulation does not replace the wall anchor.
- [ ] **G:** while both channels are active, press G; only the wall unhooks and the object remains held while RMB is held.
- [ ] **RMB release while anchored:** while both channels are active, release RMB; only the object drops and the wall remains anchored.
- [ ] **T freeze:** preserve multi-item freeze; frozen objects cannot be dragged. Confirm freeze behavior with both grapple channels in use.
- [ ] **Switch gravity while wall-anchored:** change zero gravity on/off during approach and holding. The wall remains attached; switching to normal gravity stops the object rope and restores the normal baseline and downward `MOVE-BOXES` custom gravity.
- [ ] **Invalid targets:** destroy or otherwise invalidate each target independently; its channel releases safely without cancelling the other valid channel.
- [ ] **Visual independence:** confirm cable sag or visible slack does not override tension logic or produce pull when the gameplay rope is slack.

Character and gravity-switch compilation passed. Runtime motion, wall holding, wheel feel, and visible rope slack remain unverified. Both cables are hidden until gameplay, so an editor component preview would not validate those effects.
