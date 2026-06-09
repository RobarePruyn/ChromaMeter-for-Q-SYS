# Mediacast ChromaMeter

A brand-styleable audio level meter for Q-SYS Designer UCIs. Reusable across
venue audio projects; the first deployment is a soccer training facility, but
nothing venue-specific is baked in.

- **Plugin file:** `ChromaMeter.qplug`
- **Author:** Mediacast Network Solutions
- **Version:** 2.0.0
- **Plugin Id:** `b4e2c8f1-9d3a-4f7e-a5b2-3c1d8e9f6a4b`

---

## How this works (read first)

The most important thing to understand about Q-SYS plugins and UCIs:

> **A plugin does not paint its own appearance onto a UCI.** The plugin's
> `GetControlLayout` only styles the block you see in the Designer
> *schematic*. On a UCI, you place each control individually and style it
> with **CSS**.

And the audio path, which v1 of this plugin got wrong:

> **Audio cannot enter a plugin through a control pin.** Controls declared
> in `GetControls` produce *control* pins. Audio pins exist only via
> `GetPins`, and audio can only terminate inside the plugin at an internal
> component (`GetComponents`) connected with `GetWiring`. This plugin wires
> its **Input** audio pin into an internal **True Peak/RMS Meter**
> (`meter2`) component and reads its levels from the runtime script.

So this plugin is a thin **data engine** plus default CSS class names, and
the compelling, branded look lives in a **UCI stylesheet** (provided below).

| Layer            | Owns                                                        |
| ---------------- | ---------------------------------------------------------- |
| **Plugin**       | The audio path, true peak/RMS levels, the dB readouts, peak hold, the dB range, default CSS class names. |
| **UCI stylesheet** | Colors, zone thresholds, corner radius, fonts — all visual polish. |

> ⚠️ The plugin sets a `CssClass` on each control in its layout so the class
> follows the control onto a UCI. That layout field is **not documented** by
> QSC and may be ignored on some Designer versions — if your controls arrive
> on the UCI without classes, assign the class names manually in the UCI
> editor (per-control **CSS Class Name**); everything else works the same.

The plugin's **Min dB / Max dB** range maps the audio level to the meter's
`0..1` fill. The stylesheet's `:value()` breakpoints key off that same `0..1`,
so the property thresholds and the CSS breakpoints line up.

---

## Installation

Copy `ChromaMeter.qplug` into your Q-SYS user plugins folder:

| OS      | Path                                                     |
| ------- | ------------------------------------------------------- |
| Windows | `%USERPROFILE%\Documents\QSC\Q-Sys Designer\Plugins\`   |
| macOS   | `~/Documents/QSC/Q-Sys Designer/Plugins/`               |

Restart Q-SYS Designer. The plugin appears in the schematic library as
**Mediacast~ChromaMeter**.

---

## Wiring

The plugin block has one mono **audio input pin** named **Input**. Drag a
wire from your audio source to it. Internally it feeds a True Peak/RMS
Meter (`meter2`) component; the script reads that meter's levels and drives
the visible controls.

```
[ Program Bus ]──▶ Input (audio)   [ Mediacast~ChromaMeter ]
```

> The LevelMeter *control* pin on the block is an **output** (the live peak
> in dB as control data, usable elsewhere in the design). Do not try to wire
> audio to control pins — Designer won't allow it, and that misconception is
> exactly what v1 shipped with.

---

## Controls

All controls are user pins and can be placed on a UCI individually.

| Control      | Type                  | Direction | Default CSS class  | Description                                              |
| ------------ | --------------------- | --------- | ------------------ | -------------------------------------------------------- |
| `LevelMeter` | Knob (drawn as Meter) | output    | `meter-bar`        | The visible bar. Script-driven with the true peak level in dB. |
| `PeakDB`     | Text                  | output    | `meter-readout`    | True peak level, e.g. `"-12.4 dB"`.                      |
| `RMSDB`      | Text                  | output    | `meter-readout`    | True RMS level (see probe note below).                   |
| `PeakHold`   | LED                   | output    | `meter-peak-hold`  | Lit while a recent peak is inside its hold window.       |

The **bar is a single draggable control** (`LevelMeter`) — that one element is
your meter on the UCI. It is a Knob control rendered as a level meter (the
same idiom QSC's own Shure MXW plugin uses for its script-driven audio
bars), so its Min/Max give the UCI the dB-to-`0..1` fill mapping. The
readouts and peak-hold light are optional extras you can place wherever you
like.

---

## Properties

| Name               | Type    | Default   | Range        | Purpose                                                            |
| ------------------ | ------- | --------- | ------------ | ----------------------------------------------------------------- |
| Min dB             | double  | -60       | -100 – 0     | Bottom of the meter scale. **Drives the live `0..1` mapping.**     |
| Max dB             | double  | 0         | -10 – +20    | Top of the meter scale. Must be greater than Min dB.              |
| Peak Hold Enabled  | boolean | false     | —            | Enables the peak hold indicator.                                  |
| Peak Hold Time     | double  | 1.5       | 0.0 – 10.0   | Seconds a peak is held before it decays.                         |
| Zone 1 Threshold % | integer | 70        | 0 – 100      | Where the warning zone begins. **Seeds the stylesheet + preview.** |
| Zone 2 Threshold % | integer | 85        | 0 – 100      | Where the clipping zone begins.                                   |
| Zone 1 Color       | string  | `#64CCC9` | hex          | Nominal color (teal). Seeds the stylesheet + preview.            |
| Zone 2 Color       | string  | `#5DCAA5` | hex          | Warning color (green-teal).                                      |
| Zone 3 Color       | string  | `#CB333B` | hex          | Clipping color (red).                                            |

