# Meme Cam

Live webcam reactions for video calls. Meme Cam uses MediaPipe face, hand, and
upper-body landmarks to detect expressions and gestures, then places an image
or animated GIF over your face. The processed stream can be previewed in an
OpenCV window and, when `pyvirtualcam` is available, sent to a virtual camera
for use in Zoom, Google Meet, Discord, OBS, or other camera-enabled apps.

The detector is calibrated to your resting face instead of relying only on
fixed expression values. This makes reactions more personal and helps avoid
triggering them accidentally because of differences in lighting, face shape,
or camera position.

## Requirements

- Windows, macOS, or Linux
- Python 3.10 or newer
- A working webcam
- Good, reasonably even light on your face
- A terminal that can open an OpenCV preview window

The application uses:

- OpenCV for camera capture, drawing, preview, and image compositing
- MediaPipe Tasks for face, hand, and pose landmarks
- NumPy for image and landmark calculations
- Pillow for animated GIF loading
- `pyvirtualcam` for optional virtual-camera output

## Installation

Open a terminal in this folder and create a virtual environment:

```bash
python -m venv .venv
```

Activate it on Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Activate it on Windows Command Prompt:

```bat
.venv\Scripts\activate.bat
```

Install the required packages:

```bash
python -m pip install --upgrade pip
python -m pip install mediapipe opencv-python numpy Pillow pyvirtualcam
```

If you only want the local preview, `pyvirtualcam` is optional:

```bash
python -m pip install mediapipe opencv-python numpy Pillow
```

The first run checks for these files in `models/` and downloads any missing
MediaPipe models automatically:

```text
models/face_landmarker.task
models/hand_landmarker.task
models/pose_landmarker_lite.task
```

The downloads require an internet connection. After the files exist, they are
reused on later runs.

## Quick Start

1. Connect your webcam and allow your terminal or Python to use it.
2. Start the seven-second calibration:

	 ```bash
	 python main.py --calibrate
	 ```

3. During calibration, sit normally, look at the camera, keep a bored neutral
	 expression, and do not talk, smile, or raise your eyebrows. Blinking is fine.
4. After calibration completes, start Meme Cam:

	 ```bash
	 python main.py
	 ```

5. Focus the preview window to use keyboard controls.

Calibration is saved to `calibration.json`. A later run loads it automatically.
If the file is missing, invalid, or from an incompatible calibration version,
the program falls back to a generic baseline. Recalibrating is recommended for
the most reliable results.

## Controls

| Key | Action |
| --- | --- |
| `q` | Quit and close the camera |
| `d` | Toggle the on-screen debugging HUD |
| `c` | Recalibrate while the program is running |
| `1` | Temporarily show `time_out` |
| `2` | Temporarily show `heart` |
| `3` | Temporarily show `cover_nose` |
| `4` | Temporarily show `crashing_out` |
| `5` | Temporarily show `dance` |
| `6` | Temporarily show `nose_closed` |
| `7` | Temporarily show `flirty` |
| `8` | Temporarily show `hand_up` |
| `9` | Temporarily show `tongue_out` |
| `0` | Temporarily show `open_mouth` |
| `-` | Temporarily show `disgusted` |
| `=` | Temporarily show `talking_to_wall` |
| `[` | Temporarily show `suspicious` |
| `]` | Temporarily show `spin` |

The number and punctuation keys force a pose for approximately two seconds.
They are useful for checking that assets load and that the overlay is positioned
correctly.

## Command-Line Options

```text
python main.py [options]
```

| Option | Description |
| --- | --- |
| `--camera N` | Use webcam index `N`; default is `0`. Try `--camera 1` for another camera. |
| `--calibrate` | Run calibration, save `calibration.json`, and exit. |
| `--no-calibration` | Ignore `calibration.json` and use the generic baseline. |
| `--no-vcam` | Preview locally without attempting virtual-camera output. |
| `--skip-check` | Skip the MediaPipe startup preflight check. Use only when diagnosing startup issues. |
| `--size WIDTHxHEIGHT` | Request a capture size; default is `1280x720`. Lower sizes such as `640x480` can improve performance. |
| `--no-flip` | Do not mirror the camera image. By default, the preview behaves like a mirror. |

Examples:

```bash
python main.py --no-vcam
python main.py --camera 1 --size 640x480
python main.py --camera 1 --no-vcam --no-flip
python main.py --no-calibration --skip-check
```

## Automatic Reactions

Meme Cam uses several signals at the same time. Face expressions are measured
relative to your calibrated neutral face in standard deviations, with raw-value
minimums to prevent tiny noisy measurements from becoming hair triggers. A
reaction must generally remain detected for several frames before it appears,
which reduces flicker.

The built-in poses are:

| Pose | Trigger idea |
| --- | --- |
| `time_out` | One hand is held horizontally above another vertical hand. |
| `heart` | Two hands form a heart-like shape with nearby index fingers and thumbs. |
| `cover_nose` | Both palms are close to the mouth and nose area. |
| `crashing_out` | Hands are raised near the head while the mouth is widely open. |
| `dance` | Raised elbows and hands moving near the head. |
| `nose_closed` | Thumb and index finger pinch near the nose. |
| `flirty` | An index finger is held near the mouth. |
| `hand_up` | An open hand is raised beside the face. |
| `tongue_out` | The mouth is open and the inner-mouth region contains a tongue-like pink signal. |
| `open_mouth` | Calibrated jaw-opening threshold is exceeded. |
| `disgusted` | Nose sneer, brow, frown, or upper-lip signals combine strongly. |
| `talking_to_wall` | A hand is moving while no higher-priority reaction wins. |
| `suspicious` | The head turns while the eyes squint. |
| `spin` | No face, hands, or visible upper body is detected. |

Some reactions have higher priority than others. For example, a two-hand pose
is evaluated before individual-hand poses, and hand reactions are evaluated
before expression-only reactions.

## Assets

Reaction images live in `assets/`. The filename stem must exactly match a pose
name, or end with an underscore followed by that name. Both still images and
animated GIFs are supported:

```text
assets/open_mouth.png
assets/open_mouth.gif
assets/my_custom_open_mouth.jpeg
```

Supported extensions are `.gif`, `.png`, `.jpg`, and `.jpeg`. GIF frame
durations are preserved, with a minimum frame duration of 20 milliseconds.
Images are resized to follow the detected face and are alpha-composited when
they contain transparency.

If an asset is missing or cannot be read, the program creates a visible red
placeholder rather than stopping. This makes it easy to identify which pose
needs an asset. Keep replacement assets reasonably large and centered around
the intended reaction; the program scales them proportionally.

## Virtual Camera

Without extra configuration, the program always shows a local preview window.
When `pyvirtualcam` starts successfully, the terminal prints the virtual-camera
device name. Select that device as the camera in the video-call application.

The virtual camera must be supported by the operating system and available to
`pyvirtualcam`. On Windows, install and enable a compatible virtual-camera
provider if the printed error says that no backend is available. You can still
use `--no-vcam` to test detection and overlays without one.

Start the video-call application after Meme Cam has created the virtual camera,
or reopen its camera selector if the device does not immediately appear.

## Troubleshooting

### The camera cannot be opened

- Check Windows **Settings > Privacy & security > Camera** and allow access for
	your terminal or Python.
- Try another camera index:

	```bash
	python main.py --camera 1 --no-vcam
	```

- Close other applications that may already be using the webcam.
- Try a lower capture size:

	```bash
	python main.py --size 640x480 --no-vcam
	```

### Reactions trigger too easily or barely trigger

Run calibration again in the same lighting and camera position you will use:

```bash
python main.py --calibrate
```

Keep your face neutral and still during the seven-second window. If the
calibration reports that your mouth or eyebrows were raised, repeat it. Avoid
backlighting, heavy shadows, and changing distance from the camera.

### The preview is slow

Use a smaller capture size, close other camera-heavy applications, and use the
lighter pose model already included in `models/`:

```bash
python main.py --size 640x480 --no-vcam
```

### MediaPipe fails during startup

Make sure the virtual environment is active and reinstall the core packages:

```bash
python -m pip install --upgrade mediapipe opencv-python numpy Pillow
```

If the failure is a native startup or binary-compatibility error, use a clean
Python 3.11 or 3.12 virtual environment and reinstall there. The `--skip-check`
option can help confirm whether the preflight check is the failing step, but it
does not fix an incompatible MediaPipe installation.

### The virtual camera is unavailable

Run preview-only mode to confirm that detection works:

```bash
python main.py --no-vcam
```

Then install or enable a compatible virtual-camera backend and retry. A virtual
camera is optional; it is not needed to test the reactions locally.

### A pose shows a red placeholder

Check that the asset filename matches one of the exact pose names listed in the
**Automatic Reactions** section, that its extension is supported, and that it
can be opened by an image viewer. The program searches only the `assets/`
folder next to `main.py`.

## Project Layout

```text
meme-cam/
├── main.py                    # Application and detection logic
├── README.md                  # This guide
├── calibration.json           # Personal neutral-face profile
├── assets/                    # Reaction images and GIFs
└── models/                    # MediaPipe task model files
```

`calibration.json` contains measurements of your face and is specific to your
setup. It is safe to delete if you want to return to the generic baseline and
calibrate from scratch. Do not share it if you consider face-related
calibration data sensitive.

## Limitations

- Detection quality depends on lighting, camera resolution, face visibility,
	and the distance between you and the camera.
- The gesture rules are heuristic and are tuned for one clearly visible person.
- Only one face and one body are analyzed; up to two hands are used for gesture
	detection.
- The tongue detector is a color heuristic and can be affected by lighting,
	makeup, image compression, or similarly colored objects.
- The program sends BGR frames at the requested capture size and does not
	encode or record video itself.
