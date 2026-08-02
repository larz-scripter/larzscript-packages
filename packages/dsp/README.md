# lz-dsp

Real, from-scratch audio signal processing: a biquad peaking/shelving EQ
(the real RBJ Audio EQ Cookbook formulas), a look-up-table compressor (the
transfer curve is precomputed once, so the per-sample hot loop is index +
multiply, not a `log()` call per sample), a brickwall limiter, and
peak/RMS/normalize helpers - plus the `sin`/`cos`/`exp`/`ln` none of that
can lean on a builtin for (`pow()` only takes whole-number exponents).
Install: `larzscript pkg install dsp`.

```
import "dsp" as dsp

let sr = 22050
let eq = dsp.biquad_peaking_new(1000, 6.0, 1.0, sr)   # +6dB @ 1kHz
let buf = []
let i = 0
while i < 200 {
  buf.push(dsp.sin_(2 * 3.141592653589793 * 1000 * i / float(sr)) * 0.5)
  i = i + 1
}
let boosted = dsp.biquad_process_buffer(eq, buf)
print(round(dsp.peak_dbfs(buf), 2), "dBFS ->", round(dsp.peak_dbfs(boosted), 2), "dBFS")

let mastered = dsp.normalize_to(dsp.limit(boosted, -1.0), -3.0)
print("final peak:", round(dsp.peak_dbfs(mastered), 2), "dBFS")
```

Every filter's state is a plain dict the caller owns and threads through
calls (same convention as [`ratelimit`](../ratelimit)) - no hidden global,
so multiple independent filters/tracks never interfere.

**A real Larzscript gotcha this package works around:** `%` truncates both
operands to integers first (`1.5708 % 6.283` is `1 % 6`, not a float
remainder) - there's no float `fmod`, so `sin_()`'s range reduction uses its
own floor-division-based `_fmod()` instead.

See [`wav`](../wav) for reading/writing the actual audio files this
processes, and
[`larzscript-beatstudio`](https://github.com/larz-scripter/larzscript-beatstudio)
for the full beat-making/mixing/mastering app built on both.
