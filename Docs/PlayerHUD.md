# Player HUD Contract

## Status and scope

This document records the implemented HUD integration. Both edited Blueprints compiled cleanly, and the widget designer preview was captured successfully. Runtime behavior remains untested.

Runtime testing is needed. No runtime behavior has been verified.

Scope is limited to HUD presentation and its player-to-widget data contract. No level placement or input binding changes are required or included.

## Confirmed existing implementation

- HUD widget: `/Game/UI/WBP_ShotStrength.WBP_ShotStrength`.
- Owning player character: `/Game/FirstPerson/Blueprints/BP_FirstPersonCharacter.BP_FirstPersonCharacter`.
- The character creates the widget during `BeginPlay` and adds it to the viewport at `ZOrder 10`.
- Existing `ShotCharge` is normalized to the range `0..1` and passed to the widget's `SetStrength` function.
- Q press charges the shot; Q release fires and resets the charge.

These are confirmed integration facts, not evidence of runtime testing.

## Implemented HUD presentation

- A centered aiming plus using screen-center anchors.
- Three 56-unit circular player-status indicators anchored to the bottom-right, above the existing power bar.
- Gravity arrow, torch, and grappling-hook texture icons in `/Game/UI/PlayerHUD`.
- Readable GRAVITY ON/OFF, TORCH ON/OFF, and GRAPPLE IDLE/IN USE labels.
- A dark rounded backing for the bottom-right Q / BALL POWER meter.

Status icons are dim when inactive. Active gravity and grapple icons are cyan. The active torch icon is amber, and a small amber light appears near its tip.

## Implemented widget function

### SetPlayerStatus

| Parameter | Type | Player-provided value | Meaning when true |
| --- | --- | --- | --- |
| `GravityOn` | bool | Inverse of `bZeroGravity` | Gravity is enabled. |
| `TorchOn` | bool | Spotlight visibility | The torch light component is visible. |
| `GrappleActive` | bool | `isTethered` OR `bWallAttached` | The player is tethered or attached to a wall. |

The widget presents supplied status values; the player character remains their source. The function supplements the existing `SetStrength` contract.

`UpdatePlayerHUD` is called immediately after HUD creation and every player Tick after wall-anchor and object-rope updates. It guards the widget reference before calling `SetPlayerStatus`.

## Verification limits

Designer preview showed the aiming plus centered, all three icons and labels readable, and the bottom-right power display without visible clipping. The live scene capture was black with a MegaLights warning, so contrast against room geometry could not be assessed. Rendering settings were not changed. Runtime state transitions have not been observed.

## Runtime testing needed

- Confirm the HUD is created during player `BeginPlay` and displayed at `ZOrder 10` without duplicate instances.
- Confirm the aiming plus remains centered and the three circles remain on the right edge above the bottom-right power bar at supported viewport sizes.
- Confirm `SetStrength` receives normalized charge values and Q press/release retains the existing charge, fire, and reset behavior.
- Confirm `GravityOn` reflects the inverse of `bZeroGravity`.
- Confirm `TorchOn` follows `SpotLight.IsVisible()`.
- Confirm `GrappleActive` is true when either `isTethered` or `bWallAttached` is true, including when both are true.
- Confirm off indicators are dim, active gravity/grapple indicators are cyan, and the active torch indicator tip is amber.
- Confirm status presentation updates when player state changes.

No runtime verification is claimed by this document.

## Subsequent readability review

The player integration was reviewed and left unchanged because UpdatePlayerHUD is already a small, focused function. A proposed split of SetPlayerStatus into three status helpers failed twice with a missing-node error. Both attempts were fully rolled back; the helper functions were not created, and the existing HUD remains unchanged. See Docs/RecentCodeCleanup.md for the applied Level 2 refactors and the outstanding HUD limitation.
