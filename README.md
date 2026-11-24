# Throttle override on *early ETCS-i* Lexus and Toyota

1999 Lexus IS200 (Toyota Altezza GXE10, 1G-FE) / Toyota's own repair manual and electrical wiring diagram, with a working account of the Siecle Overtake Booster FA solution and two do-it-yourself alternatives built on the same principle (non-standalone ECU solution).

---

<table><tr><td>

**CONTENTS**

` 1.` [The symptom](#symptom)  
` 2.` [What the car actually has](#system)  
` 3.` [The electrical picture](#electrical)  
` 4.` [Dual-channel safety design](#sensors)  
` 5.` [The ECU's fault envelope](#envelope)  
` 6.` [Where the lag actually comes from](#lag)  
` 7.` [The commercial fix — Siecle Overtake Booster FA](#siecle)  
` 8.` [DIY option A — throttle-body intercept](#diy-a)  
` 9.` [DIY option B — ECU-plug intercept](#diy-b)  
`10.` [Choosing between them](#compare)  
`11.` [Build and validation phases](#phases)  
`12.` [How certain is this?](#confidence)  
`13.` [References](#refs)  

</td></tr></table>

<a id="symptom"></a>

## <sub>§ 1</sub> &nbsp;The symptom

160 PS rear-drive sedan sold on the premise of a tactile, driver-focused experience => single flaw: when the pedal is pressed, the car hesitates. The engine takes a visible fraction of a second to begin responding, and even then the response grows softly rather than sharply.

This is not a mechanical fault, and can be experienced as:
- Rolling onto the throttle from a cruise, where the engine seems to think about the request before acting on it.
- Pulling away from a standstill, where the first increment of pedal travel produces far less motion than a cable throttle of the same era would.
- Partial-throttle overtakes, where the pedal has to be pressed noticeably further than a driver expects to achieve the same acceleration.

The common early assumption is that something is wrong with the throttle cable, the throttle body, or a sensor. On a well-maintained car with no stored fault codes, none of those are the cause. The lag is a *designed property* of the throttle control system itself — a consequence of how Toyota chose to interpret the pedal signal in the first wave of its intelligent electronic throttle, implemented here in a halfway form between pure cable and pure drive-by-wire.

<a id="system"></a>

## <sub>§ 2</sub> &nbsp;What the car actually has

Lexus and Toyota called this early implementation ETCS-i, Electronic Throttle Control System — intelligent. On the IS200 it is a hybrid design: there is still a mechanical cable running from the pedal into the engine bay, but that cable no longer opens the throttle plate. The cable's only job is to rotate a small lever on the throttle body. A rotary sensor reads the lever's angle and reports it as a voltage to the engine computer. The computer in turn drives an electric motor, through a magnetic clutch, which actually rotates the throttle plate.

The result is a system that looks mechanical from the outside but is electrical where it matters.

<p align="center"><img src="figures/fig1-system-flow.svg" alt="End-to-end flow from accelerator pedal to throttle plate" width="100%"></p>

<sub><b>FIG. 1</b> &nbsp;·&nbsp; End-to-end flow of a throttle request. The accelerator cable moves the APPS lever only; the ECU decides a target plate angle from the APPS voltages and drives the throttle motor through a magnetic clutch. The TPS reads actual plate angle back to the ECU as a feedback loop.</sub>

### Why Toyota built it this way

Toyota does not say why it chose the hybrid layout, but its effect is clear. The arrangement kept a physical connection from the pedal into the engine bay, so that if the electronics failed, the cable could still partially open the plate through a mechanical fallback called the *limp mode lever*. The driver would notice reduced power but could get home. An argument could also be made that this solution was made in order to satisfy the idea of a 'luxury sports sedan'.

The arrangement also let Toyota keep what full drive-by-wire gives you: cruise control without a dedicated actuator, traction control that works by closing the throttle, shift-shock reduction on automatics, and the ability to make the throttle feel less sensitive on snow. On the IS200, cruise control is optional (and absent on many lower-spec cars), but snow mode and, on TRC-equipped cars, traction control both act through the throttle.

### The parts, named

- **Accelerator Pedal Position Sensor (APPS).** Mounted on the throttle body, read by the cable. Two outputs: `VPA` and `VPA2`. Called "accel position sensor" and coded **A9** in the manual's connector list, Toyota part number **90980-10711**.
- **Throttle Position Sensor (TPS).** Also on the throttle body, read by the plate shaft. Two outputs: `VTA` and `VTA2`. Called "throttle position sensor" and coded **T2**, same part number **90980-10711**. The connector is identical to the APPS one.
- **Throttle motor.** A small DC motor that drives the plate through a magnetic clutch and reduction gearing. Called "throttle control motor" and coded T1.
- **Magnetic clutch.** Normally engaged while the ECU is powered. If the ECU detects any ETCS fault, it cuts the clutch current and the plate returns to a mechanical rest position via its spring. In this state, the pedal's cable can still move the plate through the limp mode lever — but only partially.
- **Engine ECU.** Everything in the middle. Reads APPS, reads TPS, decides a target plate opening, drives the motor to achieve it, and watches that the plate follows the target.

### The cable is a backup, not a control

This is the point that is easiest to get wrong. The cable does mechanically touch the throttle plate linkage, but through the opener/limp mode lever, so that it only engages when the motor is dead. During normal driving, the motor opens the plate well before the cable mechanism would have, meaning the plate's position is entirely under the ECU's control. A GS400 owner on ClubLexus describes the same arrangement on that car: the cable only reaches the plate near the floor, and the motor does the work in normal driving. The throttle body inspection procedure in the IS200 repair manual agrees: with the ignition on and the engine off, turning the sensor lever by hand makes the motor open the plate to 60% or more.

So the plate's actual opening at any given moment is set by the ECU's target, computed from the APPS voltages. Rewrite those voltages, and the ECU will drive the plate differently. That is the only lever this project has.

<a id="electrical"></a>

## <sub>§ 3</sub> &nbsp;The electrical picture

The accelerator sensor is a four-pin device. Its wires leave the throttle body, pass through the engine harness, and arrive at the engine ECU on **plug E5**. The repair manual (RM684E, diagnostics section) gives the exact pin assignments and voltages, which are reproduced below.

<p align="center"><img src="figures/fig2-apps-wiring.svg" alt="Accelerator sensor wiring to engine ECU plug E5" width="100%"></p>

<sub><b>FIG. 2</b> &nbsp;·&nbsp; APPS to ECU wiring. Source: RM684E, engine diagnostics (DTC P1120 circuit page and ECU terminal voltage table), cross-referenced with the manual's connector part-number list. Wire colors are the Toyota short codes used in the manual.</sub>

### What the ECU reads — the pin voltages

The repair manual's ECU terminal table gives the standard voltages for each pin, measured against E2. These are the numbers your remapping box must stay inside at all times.



| Terminal | Signal             | Wire | Pedal closed | Pedal pressed |
|----------|--------------------|------|--------------|---------------|
| E5-2     | VC (sensor supply) | L-Y  | 4.5 – 5.5 V  | 4.5 – 5.5 V   |
| E5-18    | E2 (sensor ground) | BR   | 0 V (ref.)   | 0 V (ref.)    |
| E5-15    | VPA                | LG-B | 0.3 – 0.9 V  | 3.2 – 4.8 V   |
| E5-16    | VPA2               | L-W  | 1.8 – 2.7 V  | 4.7 – 5.1 V   |



Two things to notice. First, VC is a tightly regulated 5 V supply from the ECU, used by several sensors; your box should *measure* VC rather than assume it is exactly 5.00 V, and all your remapping should be done as a fraction of VC so that your output tracks the ECU's own reference. Second, E2 is not a general chassis ground. It is the ECU's sensor ground, and running significant current through it would offset every other sensor on that line.

<a id="sensors"></a>

## <sub>§ 4</sub> &nbsp;Dual-channel safety design

The accelerator sensor doesn't give the ECU one voltage; it gives two. Both VPA and VPA2 move with pedal position, but they start from different voltages and climb differently. This is a safety redundancy. If one channel fails open, shorts, or drifts, its disagreement with the other tells the ECU the sensor can't be trusted.

<p align="center"><img src="figures/fig3-vpa-vpa2.svg" alt="VPA and VPA2 voltages across pedal travel" width="100%"></p>

<sub><b>FIG. 3</b> &nbsp;·&nbsp; Approximate voltage characteristics of the two pedal signals across travel, drawn to the manual's calibration points. VPA rises roughly linearly from about 0.6 V to about 4.0 V. VPA2 starts near 2.2 V, climbs faster, and flattens as it approaches 4.9 V. The ECU uses the difference between them as a validity check.</sub>

### Why two channels, not just redundancy

If the two channels had the same voltage, a short between the two wires would be invisible to the ECU: both would read identically. By making them start from different voltages and rise at different rates, Toyota ensures that any single-wire failure — short to ground, short to +5 V, short between VPA and VPA2, open circuit on either — produces a signal pair that no healthy sensor could ever produce. The ECU trips a fault and drops to limp mode.

For your remapping box, this means you cannot just scale one channel. You must either:

- remap both channels along their own voltage curves, keeping them in a consistent relationship, or
- remap VPA and then 'regenerate' VPA2 to match the implied pedal position using a learned curve of VPA2 against VPA.

The second approach is cleaner and is what Siecle appears to do, judging by the small signal-processing box embedded in the DCX-T3 harness.

<a id="envelope"></a>

## <sub>§ 5</sub> &nbsp;ECU fault envelope

The repair manual spells out exactly what will cause the ECU to log a fault and cut power to the throttle motor and magnetic clutch. These are the rails your remapping has to run between. The relevant codes are `P1120`, `P1121`, `P1125`, `P1126`, `P1127`, `P1128`, `P1129`, and `P1633`.

For a pedal-signal remapper, only two of them — `P1120` and `P1121` — are directly in reach. The others concern the throttle motor circuit, the magnetic clutch, and the plate feedback, which a good design should never touch.



| Code  | Trigger (held for) | Meaning                                               |
|-------|--------------------|-------------------------------------------------------|
| P1120 | 2.0 s              | VPA ≤ 0.2 V, or VPA ≥ 4.8 V, or VPA2 ≤ 0.5 V          |
| P1120 | 2.0 s              | VPA in low band (0.2–1.8 V) while VPA2 ≥ 4.97 V       |
| P1120 | 2.0 s              | VPA − VPA2 ≤ 0.02 V (channels too close)              |
| P1120 | 0.4 s              | Both VPA ≤ 0.2 V and VPA2 ≤ 1.5 V (both low)          |
| P1121 | 2.0 s              | Difference between VPA and VPA2 outside expected band |
| P1129 | —                  | Plate opening keeps missing its target (ETCS)         |
| P1633 | —                  | ECM internal fault (ETCS circuit)                     |



> [!WARNING]
> **DESIGN RULE**  
> No output ever exceeds 4.70 V on VPA or 4.95 V on VPA2. Both channels must always move together, and the pair must always look like a real pedal. All checks have time windows of 0.4–2 s, so a very brief glitch during a mode switch is tolerated — but a wrong steady value is not.

Fail-safe behavior cuts power to the motor and the magnetic clutch, the plate spring closes the plate, and the car becomes mechanical: whatever cable travel is still useful goes through the limp mode lever, giving a reduced-power limp-home. The MIL (check-engine lamp) illuminates. The code must then be cleared with a scan tool.

<a id="lag"></a>

## <sub>§ 6</sub> &nbsp;Where lag comes from

The ECU converts the pedal reading into a target throttle plate opening through a non-linear map. Toyota's own training material lists eight control modes layered on top of this map: non-linear control, power mode, snow mode, shift shock reduction, idle speed control, TRAC throttle control, VSC coordination, and cruise control. On a manual IS200 without VSC or cruise, the ones that matter are non-linear control, snow mode, idle control and TRC. Shift shock reduction applies to automatics.

<p align="center"><img src="figures/fig4-pedal-map.svg" alt="Pedal position to plate opening map, stock versus remapped" width="100%"></p>

<sub><b>FIG. 4</b> &nbsp;·&nbsp; Stylised pedal-to-plate map. Toyota's calibration keeps small pedal movements small for smoothness and emissions. A remap climbs faster in the lower half of travel, so that 30% of pedal travel asks the ECU for roughly half the plate opening instead of a small fraction. The curves are illustrative; the real stock map has to be measured (phase 1 in §11). At the top end both curves must reach exactly 100% plate — and no further.</sub>

Three mechanisms are at work in the stock calibration:

1.  **A progressive map.** Small and medium pedal travel asks for less plate opening than a cable would. Toyota chose this for smoothness in traffic, for fuel economy, and for emissions. The map is non-linear by design.
2.  **Possibly a rate limit.** If the pedal is stabbed quickly, the ECU may ramp the target plate opening over time rather than letting it snap. This is common in Toyota and Denso systems of this era to reduce jerkiness and intake manifold shock. The IS200 repair manual does not spell out whether this is active or not; it would need a plate-position log to confirm.
3.  **Overrides.** Traction control reduces the plate opening when wheel slip is detected. The snow mode button swaps in an even softer map — UK IS200 owners confirm this on the forums. Shift shock reduction momentarily closes the plate during an automatic gear change, but has little effect on a manual car.

### Flooring it is not where the lag lives

The lag the driver feels is almost entirely in the first 50% of pedal travel. At full pedal the stock map still eventually reaches 100% plate opening. What a remapper really does is reach that opening *earlier* in the pedal travel — i.e. it is a sensitivity change, not a power change. The engine is still a naturally aspirated 2.0 L straight six producing 160 PS, and at low rpm the intake is close to atmospheric pressure well before the plate is fully open, so extra plate opening there adds little torque. Where exactly that point sits on this engine has not been measured.

In other words: the remapper improves response and the *sensation* of acceleration. It does not increase peak power. Owner reviews of Siecle units on Altezzas say the same: sharper response, no extra power.

<a id="siecle"></a>

## <sub>§ 7</sub> &nbsp;The commercial fix — Siecle Overtake Booster FA

Siecle (J-Road) has been building pedal-signal remappers for Japanese cars since 2004. They are the only aftermarket vendor found that lists the GXE10 1G-FE with TRC as a supported vehicle, through their **DCX-T3** harness and **FA-OTB** main unit. The **FAOBD** is an optional power adapter that plugs into the diagnostic socket.

### Topology

The Siecle approach puts the intercept at the ECU end of the harness, not at the sensor. The likely reasons are practical: the ECU is in a cooler, sheltered location (not vibrating on the throttle body), and the ECU connector is in a more serviceable position for routing. Electrically the two intercept points are equivalent, because the four accelerator-sensor wires run directly from the sensor to ECU plug E5 with nothing in between.

<p align="center"><img src="figures/fig5-siecle-topology.svg" alt="Siecle complete system topology" width="100%"></p>

<sub><b>FIG. 5</b> &nbsp;·&nbsp; Siecle's complete install on the GXE10. The DCX-T3 adapter sits inline on ECU plug E5; its small signal-processing module and trimmer live in the engine bay. A red wire splices to ignition 12 V at ECU plug E2 (by the repair manual's layout, E2-16 +B; confirm with a meter). Two signal cables pass through the firewall into the cabin to the FA-OTB main unit. The main unit is powered through the FAOBD at the OBD-II socket, and a mode switch sits on the dash.</sub>

### The DCX-T3 harness

Despite the box on the harness, the DCX-T3 is *mostly* a plug-and-socket adapter. Its female plug clips onto the car's loom-side connector; its male plug clips to the ECU, standing in where plug E5 was. The four accelerator-sensor wires (VC, VPA, VPA2, E2) pass through, but VPA and VPA2 are tapped to a small signal-processing module permanently attached to the harness. That module contains a potentiometer trimmer, which the owner's manual tells you to turn one notch counter-clockwise if full throttle triggers a fault code.

That one detail — a user-accessible trim adjustment specifically for the top-of-travel fault — is strong evidence that the module scales or clips the voltages leaving its output, and that it was tuned by Siecle to sit just inside the fault envelope.

### The FA-OTB main unit

The FA-OTB is a small black box with four connectors and a mode switch. It plugs into the DCX harness's two signal cables, takes constant 12 V from the OBD adapter, and drives a dash-mounted switch with LED. Three modes:

| | |
|:--|:--|
| **Mode 1 · Auto** | A quick stab of the pedal switches boost on; another stab moves to Power Plus with a beep. After 90 s with no pedal input it returns to normal. **Siecle warns this mode may not suit manual cars** — a quick blip during a gear change can trigger it. |
| **Mode 2 · Sport** | The dash switch toggles between Sport and Normal. The recommended mode for a manual GXE10. Each ignition cycle begins in Normal, so one button press per drive activates boost. |
| **Mode 3 · Power Plus** | The dash switch toggles between Power Plus and Normal, with the strongest curve. No audible beep. Also suitable for manual cars. |

The mode choice itself is saved in the unit; it survives key-off. But the boost state (Normal versus active) is *not* saved. Every ignition cycle begins in Normal, and the driver reactivates boost either with the switch or (in Mode 1) with a pedal gesture. This is almost certainly a deliberate safety choice: cars begin in their stock state until the driver asks for something more.

### What Siecle doesn't do

The unit cannot increase peak power, cannot modify fuel or ignition, cannot change the TRC behaviour, cannot bypass the fault envelope, and cannot be reprogrammed by the owner. It is a well-executed, single-purpose signal remapper. For about €280 landed in Europe, it is the shortest path to the effect; the DIY alternatives in the next two sections exist for owners who want to understand what they're doing, pay less, or add their own features (logging, telemetry, remembering the last mode on key-on, custom curves).

<a id="diy-a"></a>

## <sub>§ 8</sub> &nbsp;DIY option A — throttle-body intercept

This is my recommended DIY approach for a one-off build. It differs from Siecle only in where it taps in: at the sensor end of the four signal wires rather than at the ECU end. Electrically it is identical, and the ECU cannot tell the difference. Practically, it is much easier because the four-pin accelerator sensor connector (Toyota 90980-10711) is a stock service part — you can buy pigtails and mating plugs on eBay and from motorsport electrical suppliers.

### Overall arrangement

<p align="center"><img src="figures/fig6-diy-a-topology.svg" alt="DIY option A — throttle-body intercept topology" width="100%"></p>

<sub><b>FIG. 6</b> &nbsp;·&nbsp; DIY option A topology. A Y-adapter sits at the throttle body's APPS plug. VC and E2 pass straight through; VPA and VPA2 are cut, with the sensor halves going up to the DIY box and the ECU halves coming back from it. A six-core shielded cable through a firewall grommet runs to the box in the cabin, where a bypass relay restores the stock wiring whenever the system is off or the MCU is unhealthy.</sub>

### Connectors

The IS200 1G-FE accelerator pedal position sensor uses **Toyota 90980-10711**, a sealed 4-pin plug from the Sumitomo TS series. The same connector is used on the throttle position sensor on the same throttle body — and aftermarket suppliers sell it as the 2JZ throttle position sensor plug, so pigtails are easy to source. (The 2JZ-GE *accelerator* sensor uses a different plug, 90980-11150.)

You need both halves of the Y-adapter:

- **Female pigtail (plugs onto the sensor):** buy as a ready-made pigtail or as a housing + Sumitomo TS terminals + wire seals. Ballenger Motorsports sells a kit explicitly for Toyota 90980-10711.
- **Male pigtail (the car's original plug clips onto this):** the mating half of the same connector, not a Toyota service part. Order it from a Sumitomo TS supplier, or buy an *extension pigtail* — a male-to-female cable — and cut it in the middle, which gives you both halves and some ready-crimped wire.

> [!WARNING]
> **KEYWAY CAUTION**  
> The APPS and TPS plugs on the throttle body look identical. Label everything before unplugging anything. If you cross them, the ECU reads plate angle as pedal position and the car goes straight into limp mode. The way to tell them apart: APPS wires are Brown, Blue/White, Light-green/Black, Blue/Yellow. TPS wires are Brown, Red/Black, White/Red, Blue/Yellow. The middle two colours differ.

### Electrical schematic

<p align="center"><img src="figures/fig7-diy-schematic.svg" alt="DIY interceptor schematic — input stage, relay bypass, DAC output" width="100%"></p>

<sub><b>FIG. 7</b> &nbsp;·&nbsp; Block schematic of the DIY interceptor. VC and E2 pass through untouched. VPA and VPA2 are read by the MCU through a high-impedance input stage, remapped, and output through a 12-bit DAC and rail-to-rail op-amp buffer. A DPDT signal relay provides a hardwired bypass: NC contacts carry the raw sensor signal to the ECU whenever the relay is off, which happens on power loss, watchdog timeout, selected Normal mode, or detected input fault.</sub>

### Fail-safe behavior

The bypass relay is the heart of the safety design. It is a two-pole signal relay with the following discipline:

1.  **Default state is bypass.** With the relay unpowered, both poles carry the raw sensor signal straight to the ECU. The car is in its stock condition.
2.  **The coil is energised only when all of:** power is good, the MCU's watchdog pin is being toggled regularly, the mode switch is set to active, and the input readings look sensible.
3.  **Loss of any of those drops the relay.** Switching time is on the order of 5 ms — two orders of magnitude shorter than the shortest ECU fault window (0.4 s), so the switching glitch itself cannot cause a limp mode.
4.  **Hardware clamp on the DAC output.** Belt and braces: a Zener or precision reference holds the DAC output below 4.70 V in hardware. Even if firmware tries to send 5 V, it physically cannot.

### Firmware approach

The MCU runs a loop at approximately 1 kHz. Each cycle:

    // Pseudocode
    loop forever at 1 kHz:
        vc    = read_adc(VC_CH)         // measure 5V ref live
        vpa   = read_adc(VPA_CH)
        vpa2  = read_adc(VPA2_CH)

        // Convert to pedal fractions of VC
        p_vpa  = (vpa  - cal.vpa_closed)  / (cal.vpa_open  - cal.vpa_closed)
        p_vpa2 = (vpa2 - cal.vpa2_closed) / (cal.vpa2_open - cal.vpa2_closed)

        // Plausibility: both channels agree within tolerance
        if abs(p_vpa - p_vpa2) > 0.08 or vpa > 4.75 or vpa2 > 4.95:
            drop_relay()           // let the ECU see the real sensor
            set_fault_led()
            continue

        // Below ~2% pedal, pass through for clean idle learning
        if p_vpa < 0.02:
            out_vpa  = vpa
            out_vpa2 = vpa2
        else:
            p_new = curve[current_mode].apply(p_vpa)   // remap
            out_vpa  = cal.vpa_closed  + p_new * (cal.vpa_open  - cal.vpa_closed)
            out_vpa2 = cal.vpa2_closed + lut_vpa2(p_new) * (cal.vpa2_open - cal.vpa2_closed)

        // Hard clamps
        out_vpa  = min(out_vpa,  4.70)
        out_vpa2 = min(out_vpa2, 4.95)

        dac_write(0, out_vpa)
        dac_write(1, out_vpa2)
        kick_watchdog()

The `cal` values come from a one-time pedal-sweep calibration (identical to Siecle's initial setup). `lut_vpa2` is the learned curve of VPA2 as a function of VPA, captured during the same sweep. The remapping curves themselves can be a simple set of modes (Normal, Mild, Sport, Power) or interpolated from a user-editable table over USB.

### Parts list

| | |
|:--|:--|
| **MCU** | STM32 "Black Pill" or RP2040 (Pi Pico). Either is enough. The onboard 12-bit ADC is adequate with oversampling; compute every reading as a fraction of the measured VC. |
| **DAC** | MCP4822 dual 12-bit DAC over SPI. Internal 2.048 V reference, usable range doubled with its internal 2× gain. Followed by an OPA2171 or similar rail-to-rail op-amp running from an 8 V rail. |
| **Relay** | Omron G6K-2P or Panasonic TQ2-5V DPDT signal relay, gold contacts, with a flyback diode and a transistor driver gated by the watchdog AND the mode switch state. |
| **Power** | Automotive-rated buck regulator (e.g. TPS54232-Q1) for 5 V, LDO for 3.3 V, small boost converter for the 8 V op-amp rail. 1 A blade fuse, SMAJ18A-class TVS, Schottky reverse-polarity diode. |
| **Enclosure** | Sealed ABS or aluminium box, mounted in the footwell. All cable entries through strain-relieved cord grips. Switch and LED on a separate flying cable to the dash. |
| **Cable** | 6-core shielded 0.25–0.5 mm², automotive rated. VPA and VPA2 twisted with their returns if possible. Shield earthed at the box end only. |

Total parts cost is in the €40–80 range depending on sourcing. The design is deliberately conservative: every failure mode the design anticipates defaults to the stock wiring. It still needs the phased testing in §11 before it is trusted on the road.

<a id="diy-b"></a>

## <sub>§ 9</sub> &nbsp;DIY option B — ECU-plug intercept

This is the direct equivalent of Siecle's own approach. The same interceptor box lives in the cabin, but the intercept point is at ECU plug E5 rather than at the APPS plug on the throttle body. The electrical schematic is unchanged; only the harness differs.

<p align="center"><img src="figures/fig8-diy-b-topology.svg" alt="DIY option B — ECU-plug intercept topology" width="100%"></p>

<sub><b>FIG. 8</b> &nbsp;·&nbsp; DIY option B topology. The interceptor box is identical to option A; only the Y-adapter changes, now plugging into the ECU's 24-pin E5 connector. The harness between the throttle body and the ECU is untouched.</sub>

### Pros and cons compared to option A

Electrically, option B is identical. Thermally and mechanically, it is slightly better: the ECU lives in a cooler, less vibration-prone spot than the throttle body, so the Y-adapter's crimps and seals have an easier life. Routing is marginally shorter.

Against it: the connector is harder to source. Toyota does not sell the ECU plug or its mating socket as a service item. A workable supply chain is to buy a scrap 1G-FE engine ECU plus a short section of its loom from a breakers' yard, extract the E5 plug and its loom-side mate, and re-pin them with new terminals. For a one-off build this is not worth the trouble; Siecle has already done it at scale with their DCX-T3, which is the simple way out if ECU-end interception is a requirement.

In practice the reason to choose option B over A is small: unless you have a specific reason to prefer the ECU location, the throttle body connector is the cleaner DIY choice because its mating hardware is a commodity.

<a id="compare"></a>

## <sub>§ 10</sub> &nbsp;Choosing between them



| Dimension                | Siecle FA-OTB + DCX-T3       | DIY option A (TB)             | DIY option B (ECU)            |
|--------------------------|------------------------------|-------------------------------|-------------------------------|
| Cost delivered           | ≈ €280                       | €40 – 80                      | €50 – 120                     |
| Time to install          | 1 – 2 h                      | weeks (design + build)        | weeks + sourcing              |
| Intercept location       | ECU plug E5                  | APPS plug (throttle body)     | ECU plug E5                   |
| Connector availability   | DCX-T3 (made-to-order)       | 90980-10711, off the shelf    | Scrap-yard harvest            |
| Firmware editable        | No                           | Yes (USB)                     | Yes (USB)                     |
| Logging                  | No                           | Yes (adds a tap on VTA)       | Yes                           |
| Remembers mode at key-on | No (always starts Normal)    | Design choice                 | Design choice                 |
| Proven on this exact car | Yes (Siecle fitment)         | Requires validation           | Requires validation           |
| Risk if poorly executed  | Minimal (engineered product) | Fault / limp mode until fixed | Fault / limp mode until fixed |



For an owner who wants the result quickly and reliably, buy Siecle. For an owner who likes to understand what their car does and would enjoy a working MCU project with a real-world payoff, build option A. The two outcomes are indistinguishable from the driver's seat.

<a id="phases"></a>

## <sub>§ 11</sub> &nbsp;Build and validation phases

Whichever path you take, do not energise the remapping on the first day. The failure modes to design against are not catastrophic (the ECU is defensive and will drop to limp mode cleanly), but they produce stored codes and warning lights that require a scan tool to clear. The following phased approach catches issues before they become shop visits.

1.  **Phase 0 — measure.** With a multimeter, back-probe VPA, VPA2, and VC at the APPS plug with ignition ON. Confirm they match the manual's spec. Note the exact closed-pedal and full-pedal voltages, which will become your firmware calibration.
2.  **Phase 1 — log, do not intercept.** Build the interceptor as a listener only: wire it in with the relay permanently de-energised, so every signal passes through untouched. Add a high-impedance tap on the TPS plug (VTA) at the same throttle body. Record VPA, VPA2, and VTA while driving under a variety of conditions — standing starts, rolling throttle, part-pedal cruise, snow mode on, snow mode off. This gives you empirical pedal-to-plate behavior for your car, which is what we have been inferring from Toyota's documentation.
3.  **Phase 2 — bench test.** Enable remapping on the bench with a 5 V supply and a spare APPS sensor (easy to source) in place of the car. Drive the pedal manually and verify on a scope or multimeter that the outputs stay inside the fault envelope across the entire range of input conditions you can simulate. Verify that cutting power, setting the mode to Normal, or asserting the fault LED drops the relay as expected.
4.  **Phase 3 — in-car, ignition on, engine off.** Enable a mild curve (e.g. 1.2× multiplier below 50% pedal). Press the pedal slowly and quickly, cycle the mode switch, pull the fuse. After a few minutes, check for stored codes with an OBD scanner. If any P1120, P1121, P1125, P1127, P1129, or P1633 appears, drop back to the bench.
5.  **Phase 4 — engine on, quiet road.** Drive at low speed in a safe area with the switch in reach. Normal is your instant hardware bypass. Gradually enable stronger curves and check that the car behaves normally through gear changes, hill starts, and cruise.
6.  **Phase 5 — live with it.** After a few weeks, you'll either leave it in a favourite mode permanently, or realise that you prefer the stock calibration for 90% of driving and only switch to Sport when you want it. Either answer is fine.

<a id="confidence"></a>

## <sub>§ 12</sub> &nbsp;How certain is this?

Not every statement here carries the same weight. This table separates what comes straight from documents from what is inference.

| Claim | Confidence | Basis |
|:--|:--|:--|
| APPS pinout, wire colours, ECU terminals E5-2/15/16/18 | **High** | RM684E ECU terminal table and DTC P1120 circuit page |
| VPA / VPA2 voltage ranges (closed and pressed) | **High** | RM684E terminal table |
| P1120 trigger conditions and time windows | **High** | RM684E DTC P1120 page |
| Connector 90980-10711 for both APPS (A9) and TPS (T2) | **High** | RM684E connector part-number list |
| Motor, not cable, drives the plate in normal running | **High** | Manual inspection procedure (lever by hand → plate opens ≥ 60%) |
| Siecle DCX-T3 fits GXE10 with TRC and plugs in at ECU plug E5 | **High** | Siecle fitment table and DCX-T3 instruction sheet |
| FA-OTB modes, start-up in Normal, wire colours | **High** | Siecle FA-OTB manual Vol. 2 |
| DCX-T3 red wire goes to **E2-16** | **Medium** | Read off Siecle's drawing against the manual's plug layout; confirm with a meter |
| Exact *shape* of the VPA/VPA2 curves (Fig. 3) | **Medium** | End points are from the manual; the curve between them is drawn, not measured |
| Stock pedal-to-plate map (Fig. 4) | **Low · illustrative** | Toyota documents describe a non-linear map but give no numbers |
| Whether the ECU also rate-limits the plate | **Unknown** | Needs a logged drive (phase 1) |
| What the DCX-T3 box does internally | **Inferred** | Trimmer for full-throttle faults implies clamping or scaling; not opened or measured |
| DIY circuit and firmware | **Design proposal** | Untested; follows the manual's limits but needs the phased validation in §11 |
| Part costs and landed price | **Estimate** | Retail listings at time of writing; varies |

<a id="refs"></a>

## <sub>§ 13</sub> &nbsp;References

### Primary sources

- Toyota, *Lexus IS200 Repair Manual, RM684E*, 01/1999 — engine diagnostics (ECU terminal voltages, ETCS-i DTC pages P1120–P1633), fuel injection (throttle body inspection), and the connector part-number list.
- Toyota, *Lexus IS200 (GXE10) Electrical Wiring Diagram*, 1999 — Engine Control circuit, APPS and TPS connector views.
- Siecle (J-Road), *DC Connector Instruction Manual (DCX-T3)*, Vol. 5.
- Siecle (J-Road), *Overtake Booster FA Instruction Manual*, Vol. 2.
- Toyota of Canada, *ETCS-i System Training Material*.
- US Patent 5,950,597 (electronic throttle control with fail-safe operation).

### Secondary sources

- 2JZGARAGE, *ETCS-i / DBW System Troubleshooting*.
- Lexus Owners Club (UK), threads on IS200 acceleration lag, snow button behaviour, ECU pin voltages.
- ClubLexus, threads on GS300/GS400 APPS behaviour and limp-mode throttle authority.
- Minkara, user reviews of Siecle TREC and Response Booster series fitted to GXE10 and SXE10 Altezzas.
- PartSouq VIN decode of JTDBS182600025726 confirming GXE10L-AEFVKW, LHD, manual transmission, 1G-FE, June 1999.

### Vendor references

- Ballenger Motorsports — 4-way SSC TPS plug kit for Toyota 90980-10711.
- Siecle (J-Road) — FA-OTB (JAN 4589816380545), DCX-T3 (made-to-order), FAOBD (JAN 4589816382914).
- Nengun, Buyee (JDirectItems) — forwarding routes for Japanese Siecle sales.

This document synthesizes publicly available technical documentation for the 1998–2005 Toyota Altezza / Lexus IS200 (GXE10) with 1G-FE engine. It is not affiliated with Toyota, Lexus, or Siecle (J-Road). Any work on a car's throttle control system carries risk; the author assumes none of it. Verify every voltage, pin, and procedure against your own vehicle before modifying it.