`RectifyProperties` enforces `Max dB > Min dB` and
`Zone 2 Threshold % > Zone 1 Threshold %`.

> **Why are colors/thresholds properties if CSS owns the look?** They keep the
> Designer schematic preview representative and they document the values the
> example stylesheet below is built from. To change the **live UCI** look, you
> edit the CSS — the properties don't repaint the panel by themselves.

---

## The stylesheet (this is where the look lives)

Apply this to your UCI (UCI **Properties → Style**, or a per-control **CSS
Class Name**). It targets the default classes the plugin assigns.

### Primary — fixed color bands (the mockup look)

Uses `-qsc-render-style: layer` with a `linear-gradient` indicator. The three
colors are visible **at once** at fixed heights, and the fill rises over them
bottom-up — the look from the design mockup. The `:value(1)` layer is drawn at
full size and clipped to the current level, so as the signal rises it reveals
more of the gradient from the bottom.

```css
/* ---- The meter bar : fixed bands, fill rises over them ---------------- */
.meter-bar {
  -qsc-render-style: layer;
  background-color: #0D2533;          /* unfilled track                    */
  border-radius: 3px;
  -qsc-meter-indicator-class: meter-fill;
  -qsc-meter-indicator-width: 100vw;  /* indicator fills the control       */
  -qsc-meter-indicator-height: 100vh;
}
.meter-fill:value(1) {
  border-radius: 3px;
  background: linear-gradient(
    to top,
    #64CCC9 0%,                       /* Zone 1 nominal                    */
    #64CCC9 70%,
    #5DCAA5 70%,                      /* Zone 2 warning (starts at 70%)    */
    #5DCAA5 85%,
    #CB333B 85%,                      /* Zone 3 clipping (starts at 85%)   */
    #CB333B 100%
  );
}

/* ---- Peak hold indicator --------------------------------------------- */
.meter-peak-hold {
  background: #FFFFFF;
  border-radius: 2px;
  box-shadow: 0 0 6px rgba(255, 255, 255, 0.7);
}

/* ---- dB text readouts ------------------------------------------------- */
.meter-readout {
  color: #DCDCDC;
  font-size: 14px;
  font-weight: 600;
}
```

**Re-brand** by swapping the three zone hex values. The hard stop pairs
(`… 70%, … 70%`) make crisp bands; for a smooth blend instead, use soft stops:
`linear-gradient(to top, #64CCC9, #5DCAA5 75%, #CB333B)`. The `70%` / `85%`
positions correspond to **Zone 1 / Zone 2 Threshold %**.

> ⚠️ **Firmware caveat — read this.** QSC's known-issues pages document a bug
> where a meter under `-qsc-render-style: layer` "do[es] not render the
> gradient defined in the CSS and instead display[s] the fill color from the
> UCI Control's properties," listed through 10.3 with status **"To be fixed in
> 10.4."** Two honest qualifications:
> 1. "Fixed in 10.4" is QSC's stated *intent* in the 10.3-and-earlier
>    known-issues notes — we could not find a shipped 10.4 release note that
>    independently confirms the fix landed. Verify on your build.
> 2. The documented bug is specifically described for a meter fed by a
>    **Custom Control Generic Integer Knob**. This plugin's bar is a
>    **script-driven Knob rendered as a meter** — structurally close to that
>    case, so assume the bug **can** apply here: test the primary stylesheet
>    on your build, and keep the filmstrip fallback ready. See the "Firmware
>    references" section below.

### Fallback — stepped whole-bar zones (works on 10.3 today)

If the layer gradient does not render on your firmware, this `filmstrip`
version is robust on all current builds. The bar fills bottom-up and
**recolors as a whole** when the level crosses each threshold (teal → green →
red as it gets louder) — not simultaneous bands, but reliable.

```css
.meter-bar {
  -qsc-render-style: filmstrip;
  background: #64CCC9;            /* Zone 1 (nominal) — below 70%          */
  border-radius: 3px;
}
.meter-bar:value(0.70) {
  background: #5DCAA5;            /* Zone 2 (warning) — at/above 70%       */
}
.meter-bar:value(0.85) {
  background: #CB333B;            /* Zone 3 (clipping) — at/above 85%      */
}
```

### Firmware references

