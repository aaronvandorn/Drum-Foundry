# Drum Foundry

An analog-style drum synthesizer for designing one-shot drum samples in the browser and saving them as WAV files. It's a sound design palette, not a drum machine: pick a starting point, shape it, hear it, save it.

Nothing is uploaded; every sound is rendered in your browser.

## How it works

Every hit is rendered sample by sample at the sample rate you choose, then quantized to the bit depth you choose, so **what you hear is exactly what gets saved**: an 8-bit, 22 kHz kick sounds 8-bit and 22 kHz in the preview too. The waveform and spectrum views show the rendered file.

A sound has three parts plus a master section:

- **Tone layer:** two oscillators (sine, triangle, saw, pulse).
  - **OSC 1:** tune (type a note name like `A1` or a frequency), level, pulse width, **PWM** depth and rate, and **Drift** (analog-style random detune and start phase).
  - **OSC 2:** tuned as a ratio of OSC 1, with fine tune, level, its own PWM, **hard sync** to OSC 1, linear **FM** into OSC 1 (a fixed amount plus a decaying **FM envelope** for bark and knock), and **ring mod**.
  - **Pitch envelope:** amount in semitones (up or down), decay, and curve from linear to exponential, applied to OSC 1 and optionally OSC 2.
  - **VCF:** LP12, LP24, BP, HP12, HP24 or notch, with cutoff, resonance up to near self-oscillation, an envelope (in octaves, with its own decay), and pitch tracking so the filter can follow the pitch sweep.
  - **VCA:** attack, hold, decay and curve.
- **Noise & metal layer:**
  - **Sources:** noise (white, pink, brown, or digital, a short shift-register noise), a **metal bank** of six square-wave oscillators at inharmonic ratios (six-square cymbal set, a tighter set, harmonic, two-oscillator bell, glass) with tune and spread, and **stereo** width when saving stereo.
  - **VCF:** its own filter with a separate low cut.
  - **VCA:** attack, hold, decay and curve, plus **bursts** (count, gap, burst length) for claps and rattles.
- **Click:** a short noise or blip transient for attack.
- **LFO:** sine, triangle, square, saw or sample-and-hold, up to audio rate, to pitch, either oscillator, FM depth, pulse width, either filter, the metal bank, or level.
- **Mix & drive:** layer levels, drive (warm, clip, or fold), master low cut and high cut.
- **Lo-fi & length:** bit crush and sample-rate reduction (for the sound, separate from the file format), auto length (trimmed where the sound falls 66 dB below its peak) or a fixed length, end fade, gain, and normalize.

## Working with it

- **Categories:** Kick, Snare, Clap, Closed hat, Open hat, Cymbal, Tom, Percussion, Click & FX, each with several starting points.
- **Audition on change** plays the sound after each edit. **Play** (or the space bar) plays it any time.
- **Knobs:** drag up or right to raise, Shift for fine, double-click to reset. Click the number to type a value. Arrow keys and the wheel work while a knob is focused.
- **Vary** nudges every setting by the Vary amount. **Random** picks a starting point in the category and varies it. **New seed** keeps the settings and rerolls the noise and drift.
- **Undo / Redo** (Ctrl+Z, Ctrl+Shift+Z) cover every change.
- **MIDI in:** press it once to connect; any note plays the sound, with velocity. Press it again to have notes play the sound at their pitch (C4 plays it as designed).

## Saving

- **Output:** 22.05, 32, 44.1, 48, 88.2 or 96 kHz; 8, 16 or 24-bit PCM or 32-bit float; mono or stereo; dither on or off.
- **Save WAV** saves the current sound. Every WAV carries its full patch in a small extra chunk (ignored by other software), so **Load patch or WAV** brings any sound saved here back for editing. **Save patch** saves just the settings as JSON.
- **Kit:** collect sounds with **Add to kit** or **Add 8 variations**, rename them, edit and replace them, then **Save kit** for one .zip of numbered WAVs plus `kit.json`. **Load kit** reads that .zip (or the JSON) back in.
- The current sound, settings and kit are kept in your browser between visits.

## Session recorder

Uses the shared `jam-clips.js` module (the `JAM-CLIPS` block in `index.html`), like the other apps: the last 48 seconds of what you've auditioned can be saved as WAV plus a MIDI file of the hits (General MIDI drum notes on channel 10), and Tape Loop Deck can capture from it.
