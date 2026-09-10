# Otonom İHA — ArUco Alignment and MAVLink Helpers

Bench code from the TEKNOFEST autonomous UAV competition for team GRAVITEAM:
ArUco marker detection driving a vehicle-alignment command, plus small pymavlink
helpers for connecting to and debugging a flight controller.

> **Status: bench and diagnostic scripts.** This is the scratch work from a
> competition build, not a finished system. It is kept because the ArUco
> alignment approach and the connection helper were reused later.

![demo](docs/demo.gif)

## Tech stack

![Python](https://img.shields.io/badge/Python-3-3776AB?logo=python&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-ArUco-5C3EE8?logo=opencv&logoColor=white)
![pymavlink](https://img.shields.io/badge/pymavlink-MAVLink-2C5F2D)
![ArduPilot](https://img.shields.io/badge/ArduPilot-Flight_stack-6E4C9E)
![License](https://img.shields.io/badge/License-MIT-blue)

## Quick start

```bash
git clone https://github.com/yunusemretom/Otonom_IHA.git
cd Otonom_IHA

python3 -m venv .venv
source .venv/bin/activate
pip install opencv-contrib-python numpy pymavlink

python auroco.py            # ArUco detection and alignment output
python kamera_test.py       # find which index the camera is on
python baglanti_denem.py    # find which port the vehicle is on
```

`opencv-contrib-python` is required rather than `opencv-python`, because the
ArUco module lives in contrib.

## How it works

| File | Purpose |
|---|---|
| `auroco.py` | ArUco detection, converts marker offset to an alignment command |
| `connection_lib.py` | pymavlink wrapper for sending commands more reliably |
| `baglanti_denem.py` | port discovery for the flight controller |
| `kamera_test.py` | camera index discovery |
| `main.py` | the combined on-vehicle script |
| `deneme.py`, `deneme2.py` | face detection experiments, unrelated to the mission |

### ArUco instead of QR

The mission needed the aircraft to align itself over a ground marker. QR code
libraries were tried first and were too slow and too fragile from altitude: a
code seen small and at an angle either decodes or returns nothing, with no
useful middle ground.

ArUco markers are designed for exactly this. They are detected geometrically
rather than decoded, so a partially degraded marker still yields a position and
an orientation. Detection is also fast enough to run in a control loop, which a
QR decode attempt per frame was not.

The alignment maps pixel offset from the frame center onto an RC value around
neutral:

```python
value = 1500 - ((center - measured) // 5)
```

The divisor is a hand-tuned proportional gain. It is deliberately small, because
the failure mode when it was larger was the aircraft overshooting the marker and
oscillating across it. This is a proportional controller and nothing more, which
was adequate for alignment over a stationary marker and would not be for
tracking a moving one.

The dictionary is `DICT_6X6_1000`, with the other options left commented in the
source so a different marker set can be swapped in without looking up names.

## Known limitations

- Scripts, not a system. No configuration, no error handling, no packaging.
- Proportional control only. No integral or derivative term, so a steady wind
  leaves a standing offset.
- The gain and the marker dictionary are hardcoded for one camera and altitude.
- `deneme.py` and `deneme2.py` are unrelated face-detection experiments left in
  place.
- Camera and port discovery exist because the hardware kept moving between
  indices, which is a workaround rather than a fix.
- No tests, and no logs from the competition runs.

## Roadmap

Superseded by the integrated systems in
[AybuHavk](https://github.com/yunusemretom/AybuHavk) and
[DogFight](https://github.com/yunusemretom/DogFight).

## License

MIT. See [LICENSE](LICENSE).

Turkish per-file descriptions are preserved in [README.tr.md](README.tr.md).