- **Layer / filmstrip render styles and the verbatim meter example:**
  [UCI Styles](https://q-syshelp.qsc.com/Content/Schematic_Library/uci_styles.htm)
- **Which CSS properties meters support:**
  [Supported CSS Properties](https://q-syshelp.qsc.com/Content/Schematic_Library/uci_supported_css_properties.htm)
- **The layer-meter gradient bug + "To be fixed in 10.4" note** (appears on the
  10.1 / 10.2 / 10.3 known-issues pages, titled *"Custom Control generic
  integer knob CSS rendering issue"*):
  [Known Issues — Q-SYS Designer 10.1.0](https://support.qsys.com/en_US/known-issues-|-q-sys-designer-software/known-issues-|-q-sys-designer-software-v1010)
  · [9.13.1 LTS known issues](https://qsys.helpjuice.com/en_US/known-issues-|-lts/known-issues-|-q-sys-designer-software-v9131-lts)

---

## Verification / test procedure

1. Drop **Mediacast~ChromaMeter** into a design. (If the design
   fails to compile naming the wiring, see the **meter2 control-name probe**
   note below — the same section covers the wiring pin spelling.)
2. Wire an audio source (oscillator, file player, or live input) to the
   plugin's **Input** audio pin.
3. Open a UCI; drag the **LevelMeter** control onto the panel. Optionally add
   `PeakDB`, `RMSDB`, `PeakHold`.
4. Paste the stylesheet above into the UCI Style (or per-control CSS class).
5. Save and run the design (or emulate).
6. **Check the debug console first.** The plugin prints which internal
   meter controls its startup probe found, e.g.:
   `ChromaMeter: peak level control = 'meter.1'`. A WARNING line
   means the probe missed — see the probe note below.
7. Play audio and confirm:
   - The bar fills bottom-up with level.
   - With the **primary** stylesheet: fixed teal / green / red bands are
     visible and the fill rises over them, crossing into green at 70% and red
     at 85% of the range. If the bar instead shows one flat color, you have
     hit the layer-gradient firmware bug — switch to the **fallback**
     stylesheet (whole-bar recolor) until you are on a build that renders it.
   - `PeakDB` / `RMSDB` show plausible, changing dB values.
   - With **Peak Hold Enabled**, `PeakHold` lights on peaks and clears after
     **Peak Hold Time**.

---

## The meter2 control-name probe (one bench check)

Two facts about the internal `meter2` component are not published in any QSC
documentation we could find, so the plugin is built to fail loudly rather
than silently:

1. **Runtime control names.** The script reads the meter's levels via the
   internal-component global (`level_meter["<control name>"].Value`). The
   exact names (`meter.1`? `peak.1`? `rms.1`?) are undocumented, so on
   startup the script **probes a candidate list and prints the result to
   the debug console**. If you see a WARNING, open the design, drop a
   stock **Meter** component, give it a Code Name + Script Access, run
   `print(Component.GetControls(Component.New("<code name>")))` from a
   scripting component to learn the real names, and add them to
   `PEAK_CONTROL_CANDIDATES` / `RMS_CONTROL_CANDIDATES` at the top of the
   runtime section.
2. **The wiring pin spelling.** `GetWiring` connects `"Input"` to
   `"level_meter Input 1"` (the convention QSC's mixer example documents).
   If the design fails to compile on your Designer version, try
   `"level_meter Input"` (no channel number) in `GetWiring`.

Once verified on the bench, both answers are stable facts — note them in
this README and the probe becomes a no-op safety net.

---

## Limitations / deferred

- **Floating peak dot:** `PeakHold` is a boolean "peak is held" light, not a
  dot that parks at the peak's height on the bar. A floating dot needs custom
  CSS layering and is deferred.
- **RMS fallback:** if the RMS control-name probe misses (see above), `RMSDB`
  falls back to a script-side moving average of the peak level until the
  real control name is added — functional, but not true power RMS.
- **Simultaneous color bands depend on the layer-gradient renderer:** the
  primary stylesheet needs that renderer working (see the firmware caveat).
  Where it is not, the filmstrip fallback gives stepped whole-bar recolor
  instead of fixed bands.
- Horizontal orientation, stereo metering, and network-shared sync remain out
  of scope for v1.

---

## Version history (and the lessons in it)

- **v1 (segmented-LED design, abandoned):** rendered the meter via
  `GetControlLayout` LED segments. Dead end — `GetControlLayout` only styles
  the **schematic block**, never the UCI. The CSS-on-UCI approach replaced
  it.
- **v1.0.0 (control-pin audio, broken):** claimed audio could be wired
  straight into the LevelMeter control's input pin. It cannot — controls
  produce *control* pins; the block shipped with **no audio pin at all** and
  could never meter anything. The earlier `meter2` attempt had been abandoned
  over compile errors and undocumented control names, but `meter2` via
  `GetComponents`/`GetWiring` is the documented idiom (per QSC's own plugin
  framework docs); the right move was to harden it, not avoid it.
- **v2.0.0 (current):** real audio pin → internal `meter2` → script-driven
  bar. The undocumented control names are handled by a startup probe that
  reports through the debug console instead of failing silently, true
  peak/RMS readouts come from the component, text updates are rate-limited,
  and the `PeakHold` LED now actually clears after the hold window.
