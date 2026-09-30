# Audio production

Read this when audio analysis, sound effects, or a mix is needed. Use supplied or appropriately licensed tracks. If music is requested without a track, select a suitable authorized source or create an arrangement suited to the brief.

Keep voice intelligible, place major actions deliberately against musical structure, and inspect clipping, alignment, and endings in the final encoded file. Choose loudness targets for the destination; the examples below use a web-video target rather than a universal delivery rule.

## Contents

- Beat analysis
- Procedural sound effects
- Mixing and muxing commands

These starter examples were retained from the original guide and have not been runtime-tested in this revision. Copy only what the project needs and verify its output. See [setup.md](setup.md) for optional analysis dependencies.

### Beat analysis

Create `scripts/beats.py`. Its output intentionally does not label every fourth detected beat as a downbeat. Verify musical meter and the first downbeat by listening or use a dedicated downbeat detector before adding bar-level cues.

```python
import json
import sys
import librosa
import numpy as np

y, sr = librosa.load(sys.argv[1], sr=None, mono=True)
tempo, beat_frames = librosa.beat.beat_track(y=y, sr=sr)
onset_frames = librosa.onset.onset_detect(y=y, sr=sr)
def times(frames):
    return librosa.frames_to_time(frames, sr=sr).round(4).tolist()
json.dump({
    "bpm_estimate": float(np.asarray(tempo).reshape(-1)[0]),
    "beats": times(beat_frames),
    "onsets": times(onset_frames),
    "downbeats": [],
    "note": "Downbeat positions require verification; detected beats may need correction."
}, sys.stdout, indent=2)
```

```bash
source .venv/bin/activate
python scripts/beats.py assets/music.wav > assets/beats.json
```

### Procedural sound effects

Create `scripts/sfx.py`. This original generator uses the installed NumPy and SoundFile packages. It validates cue names/times, supports empty cue lists, and uses a fixed noise seed. Effects support the music; they are not a full score.

```python
import json
import sys
import numpy as np
import soundfile as sf

cues_path, destination, duration_text = sys.argv[1:4]
sr, duration = 48000, float(duration_text)
if not np.isfinite(duration) or duration <= 0:
    raise ValueError("Duration must be positive")
audio = np.zeros(round(duration * sr), dtype=np.float64)
rng = np.random.default_rng(420)
lengths = {"click": .06, "pop": .16, "thump": .45, "whoosh": .3}
with open(cues_path) as handle:
    cues = json.load(handle)
for cue in cues:
    kind, start_seconds = cue["type"], float(cue["t"])
    if kind not in lengths or not 0 <= start_seconds < duration:
        raise ValueError(f"Invalid cue: {cue}")
    t = np.arange(round(lengths[kind] * sr)) / sr
    if kind == "whoosh":
        wave = rng.uniform(-1, 1, len(t)) * np.sin(np.pi * t / lengths[kind]) * .18
    else:
        frequency, decay = {"click": (1600, 85), "pop": (750, 30), "thump": (85, 12)}[kind]
        wave = .35 * np.sin(2 * np.pi * frequency * t) * np.exp(-decay * t)
    start = round(start_seconds * sr)
    count = min(len(wave), len(audio) - start)
    audio[start:start + count] += wave[:count]
peak = np.max(np.abs(audio))
if peak > .95:
    audio *= .95 / peak
sf.write(destination, audio, sr, subtype="PCM_16")
```

Example `assets/cues.json`:

```json
[{"t":0.4,"type":"click"},{"t":1.2,"type":"whoosh"},{"t":2.5,"type":"thump"}]
```

```bash
python scripts/sfx.py assets/cues.json out/sfx.wav 4
ffmpeg -i assets/music.wav -i out/sfx.wav -filter_complex "[0:a]volume=0.65[m];[m][1:a]amix=inputs=2:duration=longest,loudnorm=I=-14:TP=-1:LRA=11[a]" -map "[a]" -t 4 out/mix.wav
ffmpeg -i out/silent.mp4 -i out/mix.wav -map 0:v:0 -map 1:a:0 -c:v copy -c:a aac -b:a 192k -shortest -movflags +faststart out/final.mp4
```

Replace the example four-second duration with the production duration. Ensure music and effects cover the entire video before using `-shortest`. For measured delivery requirements, use FFmpeg's two-pass loudness normalization and verify the encoded result. If music is to be synthesized, create an arrangement with harmonic, rhythmic, and dynamic development on the same timing grid; the effects script does not satisfy that requirement.

