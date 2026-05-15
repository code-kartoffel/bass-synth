
# Modular Sequenced Bass Synth

A browser-based rack-style sequenced bass synthesizer for techno-oriented basslines.

The synth runs as a standalone static HTML file using only HTML, CSS, JavaScript, Canvas, and the Web Audio API. No build step, backend, package manager, or external library is required.

## Overview

This instrument is a compact modular-style bass patch with a fixed internal signal flow:

```text
Clock -> 16 Step Sequencer -> V/OCT Pitch CV -> Bass VCO
Clock -> 16 Step Sequencer -> Gate -> Envelope -> VCA CV

Bass VCO -> Low-pass Filter -> Drive -> VCA/Output -> Scope
```

The sequencer generates 16th-note triggers from a clock. Each step can be turned on or off and assigned a quantized scale degree using a small knob. The default musical setup is:

```text
BPM:   145
Root:  F
Scale: minor
Grid:  16 steps / 16th notes
```

## Features

- Rack-style modular interface
- Red-accent visual theme
- BPM-controlled 16th-note clock
- 16-step sequencer in a 4 x 4 layout
- Per-step on/off buttons
- Per-step quantized note knobs
- Root and scale selection
- Default F minor scale
- Bass VCO with selectable waveform
- Cascaded low-pass filter
- Filter envelope amount
- Drive/saturation stage
- ADSR-style envelope
- VCA/output module
- One-bar waveform monitor
- Sequencer CV and gate monitor
- Moving vertical playhead in both plots
- Random sequence generation
- One-bar playback mode
- Spacebar start/stop shortcut

## Modules

### Clock

Generates the timing signal for the sequencer.

| Control | Description |
|---|---|
| BPM | Sequencer tempo. Default is `145 BPM`. |
| Run lamp | Indicates whether the clock is running. |

Patch points:

| Jack | Direction | Destination |
|---|---:|---|
| CLK OUT | Output | Sequencer clock input |

### 16 Step Sequencer

Creates the rhythmic and melodic pattern.

The sequencer has 16 steps arranged as four rows of four steps. Each step contains:

```text
Step number
Note knob
ON/OFF button
```

| Element | Description |
|---|---|
| Step number | Step position from 1 to 16 |
| Note knob | Quantized scale-degree selector |
| ON/OFF | Enables or disables the step trigger |

The note knob does not select arbitrary frequencies. It selects scale degrees based on the current root and scale.

Patch points:

| Jack | Direction | Destination |
|---|---:|---|
| CLK IN | Input | Clock output |
| GATE | Output | Envelope gate input |
| V/OCT | Output | Bass VCO pitch input and scope CV input |

### Bass VCO

Generates the raw bass oscillator signal.

| Control | Description |
|---|---|
| Wave | Oscillator shape: saw, square, triangle, or sine |
| Octave | Global octave offset for the sequencer pitch |

Patch points:

| Jack | Direction | Destination |
|---|---:|---|
| V/OCT | Input | Sequencer pitch CV |
| OUT | Output | Low-pass filter input |

### VCF

Cascaded low-pass filter used to shape the bass tone.

| Control | Description |
|---|---|
| Cutoff | Base low-pass cutoff frequency |
| Res | Filter resonance |
| Env Amt | Amount of envelope modulation applied to filter cutoff |

Patch points:

| Jack | Direction | Destination |
|---|---:|---|
| IN | Input | Bass VCO output |
| OUT | Output | Drive input |

The implementation uses two cascaded `BiquadFilterNode` low-pass stages. This makes the filter response more pronounced than a single low-pass stage.

### Drive

Adds saturation and harmonic density.

| Control | Description |
|---|---|
| Drive | Tanh saturation amount |
| Tone | Post-drive low-pass tone control |

Patch points:

| Jack | Direction | Destination |
|---|---:|---|
| IN | Input | Filter output |
| OUT | Output | VCA input |

### Envelope

Controls the VCA and also modulates the filter cutoff.

| Control | Description |
|---|---|
| Attack | Fade-in time |
| Decay | Time to fall toward sustain |
| Sustain | Held level while the gate is active |
| Release | Fade-out time after gate-off |

Patch points:

| Jack | Direction | Destination |
|---|---:|---|
| GATE | Input | Sequencer gate output |
| CV OUT | Output | VCA CV input |

### VCA / Output

Final amplitude stage and master output.

| Control | Description |
|---|---|
| Gate Len | Length of each sequencer gate as a fraction of one 16th note |
| Gain | Final output level |

Patch points:

| Jack | Direction | Destination |
|---|---:|---|
| IN | Input | Drive output |
| CV | Input | Envelope CV output |
| MAIN | Output | Scope audio input |

### Scope

Visual monitor for the generated signal.

Displays:

- One-bar audio waveform
- Sequencer pitch CV
- Sequencer gates
- Moving vertical playback position
- Current BPM
- Current step
- Root and scale
- Peak output level

