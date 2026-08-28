# turing-lcd

Shared vendored copy of the USB-C "Turing"/UsbMonitor smart screen
display driver used by [mister_turing_client](https://gitea.ninjas.asia/kadafi/mister_turing_client)
and [turing-random-art-display](https://gitea.ninjas.asia/kadafi/turing-random-art-display).

Exists so there's one canonical place to check for drift - both
consumers previously carried their own independently-copied `turing_lcd/`
directory with no way to tell whether either had fallen behind the
other, or behind upstream. See `VENDORED_FROM.md` for exactly which
upstream commit this reflects and what's been changed locally.

## Using this in a project

Copy the whole `turing_lcd/` directory into the consuming project.
Needs `pyserial`, `numpy`, and `Pillow`.

```python
from turing_lcd.lcd_comm import Orientation
from turing_lcd.lcd_comm_rev_a import LcdCommRevA

comm = LcdCommRevA(com_port="AUTO", display_width=320, display_height=480)
comm.InitializeComm()
comm.SetBrightness(50)
comm.Clear()
comm.SetOrientation(Orientation.LANDSCAPE)
```

## Updating

If either consumer needs a fix to `turing_lcd/` itself (not just their
own code), make the change here first, then re-copy `turing_lcd/` into
both consumers - keeps them from drifting apart from each other, same
as this repo keeps them from silently drifting from upstream.

## License

GPL-3.0-or-later - see `LICENSE`. Vendored from
[mathoudebine/turing-smart-screen-python](https://github.com/mathoudebine/turing-smart-screen-python);
see `VENDORED_FROM.md` for full provenance and local changes.
