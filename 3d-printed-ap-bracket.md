# 3D-Printed Access Point Bracket

A mounting bracket for the Cisco Aironet 3802 access points, 3D printed in PETG on a Creality Ender 3 V3. This page covers how to print it, with the most important point first: **the countersunk screw recesses must face up.**

- **Model file:** [`AIR-AP-BRACKET-1.stl`](../AIR-AP-BRACKET-1.stl)
- **Printer:** Creality Ender 3 V3
- **Material:** PETG
- **Supports:** none needed when printed in the orientation below

## Print Orientation (important)

> **Print the bracket flat, with all of the countersunk screw recesses facing up.**

The STL is already modeled in the correct print orientation. Import it into the slicer **as-is and do not rotate or flip it**.

| | Correct | Incorrect |
|---|---|---|
| Face on the print bed | Flat underside | The top face with the screw recesses |
| Countersunk screw recesses | Open at the top, facing up | Facing down toward the bed |
| Supports | None | Needed inside every screw recess |

### Why the recesses face up
- **No supports.** Each countersunk recess is a cone that widens toward the top. Printed upright, every layer is wider than the one below it, so nothing hangs in the air. Printed upside down, the cones become overhangs that need support material inside the recess.
- **Clean screw seat.** The surface a screw head sits against is printed in the open instead of on support material, so it is smoother and seats the screw evenly.
- **Flat, accurate part.** The large flat underside sits on the bed for strong adhesion, which keeps the bracket flat and the holes accurate.

### Check in the slicer before printing
1. The bracket lies flat on the build plate.
2. In the layer preview, **every** countersunk recess opens at the top of the part.
3. Supports are off, and the preview shows none.

## Print Settings

PETG on the Ender 3 V3. These are suggested starting points, so adjust them for your filament and printer profile.

| Setting | Suggestion |
|---|---|
| Orientation | As exported: flat underside on the plate, countersunk recesses up |
| Supports | Off |
| Layer height | 0.2 mm |
| Walls / perimeters | 4 |
| Infill | 30 to 40% |
| Top and bottom layers | 4 or more |
| Nozzle temperature | PETG range for your filament, typically about 230 to 245 C |
| Bed temperature | Typically about 70 to 80 C |
| Part cooling fan | Low to moderate, because high cooling weakens PETG layer bonding |
| Brim | Not normally needed because of the large flat base |
| Scale | 100% |

### PETG tips
- Dry the filament first if it has been exposed to humidity. Wet PETG strings and prints rough.
- Do not set the nozzle too close on the first layer. PETG sticks very firmly to a PEI plate, and too little gap can make the part hard to remove or damage the plate.
- Let the plate cool before removing the part.
- PETG tolerates the heat of access points better than PLA, which is why it was used here.

## Test Fit
Before mounting, check the printed bracket:
- Seat a screw in each countersunk recess and confirm the head sits flush or just below the top face.
- Confirm the other openings line up with the access point.
- If a hole is tight, ream it lightly with a drill bit.

## Assembly Notes
- Keep the face with the countersunk recesses as the visible side when screwing the bracket in place.
- Use countersunk or flat-head screws that fit the recesses.
- Do not overtighten. Printed plastic can crack around the countersinks.
- Mount the access point only after the bracket is firmly fastened.

## Files
| File | Description |
|---|---|
| [`hardware/AIR-AP-BRACKET-1.stl`](../hardware/AIR-AP-BRACKET-1.stl) | Printable model |