Patch points:

| Jack | Direction | Source |
|---|---:|---|
| AUDIO IN | Input | Main output |
| CV IN | Input | Sequencer V/OCT output |

## Controls

### Main buttons

| Button | Action |
|---|---|
| Start | Starts the clock and sequencer |
| Stop | Stops playback |
| Randomize Sequence | Generates a new bassline pattern |
| One Bar | Plays one bar and stops |

### Keyboard shortcut

| Key | Action |
|---|---|
| Space | Start or stop playback |

The shortcut is ignored when a form control or button is focused.

### Knob behavior

Knobs are controlled by vertical dragging:

```text
Drag up   -> increase value
Drag down -> decrease value
```

This applies to module knobs and per-step note knobs.

## Musical System

The sequencer uses a root note and scale to quantize each step.

Available roots:

```text
C, C#, D, D#, E, F, F#, G, G#, A, A#, B
```

Available scales:

```text
minor
phrygian
dorian
major
chromatic
```

Default setting:

```text
F minor
```

The step note knobs select scale degrees, not raw MIDI notes. This keeps randomized and manually edited patterns musically constrained.

## Signal Flow

### Timing and control voltage

```text
Clock CLK OUT -> Sequencer CLK IN
Sequencer GATE -> Envelope GATE
Sequencer V/OCT -> Bass VCO V/OCT
Sequencer V/OCT -> Scope CV IN
Envelope CV OUT -> VCA CV
```

### Audio

```text
Bass VCO OUT -> VCF IN
VCF OUT -> Drive IN
Drive OUT -> VCA IN
VCA MAIN -> Scope AUDIO IN
```

## Synthesis Details

### Oscillator

The bass oscillator supports:

```text
sawtooth
square
triangle
sine
```

A sine sub-oscillator is also generated internally at one octave below the main oscillator to reinforce the low end.

### Pitch

Pitch is generated from quantized sequencer steps and converted from MIDI note number to frequency:

```js
frequency = 440 * Math.pow(2, (midi - 69) / 12)
```

### Filter

The low-pass filter is implemented as two cascaded low-pass `BiquadFilterNode` stages.

The envelope can push the cutoff upward at the start of each note, creating a punchier techno/acid-style bass articulation.

### Drive

The drive stage uses a tanh waveshaper:

```js
Math.tanh(x * amount) / Math.tanh(amount)
```

This creates soft clipping and additional harmonics.

### Envelope

The envelope uses ADSR-style timing with exponential ramps for real-time Web Audio playback.

### Scope

The scope renders a simulated one-bar buffer based on the current patch and sequencer settings. The moving vertical line shows the current playhead position while playback is running.

## Running Locally

No installation is required.

1. Save the synth as `index.html`.
2. Open `index.html` in a modern browser.
3. Click **Start**, **One Bar**, or press **Space**.

## GitHub Pages Deployment

Use this repository structure:

```text
repository-root/
├── index.html
├── README.md
└── .nojekyll
```

Then enable GitHub Pages:

1. Open the repository on GitHub.
2. Go to **Settings**.
3. Open **Pages**.
4. Under **Build and deployment**, select **Deploy from a branch**.
5. Choose:
   - Branch: `main`
   - Folder: `/ root`
6. Save.

The site will be available at:

```text
https://YOUR_USERNAME.github.io/YOUR_REPOSITORY_NAME/
```

## Browser Requirements

The synth uses:

- HTML5
- CSS grid
- CSS custom properties
- Canvas 2D rendering
- Web Audio API
- `OscillatorNode`
- `GainNode`
- `BiquadFilterNode`
- `WaveShaperNode`

Recommended browsers:

- Chrome
- Edge
- Firefox
- Safari

## Suggested Settings

For a classic techno bassline:

| Parameter | Suggested Range |
|---|---|
| BPM | 130-155 |
| Root | F, F#, G, G# |
| Scale | minor or phrygian |
| VCO Wave | sawtooth or square |
| Cutoff | 250-1200 Hz |
| Resonance | 6-18 |
| Filter Env Amt | 0.35-0.85 |
| Drive | 0.20-0.65 |
| Gate Len | 0.35-0.75 |
| Decay | 100-320 ms |
| Sustain | 0.00-0.20 |

## Current Limitations

- Fixed internal patch routing
- No draggable cables
- No MIDI input
- No WAV export
- No preset save/load
- No swing/shuffle control yet
- No accent or slide per step yet
- Scope is a simulated render of the current patch, not a live audio capture

## Possible Future Improvements

- Step accents
- Step slides/glides
- Swing control
- Pattern save/load
- MIDI clock sync
- MIDI note output
- WAV export
- Preset browser
- Additional filter models
- Multiple oscillator detune
- Pattern length control
- Copy/paste/clear sequence actions

## File

Main application file:

```text
index.html
```

Documentation file:

```text
README.md
```

## License


```text
MIT License

Copyright (c) Wiesław Šoltés

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
