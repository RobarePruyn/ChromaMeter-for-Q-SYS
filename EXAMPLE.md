# Example: Dropping the Custom Level Meter into a Design

Assumes `CustomLevelMeter.qplug` is in your Q-SYS Plugins folder and Designer
has been restarted.

## Scenario

A soccer training facility has one program-audio bus feeding the seating bowl.
Operators want a slim, branded level meter on a touch-panel UCI: a bar with
teal / green / red zones and a peak-hold light.

## 1. Place the plugin

1. Open or create a design.
2. In the schematic library, expand **User Plugins**.
3. Drag **Mediacast~Custom Level Meter** onto the schematic. It renders as a
   compact block with a preview bar, a peak-hold indicator, and two dB
   readouts.

## 2. Configure properties

Select the plugin and set, in the Properties pane:

| Property            | Value     |
| ------------------- | --------- |
| Min dB              | `-60`     |
| Max dB              | `0`       |
| Peak Hold Enabled   | `true`    |
| Peak Hold Time      | `1.5`     |
| Zone 1 Threshold %  | `70`      |
| Zone 2 Threshold %  | `85`      |
| Zone 1 Color        | `#64CCC9` |
| Zone 2 Color        | `#5DCAA5` |
| Zone 3 Color        | `#CB333B` |

Min/Max dB define how the audio level maps to the bar's `0..1` fill. The zone
values seed the schematic preview and the stylesheet you will paste in step 5.

## 3. Wire audio in

The plugin block has one mono **audio input pin** named **Input**. Wire your
program bus straight to it:

```
[ Matrix Mixer ]──output 1──▶ Input   [ Mediacast~Custom Level Meter ]
```

No real source yet? Drop a **Tone Generator** or **Audio File Player** and
wire its output to the Input pin for testing.

## 4. Build the UCI

1. Add or open a UCI on a design controller.
2. From the plugin's control panel, drag **LevelMeter** onto the UCI canvas —
   that one control is your meter bar. Size it as a tall, narrow rectangle.
3. Optionally drag **PeakDB**, **RMSDB**, and **PeakHold** onto the panel where
   you want the readouts and peak light.

## 5. Apply the stylesheet

In the UCI's **Properties → Style**, paste the primary stylesheet from the
README. The short version:

```css
.meter-bar {
  -qsc-render-style: layer;
  background-color: #0D2533;
  border-radius: 3px;
  -qsc-meter-indicator-class: meter-fill;
  -qsc-meter-indicator-width: 100vw;
  -qsc-meter-indicator-height: 100vh;
}
.meter-fill:value(1) {
  border-radius: 3px;
  background: linear-gradient(
    to top,
    #64CCC9 0%, #64CCC9 70%,
    #5DCAA5 70%, #5DCAA5 85%,
    #CB333B 85%, #CB333B 100%
  );
}
.meter-peak-hold {
  background: #FFFFFF;
  border-radius: 2px;
  box-shadow: 0 0 6px rgba(255, 255, 255, 0.7);
}
.meter-readout { color: #DCDCDC; font-size: 14px; font-weight: 600; }
```

To re-brand for another venue, change the three zone hex values.

## 6. Run and verify

1. Save the design.
2. Run or emulate (F5).
3. **Check the debug console.** The plugin reports which internal meter
   controls its startup probe found, e.g.
   `Custom Level Meter: peak level control = 'meter.1'`. A WARNING line
   means the probe missed — see the README's "meter2 control-name probe"
   section for the two-minute fix.
4. Play audio. The bar should fill bottom-up; teal / green / red bands sit at
   fixed heights with the fill rising over them; the readouts track the level;
   `PeakHold` lights on peaks and clears after 1.5 s.

## Common pitfalls

- **Design won't compile, error names the wiring.** Designer-version pin
  spelling — open the plugin file and change `"level_meter Input 1"` to
  `"level_meter Input"` in `GetWiring` (see README).
- **Bar shows one flat color instead of bands.** You have hit the documented
  layer-gradient firmware bug (or your control type is affected). Switch to the
  README's filmstrip **fallback** stylesheet for now. See the README "Firmware
  references" links.
- **All dark / never fills, readouts pinned at `-60.0 dB`.** First check the
  debug console for the probe WARNING (undiscovered control names — README
  has the fix). Then confirm the **Input** pin is wired and the source is
  producing signal (compare against a stock **Meter** component on the same
  bus). Check `Min dB` — if it is higher than your actual level, the bar
  never leaves the floor.
- **`PeakHold` never clears.** Confirm `Peak Hold Time` is greater than zero.
  (Steady program at a constant level no longer keeps it lit — the latch only
  refreshes on a *new* high.)
- **RMS readout looks like a smoothed peak.** The RMS probe missed and the
  script is using its fallback; the debug console says so. Add the real
  control name per the README probe section to get true RMS.
