# Arrangement format

**Draft 0.1. This may change before the write API ships.**

An arrangement is a whole multitrack project written as one JSON file: tempo, a loop region, and a list of tracks carrying MIDI notes or audio clips, with their instruments, effects and mix. It's meant to be read as much as run. Someone opening the file should be able to see how the track was put together.

InternetDJ already builds projects from arrangement files internally. [Ridiculous Pump](https://internetdj.co/public/37) was composed this way, and its file is in [`examples/`](examples/ridiculous-pump.json). Once the write API ships, you'll be able to send a file like this and get back a project that opens in the editor like any other.

- Schema: [`arrangement.schema.json`](arrangement.schema.json) (JSON Schema 2020-12)
- Example: [`examples/ridiculous-pump.json`](examples/ridiculous-pump.json)

Add the schema to the top of a file to get autocomplete and checking in most editors:

```json
{
  "$schema": "https://raw.githubusercontent.com/mbordash/internetdj-developers/main/arrangement-format/arrangement.schema.json",
  "formatVersion": "0.1",
  "title": "My track",
  "bpm": 124,
  "tracks": []
}
```

## Time

Every time in an arrangement is in **timeline units**. One unit is one second at 120 BPM, so:

| | Units, at any tempo |
|---|---|
| a 16th note | 0.125 |
| an 8th note | 0.25 |
| a beat | 0.5 |
| a 4/4 bar | 2 |

Changing `bpm` speeds the whole project up or slows it down without rewriting any times. If you're generating a file in code, define `BEAT = 0.5` and `BAR = 2` and write everything in terms of those.

## Top level

| Field | | |
|---|---|---|
| `title` | required | |
| `bpm` | required | Whole number, 60 to 240. |
| `tracks` | required | At least one. |
| `loop` | | `{ start, end, enabled }` in timeline units. |
| `swing` | | 50 (straight) to 75. Write notes straight; swing is applied on playback. |
| `swingGrid` | | `16` (default) or `8`. |
| `master` | | Master `volume` (0 to 2), `limiter` and `eq`. |
| `returns` | | Up to 4 shared effect buses that tracks send to. |

## Tracks

Every track has a `name`, unique within the file, and a `type` of `midi` or `audio`. Both kinds take `volume` (0 to 1), `pan` (-1 to 1), `swing`, `effects` and `automation`.

### MIDI tracks

```json
{
  "name": "Bass",
  "type": "midi",
  "instrument": "synth",
  "polyphonic": false,
  "volume": 0.3,
  "synth": {
    "oscillator": { "type": "sawtooth" },
    "envelope": { "attack": 0.005, "decay": 0.1, "sustain": 0.9, "release": 0.1 }
  },
  "notes": [
    { "note": "F2", "start_time": 0, "duration": 0.25, "velocity": 0.9 }
  ]
}
```

- **`instrument`**: `synth`, `amsynth`, `fmsynth`, `metalsynth`, `duosynth`, `membranesynth`, `drumsampler` or `sampler`.
- **Notes** use sharps only: write B-flat as `A#`. `velocity` is above 0 and up to 1.
- A single track's notes must fit in a 100 KB request. If a part is longer than that, split it across two tracks.

### Drums

A `drumsampler` track plays one lane per note:

| Note | Lane | Note | Lane |
|---|---|---|---|
| C2 | Kick | A2 | Low tom |
| D2 | Snare | B2 | Mid tom |
| E2 | Clap | C3 | High tom |
| F#2 | Closed hat | C#3 | Crash |
| A#2 | Open hat | D#3 | Ride |

Choose the sound for each lane with `synth.drum_kit`:

```json
"synth": { "drum_kit": { "C2": "kick-big", "E2": "clap-fat", "F#2": "hihat-analog" } }
```

<details>
<summary>Bundled drum sounds</summary>

**Kicks:** kick-808, kick-acoustic01, kick-acoustic02, kick-big, kick-classic, kick-cultivator, kick-deep, kick-dry, kick-electro01, kick-electro02, kick-floppy, kick-gritty, kick-heavy, kick-newwave, kick-oldschool, kick-plain, kick-slapback, kick-softy, kick-stomp, kick-tape, kick-thump, kick-tight, kick-tron, kick-vinyl01, kick-vinyl02, kick-zapper

**Snares:** snare-808, snare-acoustic01, snare-acoustic02, snare-analog, snare-big, snare-block, snare-brute, snare-dist01, snare-dist02, snare-dist03, snare-electro, snare-lofi01, snare-lofi02, snare-modular, snare-noise, snare-pinch, snare-punch, snare-smasher, snare-sumo, snare-tape, snare-vinyl01, snare-vinyl02

**Claps:** clap-808, clap-analog, clap-crushed, clap-fat, clap-slapper, clap-tape

**Closed hats:** hihat-808, hihat-acoustic01, hihat-acoustic02, hihat-analog, hihat-digital, hihat-dist01, hihat-dist02, hihat-electro, hihat-plain, hihat-reso, hihat-ring

**Open hats:** openhat-808, openhat-acoustic01, openhat-analog, openhat-slick, openhat-tight

**Toms:** tom-808, tom-acoustic01, tom-acoustic02, tom-analog, tom-chiptune, tom-fm, tom-lofi, tom-rototom, tom-short

**Cymbals:** crash-808, crash-acoustic, crash-noise, crash-tape, ride-acoustic01, ride-acoustic02

**Percussion:** cowbell-808, perc-808, perc-chirpy, perc-hollow, perc-laser, perc-metal, perc-nasty, perc-short, perc-tambo, perc-tribal, perc-weirdo, shaker-analog, shaker-shuffle, shaker-suckup

</details>

### Audio tracks

Audio tracks hold `clips`. Each clip names its sound in one of three ways:

- `sample_url`: a link to an audio file. It's added to your sample library if it isn't there already.
- `sample_name`: a sample already in your library.
- `sample_id`: a library id. Ids differ between accounts, so use a URL or a name in any file you share.

Clips also take `trim_start`, `trim_end`, `fade_in`, `fade_out` and `volume`.

## Effects

Up to 6 effects per track, in signal order:

```json
"effects": {
  "chain": [
    { "id": "fx-eq", "type": "eq", "params": { "lowCut": 80 } },
    { "id": "fx-verb", "type": "reverb", "params": { "decay": 3, "wet": 0.3 } }
  ]
}
```

Types: `reverb`, `delay`, `distortion`, `eq`, `compressor`, `autofilter`, `chorus`, `phaser`, `bitcrusher`, `gate`, `width`. The schema lists every parameter and its range. A parameter you leave out uses the editor's default.

Older files write effects as keys instead (`"effects": { "reverb": { "decay": 3 } }`). That's still accepted, but use `chain` in new files.

### Sidechain

Duck a track on the notes of another MIDI track, usually the kick:

```json
"effects": {
  "sidechain": { "source": "Kick", "lanes": ["C2"], "depth": 0.4, "attack": 0.003, "release": 0.25 }
}
```

`source` is the other track's `name`. `lanes` limits which of its notes trigger the duck.

### Sends and returns

Declare shared buses at the top level, then send to them by id:

```json
"returns": [{ "id": "hall", "name": "Hall", "effects": { "chain": [{ "id": "fx-hall", "type": "reverb", "params": { "decay": 6, "wet": 1 } }] } }],
"tracks": [{ "name": "Keys", "type": "midi", "effects": { "sends": { "hall": 0.3 } } }]
```

## Automation

Breakpoints per parameter. `t` is in timeline units, `v` is in the parameter's own units, and `c` is how the line moves after the point (`linear`, the default, or `hold`).

```json
"automation": {
  "volume": [{ "t": 0, "v": 0 }, { "t": 8, "v": 1 }],
  "cutoff": [{ "t": 0, "v": 400 }, { "t": 16, "v": 8000 }]
}
```

Automatable: `volume`, `pan`, `cutoff`, `resonance`, `lfoDepth`, `reverbWet`, `delayWet`.

## What's still open

This is a draft because a few things may change before it's stable:

- The full list of `synth` settings beyond the oscillator and envelopes (filter, LFO, and per-instrument options). They're accepted today and will be documented as the format settles.
- How long a stable version of the format is guaranteed to keep working.

Feedback on the format is very welcome. [Open an issue](../../../issues).
