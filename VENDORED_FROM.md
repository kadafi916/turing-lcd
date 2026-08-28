# Provenance

`turing_lcd/` is a trimmed slice of
[mathoudebine/turing-smart-screen-python](https://github.com/mathoudebine/turing-smart-screen-python)
(`library/`, GPL-3.0-or-later) - only what `LcdCommRevA` needs (the
Rev. A / CH340-based "Turing"/UsbMonitor USB-C screen), not the full
library (no `LcdCommRevB/C/D`, no `LcdCommWeAct*`, no `LcdSimulated`,
none of the drawing helpers we don't use - line graphs, radial progress
bars, etc). Not published on PyPI and has no `setup.py`/`pyproject.toml`
- it's a standalone application repo, not a packaged library, so
vendoring the needed source files is the intended reuse path (see that
project's own comment at the top of `lcd_comm.py` acknowledging this:
"Trimmed for mister_turing_client").

**Known baseline:** first vendored into `mister_turing_client` no later
than 2026-08-24 (that repo's initial commit date). Confirmed via direct
diff to already include upstream's `9b3d1060098b` (2026-08-18,
"Retry opening the display COM port instead of exiting on first
failure") - `openSerial()`'s retry loop matches upstream word-for-word.
No exact commit SHA was pinned at the original vendoring time (this
file didn't exist yet), so that's the earliest confirmed-included
upstream commit, not a guaranteed exact match to any single SHA.

## Local deltas from vanilla upstream (not yet contributed back)

- **`write_timeout=5`** added to the `serial.Serial(...)` call in
  `openSerial()`. Upstream only sets a read `timeout`; without a write
  timeout, `serial_write()` can block indefinitely if the display stops
  draining its input buffer - confirmed in the field on 2026-08-27 (the
  whole app - polling, rendering, everything inline in one loop - froze
  for 6+ minutes at a stretch with zero exceptions anywhere, because the
  write call never returned).
- **Timeout/reconnect handling moved into `serial_write()` itself**,
  not just `WriteLine()`. Needed because some call sites (e.g.
  `SetOrientation()` in `lcd_comm_rev_a.py`, called from `Clear()`,
  which runs before the main loop even starts) write directly through
  `serial_write()` - adding `write_timeout` without this turned the
  indefinite hang into an uncaught `SerialTimeoutException` crashing the
  app on startup instead.

Full history/reasoning: see `mister_turing_client`'s commits `bd0ea13`
and `dfce5a3`.

## Consumers

- [mister_turing_client](https://gitea.ninjas.asia/kadafi/mister_turing_client)
- [turing-random-art-display](https://gitea.ninjas.asia/kadafi/turing-random-art-display)

Both currently carry their own independent copy (this repo exists so
there's one place to diff against when checking either for drift, not
because they consume it via git subtree/submodule yet - re-copying this
directory into each is still a manual step after a change here).
