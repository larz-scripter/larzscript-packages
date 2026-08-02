# lz-wav

Read and write real WAV (RIFF/PCM) files, pure Larzscript. Audio content is
a list of floats in [-1, 1] (mono: one value per sample; stereo:
interleaved L, R, L, R, ...) - the same convention [`dsp`](../dsp) buffers
use, so the two packages compose directly. 16-bit PCM only. Install:
`larzscript pkg install wav`.

```
import "wav" as wav

wav.write("tone.wav", samples, 22050)          # mono, 16-bit
let w = wav.read("tone.wav")
print(w["sample_rate"], w["channels"], len(w["samples"]))
```

`read()` scans real RIFF chunks rather than assuming a fixed header layout
- verified against files this package wrote itself *and* real files from
`ffmpeg` (which add extra metadata chunks before `data`), not just its own
output. Files are built/parsed as byte lists (ints 0-255) via
`write_file()`/`read_file_bytes()`'s binary-safe contract - see
[`archive`](../archive)'s docstring for why a WAV header can't be built
through string concatenation (it's full of `0x00` bytes, which silently
truncate a Larzscript string).

See [`larzscript-beatstudio`](https://github.com/larz-scripter/larzscript-beatstudio)
for the full beat-making/mixing/mastering app built on this + `dsp`.
