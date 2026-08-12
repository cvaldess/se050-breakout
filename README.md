# SE050 Breakout

A minimal breakout board for the **NXP EdgeLock SE050E2HQ1** secure element, bringing the I²C
interface out to a 5-pin 2 mm header. 16.33 × 12.72 mm, two layers, all passives 0402.

Designed in KiCad 10. The project is self-contained: the SE050 symbol, footprint and 3D model live
in this repository, so it opens on any machine without installing extra libraries.

## Pinout (J1, 1×5, 2 mm pitch)

| Pin | Net | Notes |
|----|------|-------|
| 1 | +3.3V | supply, feeds VIN (pin 12) |
| 2 | SDA | I²C data, 4.7 kΩ pull-up on board |
| 3 | SCL | I²C clock, 4.7 kΩ pull-up on board |
| 4 | ENA | enable — **drive high to use the chip** |
| 5 | GND | |

**ENA is not optional.** R3 pulls it to ground, so the SE050 powers up in deep power-down and will
not answer on I²C until the host drives ENA high. That is deliberate: it gives the host a hard
power switch over the secure element, and deep power-down draws under 5 µA. If you want the chip
always on instead, tie pin 4 to pin 1 on your carrier.

The I²C target address of the SE050 is **0x48**.

## Bill of materials

| Ref | Value | Footprint | Purpose |
|-----|-------|-----------|---------|
| SE050 | SE050E2HQ1 | HX2QFN20 3×3 mm | secure element |
| C1 | 100 nF | 0402 | VIN decoupling |
| C2 | 33 nF | 0402 | VCC/VOUT decoupling |
| C3 | 33 nF | 0402 | ENA filtering |
| R1, R2 | 4.7 kΩ | 0402 | I²C pull-ups to 3V3 |
| R3 | 100 kΩ | 0402 | ENA pull-down |
| J1 | 1×5 header | 2 mm pitch | breakout header |
| H1 | — | M2 | mounting hole, tied to GND |

## Design notes

Worth reading before you reuse this in your own layout — most of it comes straight from the
[SE050 data sheet](https://www.nxp.com/products/SE050) and is easy to get wrong.

**The ISO7816 pins are left unconnected on purpose.** The SE050 also speaks ISO7816 and ISO14443
as a smart card, but that interface is *not enabled* in generic SE050 configurations — NXP enables
it only on customer request (see AN12436, which does not list it for any stock part number). On the
E2 those pads are inert, and the data sheet's own recommendation for pins 3 (IO1), 13 (CLK) and
16 (IO2) is *"n.c. (recommended)"*. Pin 14 (RST_N) is the exception: for that one the data sheet
says *"if not used then connect to VCC or VSS"*, with no floating option, so it is tied here.

Note that IO1 and IO2 double as the **I²C controller** pins (SDA and SCL respectively) — the SE050
acting as a bus master to talk directly to a sensor and sign its readings. The E2 supports that at
up to 1 Mbit/s. If you want it, those are the pads to break out.

**VOUT is a switch, not a regulator.** Pin 15 (VOUT) supplies pin 18 (VCC) through an internal
power switch of about 1.1 Ω, which is why they are tied together here — the arrangement the data
sheet requires for deep power-down to work. In deep power-down VOUT and VCC are switched off
entirely and only the I²C pads stay supplied from VIN.

**The exposed pad is ground.** It is stitched to the ground plane with five 0.6 mm vias (0.3 mm
drill) — four at the corners and one in the centre — leaving 0.185 mm of clearance to the edge of
the 1.95 × 1.95 mm pad. If you have the stencil made, check that your fab tents the via barrels, or
solder will wick down them during reflow.

## Fabrication

Everything the board house needs is in [`gerber/`](gerber/), unzipped:

- Gerbers for both copper layers, both solder masks, both silkscreens and the board outline
- **Paste layers for both sides** (`.gtp` / `.gbp`), for ordering a stencil
- Excellon drill files, plated and non-plated separately, plus drill maps

Two-layer board, 1.6 mm, no controlled impedance, nothing exotic. Copper pours are filled in the
exported gerbers.

> **This PCB has not been fabricated yet.** The circuit itself is in use — the same SE050 wiring
> runs on a carrier board used with Meshtastic firmware and a voice-crypto prototype — but this
> particular layout has not come back from a board house, so treat the first order as a prototype
> run and check it before committing to a batch.

## License

GPL-3.0 — see [LICENSE](LICENSE).

## Repository layout

```
se050_breakout.kicad_pro / .kicad_sch / .kicad_pcb
SE050E2HQ1.kicad_sym                 symbol
SE050E2HQ1.pretty/                   footprint + STEP model
sym-lib-table / fp-lib-table         point at the two above via ${KIPRJMOD}
gerber/                              fabrication output
```
