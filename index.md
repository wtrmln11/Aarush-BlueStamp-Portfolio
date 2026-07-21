# Facial Recognition System

This project uses a Raspberry Pi to build a real-time facial recognition system capable of identifying people at distance using a 64MP camera, digital zoom, and machine learning. The system is trained on a custom image dataset to recognize specific individuals through a connected camera with live confidence scoring.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Aarush H | Evergreen Valley High School | Computer Engineering | Incoming Senior |

<img src="Aarush%20H.jpeg" alt="Headstone Image" width="300" height="400">

---

## Final Milestone

<!--
<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

- Achieved reliable facial recognition at up to 8 meters using the 64MP Arducam Hawkeye camera with digital zoom
- Built a dynamic zoom system (= / - keys) that automatically adjusts processing detail and frame skip rate based on zoom level — more zoom means the algorithm keeps more pixel detail per face
- Implemented confidence percentage scoring on every recognized face, color coded green (high), orange (low), red (unknown)
- Solved the core range problem by exploiting the 64MP sensor: zooming in crops the sensor rather than stretching pixels, so faces at distance still have hundreds of pixels of detail for the algorithm to work with
- Overhauled model training with image augmentation (flip, brightness, rotation) that multiplies each training photo 6x without taking more pictures, and a CNN fallback for hard-to-detect faces
-->

---

## Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/xFbNuY9iE_g?si=Cp3FTndi-fisdUzk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

-Trained the model on images in the dataset using Python, allowing the live feed to show people's names

-Surprised by how much data the model needs, only 200 pictures of 4 people in the dataset wasn't enough to allow the model to recognize people past 3 feet

-Initially the live feed was extremely zoomed in, so I adjusted the resolution to fit the monitor and then added more people to the dataset to improve the accuracy of the model

-The next goal is to detect a person from across the classroom, around 10 meters

---

## First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/wKA9XhxVHsE?si=93JbfTmWjfTUcsn5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

-Set up OpenCV and Python scripts that capture photos on spacebar press and organize them into named folders so the model can associate names with faces

-The quality of the camera was worse than expected only ever sharp when the subject was directly in front of it

-Some of the captured images became corrupted silently causing me to have to mass delete entire batches of images as there was no way of knowing which were corrupted

-Next goal is to detect myself and at least one other person

---

## Starter Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/wKA9XhxVHsE?si=93JbfTmWjfTUcsn5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

-Built retro arcade device by connecting a battery to the metal contacts on the back of the device, soldering the side charging port and buttons (movement and reset), then finally putting it all together in the acrylic case with the screws.

---
# Challenges and Struggles

This project changed a lot from the first version to the last. Below are the real problems I ran into and how I solved each one.

## Getting faces at a distance

The problem: The camera catches few pixels of a face when it is at a distance. The model needs to see enough of a person to recognize who they are. The face at a distance of 3 feet was too small for the model to recognize.

The fix: The camera has a 64 megapixel sensor. There's more detail in the sensor than the screen. Cutting into the sensor captures a face and its hundreds of pixels, instead of stretching a small image and losing detail. This digital zoom works with the = and - keys on the keyboard. I made it automatic too, and later added manual pan (T/F/G/B) so I can look around the frame without moving the camera.

## Blurry and corrupted training photos

The problem: When the camera was set up, only when someone was in front of the camera did it capture sharp photos. Some of my saved photos became corrupted without any warning. This meant deleting many of my photos.

The fix: Adding autofocus to the camera means that the camera will focus when every photo is taken. A cooldown ensures the autofocus is not triggered twice in a row. Saving the photos at high resolution with a quality set at the camera ensured that no more photos would be corrupted.

## Not enough training data

The problem: My data set had only 200 photos of 4 people. The model worked well for people at close range, but failed at 3 feet away.

The fix: I take more photos of myself from many angles and distances. I used image augmentation to take each of my photos and make a set of 6 photos of myself from flipped, brighter, darker, and rotated images. This teaches my model to see more lighting angles, without taking more of my photos.

## Switching the recognition engine

The first recognition engine used was the face_recognition library together with MediaPipe. It worked well for people up-close, but would fail at a distance or high accuracy.

I changed to InsightFace, which is a stronger and more modern recognition engine together with the buffalo_l model. This model converts a face to a list of 512 numbers called an embedding. These numbers act like a fingerprint. By comparing the embedding to their saved embeddings, my program can recognize which face is which.

## The speed problem

The problem: The new InsightFace model is slow on the Raspberry Pi. It takes close to a second to recognize a person. The box freezes while the slow model runs.

The fix: I split the work across three threads that run at the same time. A detection thread finds faces fast and publishes boxes. A separate embedding thread runs the slow recognition and fills in names a moment later. The main loop follows each face every frame with a fast template tracker. Splitting detection from embedding made the box refresh about three times faster, because the detector no longer waits on the slow recognition step.

## Boxes stuck on the wall

The problem: The box with my name was stuck on the wall behind me. The tracker had drifted off my face onto the wall.

The fix: Three guards. The tracker checks that a patch has real texture, and a wall has none. Any box the detector stops confirming is removed on a timer, so a drifted box cannot live forever. The detector also raises its confidence bar so shadows on the wall are not treated as faces.

## The box showing where I used to be

The problem: The detector is about a second slow, so its box shows where I was a second ago. When I move, the box lags behind.

The fix: The fast tracker owns the box position in real time. The slow detector only supplies the name and corrects the box size. On top of that, the detection is projected forward by my measured speed to where I am now, so the box snaps onto my current position instead of my old one.

## Losing the box while moving, and duplicate boxes

The problem: Moving quickly lost the box, and sometimes one person got a second box copied elsewhere on the screen.

The fix: The tracker coasts through brief tracking failures instead of freezing, gliding at my last known speed for a short distance so a fast move or a quick head turn does not drop the box. A hard distance cap stops a lost box from flying across the screen, and duplicate suppression removes any second box that carries the same name once it goes stale. Two real people are always kept separate.

## Distance, height, speed, and age

Once tracking was solid, I added measurements from the face box. Distance comes from apparent face size, calibrated by standing at one meter and pressing D. Height comes from where the head sits in the frame relative to the camera's optical axis, calibrated with K. Speed uses the face as an on-screen ruler. Age and gender come from the model's built-in estimator. Getting distance and height accurate at zoom was the hardest part, because zooming and panning move the optical axis off the center of the frame, which had to be accounted for.

## Running out of memory

The problem: When running at full resolution, the Raspberry Pi runs out of memory.

The fix: Cap the full-mode resolution request, hold fewer frame buffers, and allocate no raw buffers. If the camera fails to start in high resolution mode, the program falls back to a lower mode instead of crashing.

## Small bugs along the way

The video feed showed everyone with blue faces, a quirk of the color conversion, fixed by swapping blue and red. Age readings were wildly off until I fed that model the color order it actually expects, separate from the recognition path.

## Portable and headless

I made the system run with no monitor, powered by a power bank, and streamed the feed to a phone or laptop browser over WiFi. The Pi can join a phone hotspot so it works anywhere, and it auto-detects when there is no display and streams only.

## Quality of life features

Scan zones to focus recognition on part of the screen. Auto zoom to follow the nearest face. A reload key to clear the screen. A screenshot key. An offline voice module that announces a recognized name above 60 percent confidence.

## What I learned

Most of my time was spent closing the gap between a model that works in theory and one that works in real time on cheap hardware. The hardest part was making the box smooth, accurate, and centered on my face at the same time as the slower recognition model ran behind it.

---

## Schematics

<div style="display:flex; flex-wrap:wrap; gap:10px;">
  <img src="Case1.png" alt="Schematic 1" width="300" height="400">
  <img src="case2.png" alt="Schematic 2" width="300" height="400">
  <img src="case3.png" alt="Schematic 3" width="300" height="400">
  <img src="case4.png" alt="Schematic 4" width="300" height="400">
  <img src="case5.png" alt="Schematic 5 (placeholder)" width="300" height="400">
  <img src="case6.png" alt="Schematic 6 (placeholder)" width="300" height="400">
  <img src="case7.png" alt="Schematic 7 (placeholder)" width="300" height="400">
  <img src="case8.png" alt="Schematic 8 (placeholder)" width="300" height="400">
</div>

---

## Code

The full source for each script is collapsed to keep this page short. Click any bar below to expand the code.
### Face Recognition (face_rec-picam.py)

The main script. Runs continuous face recognition on the Pi Camera feed with InsightFace, a three-thread pipeline (detect / embed / track) for smooth boxes, digital zoom with manual pan, scan zones, live confidence scoring, distance / height / speed / age readouts, offline voice announcements, and browser streaming for headless use.

Controls: `=` zoom in, `-` zoom out, `A` auto zoom, `T/F/G/B` pan, `V` re-center, `H` sensor mode, `E` encoding set, `D` calibrate distance, `K` calibrate camera height, `I` info overlay, `R` reload, `L` screenshot, `S` zones, `C` clear zones, `Q` quit.

<details>
<summary><b>Click to view face_rec-picam.py</b></summary>

```python
# ---------------------------------------------------------------------
#  THREAD BUDGET  (must run BEFORE onnxruntime / OpenCV are imported)
#
#  The Pi 4 has 4 cores. Left alone, ONNX gives the detector 4 threads,
#  the embedder another 4, and OpenCV takes 4 more in the display loop.
#  That is 12 threads fighting over 4 cores, so the display loop gets
#  starved. The tracker runs in that loop, which is why the box stutters
#  and drops lock exactly when recognition is busy.
#
#  Capping each pool leaves a core free for the display loop. Detection
#  barely slows, because it was never really getting 4 cores anyway.
#  Raise these only if you move to a Pi 5.
# ---------------------------------------------------------------------
import os
ONNX_THREADS = 2
os.environ.setdefault("OMP_NUM_THREADS", str(ONNX_THREADS))
os.environ.setdefault("OPENBLAS_NUM_THREADS", str(ONNX_THREADS))
os.environ.setdefault("MKL_NUM_THREADS", str(ONNX_THREADS))

import cv2
cv2.setNumThreads(2)            # leave headroom for the ONNX workers

import numpy as np
from picamera2 import Picamera2
from libcamera import controls, Transform
import time
import pickle
import subprocess
import json
from queue import Queue, Full, Empty
import socket
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer
from urllib.parse import urlparse, parse_qs
from collections import deque, Counter
from insightface.app import FaceAnalysis
from insightface.app.common import Face
from threading import Thread, Lock

# =====================================================================
#  FACE RECOGNITION v2
#
#  What changed vs v1:
#  1. FEED QUALITY AT ZOOM
#     - Sensor mode auto-switches to FULL resolution at high zoom, so
#       ScalerCrop pulls detail from the full sensor readout instead
#       of a binned mode. Big sharpness gain past ~3x.
#     - RAM friendly: the full-mode request is capped at 3840x2160,
#       no raw buffers are allocated (sensor hint instead of a raw
#       stream), and FULL mode runs with 2 buffers. If allocation
#       still fails, the script drops back to BINNED on its own.
#     - Main (recognition) stream raised to 1920x1080.
#     - Display lores stream raised to 1024x576 (was 640x480 upscaled).
#  2. SPEED
#     - Detection decoupled from embedding. Every pass runs the fast
#       SCRFD detector; the heavy ArcFace embedding only runs when a
#       face is new or its identity is stale. Boxes refresh several
#       times per second instead of once per full pass.
#     - Optional: set MODEL_PACK = "buffalo_s" for a 3-5x speedup
#       (requires re-running enrollment, embeddings differ per pack).
#  3. ACCURACY
#     - No more cv_scaler downscale before detection (v1 shrank the
#       frame 3-4x, which destroyed small/far faces).
#     - Margin test: best match must beat the best OTHER person by
#       MATCH_MARGIN, cuts sibling/family mix-ups.
#     - Per-track name voting over the last 5 identifications, stops
#       label flicker.
#     - Small zone crops get a 2x cubic upscale before detection.
#     - Fixed a vertical scale bug in box placement (v1 used the X
#       scale factor for Y).
#     - Stored encodings are L2-normalized at load, so cosine scores
#       are always on a proper 0..1 scale.
#  4. SELECTION TOOL
#     - Freeze frame now comes from the full-res MAIN stream (sharp),
#       not the upscaled lores stream (blurry).
#     - Crosshair, live pixel dimensions while dragging, a magnifier
#       loupe at native resolution, and a high-res preview panel of
#       the last zone drawn.
#     - Right-click deletes the zone under the cursor. ESC or S
#       confirms and exits.
#
#  GHOST BOX PATCH
#     - Tracks now expire after TRACK_MAX_MISSES detector passes
#       without a confirming detection. Before, a drifted template
#       could pin a named box to a wall forever.
#     - Template matching threshold raised to 0.55 and low-texture
#       patches are rejected, so templates stop sticking to flat
#       walls.
#     - Detector confidence floor raised to 0.60.
#     - Cached identities only transfer to detections of similar
#       size, and camera moves or zone edits flush stale tracks and
#       cached identities.
#
#  RESPONSIVENESS PATCH
#     - Main stream back to 720p so the loop runs fast and the box
#       keeps up with you. Zoom detail still comes from the full-res
#       sensor crop, not this stream.
#     - Tracker restored to early-version feel: low match threshold
#       and a template that refreshes every frame, so the box follows
#       through motion blur instead of losing lock.
#     - Motion predictor shifts the search window ahead of a moving
#       face, so quick head movement no longer drops the box.
#     - Drift correction: a detection nudges the box only when it is
#       close (you are still); when far it is treated as stale and
#       never yanks the box. Keeps the box centered without lag.
#     - Roughly symmetric head expansion, so your face sits centered
#       in the drawn box.
#     - R reloads: clears tracks and identities and re-detects, for
#       when a box is stuck or a face is missed.
#
#  VOICE ANNOUNCEMENTS
#     - Offline Piper TTS speaks "<Name> detected." over the speaker
#       when a known face appears above 60% confidence. New arrivals
#       only; a name re-announces after being gone 5s. Rendering is
#       cached per name, playback runs on its own thread so the video
#       never stalls. See the setup notes above ANNOUNCE_ENABLED.
#
#  Intended for recognizing enrolled household members on your own
#  hardware.
# =====================================================================

# =========================== TUNABLES ================================
MAIN_W, MAIN_H = 1280, 720      # recognition stream. 720p keeps the
                                # loop fast so the tracker follows you
                                # smoothly. Quality at zoom still comes
                                # from the full-res SENSOR crop, not
                                # this stream. Raise to 1600x900 only
                                # if you need more far-face detection
                                # and can spare the frame rate.
LORES_W, LORES_H = 1024, 576    # display source stream. MUST keep the
                                # same aspect ratio as MAIN (16:9), or
                                # the ISP may letterbox/crop instead of
                                # scaling, and every display->main
                                # conversion silently gains an unknown
                                # factor. The tiny resize to DISPLAY_H is
                                # worth the certainty.
DISPLAY_W, DISPLAY_H = 1024, 600

MODEL_PACK = "buffalo_l"        # "buffalo_s" is 3-5x faster on CPU but
                                # you must re-run enrollment with the
                                # same pack (embeddings not compatible)
DET_SIZE = (640, 640)           # 512x512 or 480x480 trades small-face
                                # detection for speed

SIMILARITY_THRESHOLD = 0.4      # cosine similarity, higher = stricter
ZONE_THRESHOLD = 0.35
MATCH_MARGIN = 0.03             # best match must beat best other
                                # person by this much

REVERIFY_SEC = 2.5              # re-run embedding on a face this often
MAX_EMBEDS_PER_PASS = 2         # cap heavy ArcFace calls per pass
MIN_FACE_EMBED_PX = 36          # skip embedding on faces smaller than
                                # this (main-frame pixels)
CACHE_SEEN_TTL = 3.0            # drop identity cache entries unseen
                                # this long
CACHE_MATCH_MIN_PX = 320        # how far a face may move between
                                # detection passes and still be
                                # recognised as the same person. Covers a
                                # brisk walk at typical detection rates.
CACHE_MATCH_FACE_FRAC = 3.0     # or this many face-heights, whichever is
                                # larger (close-up faces move further in
                                # pixels for the same real speed)

ZONE_UPSCALE_MIN = 380          # zone crops with min side below this
                                # get a 2x upscale before detection

# Sensor mode switching (feed quality at zoom)
ZOOM_FULLRES_ON = 3.0           # go to full-res sensor above this zoom
ZOOM_FULLRES_OFF = 2.2          # back to binned below this (hysteresis)
SENSOR_SWITCH_COOLDOWN = 4.0    # min seconds between mode switches
SENSOR_MAX_REQ = (3840, 2160)   # 4K cap on the full-mode request.
                                # The driver maps this onto the nearest
                                # real sensor readout.
BUFFERS_BINNED = 3              # camera buffers per stream
BUFFERS_FULL = 2                # keep FULL mode lean on RAM

# Display color path. This matches v1, which rendered skin tones
# correctly on the Pi ISP. If faces ever look blue or orange, swap
# RGB <-> BGR here (one line, nothing else changes).
LORES_COLOR = cv2.COLOR_YUV420p2RGB

FAR_ZOOM_THRESHOLD = 2.5
MAX_ZONES = 3
ZONE_COLORS = [(0, 220, 255), (0, 160, 255), (180, 0, 255)]
ZONE_NAMES = ["ZONE A", "ZONE B", "ZONE C"]

# Template tracker. The tracker runs every DISPLAY frame on the live
# image, so it owns box POSITION in real time. The slow detector only
# supplies identity, keeps tracks alive, and corrects box SIZE. This
# split is what stops the box snapping back to where you were a second
# ago when a slow detection finally returns.
TRACK_SEARCH_MARGIN = 120       # px search radius per frame, plus the
                                # motion predictor below, so fast head
                                # movement stays in range
MATCH_THRESHOLD = 0.30          # min correlation to MOVE the box. Kept
                                # low so the box follows through motion
                                # blur. Safe to be this loose because
                                # duplicate suppression and lag-corrected
                                # snapping now clean up anything wrong
MATCH_THRESHOLD_COAST = 0.20    # easier bar to REGAIN lock while
                                # coasting. Already committed to that
                                # face, so accept a weaker match rather
                                # than stall
PREDICT_GAIN = 0.9              # how much measured velocity shifts the
                                # search window ahead of the face
VEL_SMOOTH = 0.45               # EMA on velocity (px/sec)
COAST_MAX_FRAMES = 22           # frames to dead-reckon through a failed
                                # match before giving up. At ~15 fps this
                                # is ~1.5 s of gliding, which carries the
                                # box through blur or a brief head turn.
COAST_MAX_PX = 160              # floor for how far a box may coast in
                                # total. The real cap scales with face
                                # size (COAST_MAX_FACE_FRAC), because a
                                # close-up face legitimately crosses more
                                # pixels for the same real movement.
                                # Some cap is essential: without one, a
                                # box that lost lock at speed sails clear
                                # across the screen still wearing your
                                # name, which reads as a duplicate.
COAST_MAX_FACE_FRAC = 2.5       # or this many face-widths, whichever is
                                # larger
COAST_VEL_DECAY = 0.80          # bleed velocity off per coasted frame.
                                # A coasting box has no evidence you are
                                # still moving, so it should ease to a
                                # stop rather than cruise on forever.
# Template refresh adapts to motion. Standing still, refresh slowly so
# the template cannot walk off your face (anti-drift). Moving, refresh
# fast so the template stays blurred like your face is (anti-stutter).
# One blend rate cannot do both, which is what broke tracking before.
TEMPLATE_BLEND_STILL = 0.15
TEMPLATE_BLEND_MOVING = 0.55
BLEND_SPEED_FULL = 350.0        # px/sec at which the moving blend is
                                # fully applied
MIN_TEMPLATE_STD = 12.0         # tracked patch must have texture, so
                                # templates never latch onto flat walls
RESEED_SEARCH_MARGIN = 300      # a LAGGED detection still associates
                                # with your moved box instead of
                                # spawning a phantom track behind you
SPAWN_MIN_DIST = 50             # floor for the spawn guard, in px. The
                                # real guard scales with face size (see
                                # spawn_guard_px), because a fixed guard
                                # blocks two real people standing close
                                # together at distance from each getting
                                # their own box
SPAWN_FACE_FRAC = 0.9           # spawn guard = this * detected face height
# Duplicate suppression. Two boxes on one person happen when a template
# drifts onto a shoulder or hair and holds while the detector spawns a
# fresh track on the real face. This is NMS for tracks: overlapping or
# co-located tracks collapse to the one the detector confirmed most
# recently.
TRACK_MERGE_IOU = 0.35          # boxes overlapping more than this merge
TRACK_MERGE_CENTER_FRAC = 0.55  # or centers closer than this * face size
# Same-name duplicates are only removed when the copy is STALE, meaning
# the detector is not currently confirming it. Two live people who get
# briefly misidentified as each other must both keep their boxes.
SAME_NAME_STALE_SEC = 1.2
SAME_NAME_STALE_MISSES = 2
TRACK_UNCONFIRMED_SEC = 6.0     # hard time limit since the detector last
                                # confirmed a track. This is the real
                                # backstop against ghosts. Raised because
                                # a box vanishing for a few seconds and
                                # coming back is worse than a box that
                                # lingers slightly too long, and the
                                # duplicate suppressor removes wrong ones
                                # within a frame anyway.
# Lag compensation. The detector reports where a face WAS at capture
# time, typically ~1s ago on a Pi. Rather than blend partway toward that
# stale position (which leaves a permanent offset, so the box settles
# onto a neck or shoulder), project the detection forward by the
# tracker's measured velocity and snap the box onto it. Correct position
# AND no yank, instead of a compromise between the two.
LAG_COMPENSATE = True
LAG_MAX_SEC = 1.5               # ignore absurd lags (worker hiccup)
LAG_MAX_SHIFT_PX = 400          # cap the projection, so a wild velocity
                                # estimate cannot fling the box
SNAP_BLEND = 0.9                # how hard to snap onto the (projected)
                                # detection. High on purpose: detection
                                # is ground truth for position once the
                                # lag is removed
SIZE_BLEND = 0.35               # how fast the box grows/shrinks toward
                                # the detected face size (0..1 per pass)
TRACK_MAX_MISSES = 12           # detector passes with no matching
                                # detection before a track is removed.
                                # TRACK_UNCONFIRMED_SEC is the real
                                # backstop, so this can be generous
IDENTITY_STALE_SEC = 5.0
NAME_VOTE_LEN = 5
DET_CONFIDENCE = 0.60           # SCRFD score floor. Default 0.5 lets
                                # wall textures and shadows through
PENDING = ""                    # placeholder name before the first ID

# Box framing. SCRFD boxes hug eyebrows to chin. Expansion is close to
# symmetric so your face sits CENTERED in the drawn box, with just a
# little extra on top for hair.
BOX_EXPAND_X = 0.18             # total width growth
BOX_EXPAND_TOP = 0.24           # upward growth (forehead, hair)
BOX_EXPAND_BOTTOM = 0.20        # downward growth (chin, jaw)

# Auto framing. Faces smaller than this (main-frame px) do not steer
# the auto zoom or pan, so a one-pass false detection at the frame
# edge no longer yanks the camera.
# =====================================================================
#  MANUAL PAN  ("look around" without moving the camera)
#
#  The sensor is far larger than the view, so at any zoom above 1x there
#  is real image sitting outside the frame. These controls slide the
#  crop around inside it, which looks exactly like panning a camera on a
#  tripod, except nothing physically moves.
#
#    T F G B           pan up / left / right / down   (letter keys, so
#                      no numpad or working arrows needed)
#    V                 re-centre the view
#    Arrow keys        same, if your OpenCV build passes them through
#    8 4 6 2 / 5       same again, numpad-style
#
#  T F G B sits as a diamond on the keyboard:  F and G are side by side
#  for left/right, T is above them, B below.
#
#  Panning switches auto-zoom OFF, otherwise auto-pan fights you for the
#  wheel. Press A to hand control back.
#
#  NOTE: at 1.0x zoom the crop already covers the whole sensor, so there
#  is nowhere to pan to. Zoom in first, then look around.
#
#  If a direction feels backwards on your camera, flip the matching
#  invert below. The camera Transform (HFLIP/VFLIP) can reverse these
#  depending on how the module is mounted.
# =====================================================================
PAN_STEP = 0.12                 # fraction of the current VIEW to move
                                # per keypress. Independent of zoom, so
                                # a press always shifts the picture by
                                # the same amount on screen.
PAN_INVERT_X = False            # set True if left/right feel swapped
PAN_INVERT_Y = False            # set True if up/down feel swapped

# Arrow key codes differ by OpenCV backend, so accept the known sets.
# Deliberately NOT including the legacy 81-84 codes: those collide with
# uppercase Q, R, S and T, which are already bound.
PAN_KEYS_UP = (65362, 16777235, 2490368, 63232)
PAN_KEYS_DOWN = (65364, 16777237, 2621440, 63233)
PAN_KEYS_LEFT = (65361, 16777234, 2424832, 63234)
PAN_KEYS_RIGHT = (65363, 16777236, 2555904, 63235)

AUTO_MIN_FACE_PX = 30

# =====================================================================
#  PORTABLE / BROWSER STREAM  (full quality, no throttling for now)
#
#  Runs the Pi with no monitor, powered by a battery, viewed from a
#  laptop or phone browser. Standard library only, so nothing to pip
#  install in the field.
#
#  Testing full quality first on purpose. If the feed lags or the Pi
#  throttles, drop STREAM_MAX_FPS and STREAM_QUALITY, or gate the frame
#  rate on whether a face is present. Not doing any of that yet.
#
#  ---- ONE-TIME: join your phone hotspot ----
#    Phone hotspot ON, then on the Pi (while on home WiFi or a monitor):
#      sudo nmcli device wifi connect "HOTSPOT_NAME" password "PASSWORD"
#    It reconnects automatically after that. Home WiFi still works too;
#    the Pi joins whichever is in range.
#
#  ---- USING IT ----
#    1. Hotspot ON, power the Pi, wait ~30 s.
#    2. Find the Pi's IP: phone hotspot client list, or `hostname -I`.
#    3. Browse to  http://<pi-ip>:8000  from any device on the network.
#    4. Run under tmux so it survives an SSH disconnect:
#         tmux
#         python3 face_recognition_v2.py
#       Detach with Ctrl-B then D. Reattach with `tmux attach`.
#
#  Headless is auto-detected: no DISPLAY means no monitor, so the
#  OpenCV window is skipped. Force it with HEADLESS below.
#
#  ---- POWER (Pi 4) ----
#  Needs 5V/3A. A weaker bank browns out under load and reboots at
#  random, which looks like a crash. Check:  vcgencmd get_throttled
#  (0x0 is healthy). Fit a heatsink; a throttled Pi tanks FPS and
#  brings back the tracking problems.
# =====================================================================
STREAM_ENABLED = True
STREAM_PORT = 8000
STREAM_QUALITY = 90             # JPEG quality 1-100. High on purpose for
                                # this first test. Lower it if the feed
                                # or recognition struggle.
STREAM_MAX_FPS = 30             # no real throttle for now; the display
                                # loop rarely exceeds this on a Pi 4
HEADLESS = None                 # None = auto-detect from DISPLAY.
                                # True forces no window, False forces one


# =====================================================================
#  DISTANCE ESTIMATION
#
#  A face that covers more pixels is closer. With the pinhole model,
#  distance = DIST_C / apparent_size, where apparent_size is the face
#  box size normalized for the digital zoom. DIST_C bundles the focal
#  length and real face size into one number.
#
#  Two people are rarely the same face size, so the accurate path is a
#  one-time calibration: stand at DIST_CALIB_CM from the camera with
#  your face the largest in frame and press D. That reading is saved to
#  DIST_CALIB_FILE and reused on every future run. Until you calibrate,
#  a rough default is used and the readout is marked with a ~.
#
#  Caveats: tilting your head up or down changes the box size a little,
#  so expect a few percent wobble. Calibrate with the face you care
#  about most for best accuracy.
# =====================================================================
DIST_ENABLED = True
DIST_UNITS = "ft"               # "ft" or "m"
DIST_CALIB_CM = 100.0           # the known distance you stand at when
                                # you press D to calibrate
DIST_CALIB_FILE = "dist_calib.json"
DIST_CALIB_VERSION = 3          # bump whenever the units of DIST_C
                                # change, so a stale file is ignored
                                # instead of silently giving wrong
                                # numbers. v3 = display pixels (the
                                # original, self-consistent units).
DIST_C_DEFAULT = 8000.0         # rough uncalibrated constant
DIST_SMOOTH = 0.3               # EMA on the readout so it does not jitter

# =====================================================================
#  SPEED / HEIGHT / AGE
#
#  All three are derived FROM the face box, so they only exist while a
#  face is detected. None of them work on a person facing away.
#
#  Speed uses the face itself as an on-screen ruler: a face is roughly
#  FACE_REAL_CM across, so cm-per-pixel = FACE_REAL_CM / face_px. Zoom
#  cancels out because both terms are measured on the same image. The
#  readout combines lateral motion (across frame) with radial motion
#  (toward or away, from the change in distance).
#
#  Height uses the pinhole projection of the head-top against a LEVEL
#  camera at a known mount height:
#      Y = CAM_HEIGHT_CM + dy_px * distance / (zoom * focal_px)
#  Accuracy depends on CAM_HEIGHT_CM being right and the camera being
#  level. Tilt the camera and the number is wrong. Measure your mount
#  height and set it below. Expect a few inches of error at best.
# =====================================================================
FACE_REAL_CM = 16.5             # nominal face size, sqrt(width*height)
CAM_HEIGHT_CM = 100.0           # camera lens height above the floor.
                                # Do not guess. Either measure floor to
                                # LENS CENTER, or stand in view and
                                # press K to solve for it (see below).
MY_HEIGHT_CM = 178.0            # YOUR height, for the K calibration.
                                # 178 cm = 5 ft 10 in. Set this to your
                                # real height before pressing K.
SPEED_ENABLED = True
SPEED_SMOOTH = 0.25             # EMA on the speed readout
SPEED_MIN_SHOW = 0.3            # ft/s below this reads as "still"
HEIGHT_ENABLED = True
HEIGHT_SMOOTH = 0.15            # heavier smoothing, height should be
                                # stable for a given person
AGE_ENABLED = True              # buffalo_l ships a genderage model
AGE_MIN_FACE_PX = 70            # skip age on faces smaller than this.
                                # genderage needs real detail; a 36px
                                # face gives a number, but a meaningless
                                # one. Recognition still runs on smaller
                                # faces, this only gates age/gender.
AGE_VOTE_LEN = 9                # age is noisy frame to frame, so report
                                # the median of the last N readings and
                                # the majority gender
INFO_MODE_DEFAULT = "full"      # basic / full, toggled with I.
                                # "full" shows the speed / height /
                                # age line under each box, the optical
                                # axis, and the detection lag readout.
                                # "basic" hides all of that.

# Name freshness. A track keeps its voted name after the detector stops
# confirming it, which is what makes the label survive a head turn. The
# stored confidence goes stale though, so past this many seconds the box
# greys out and shows "?" instead of a percentage that is no longer
# true. The track itself lives until TRACK_UNCONFIRMED_SEC.
# This is memory of who that box was, NOT recognition of a turned face.
# Real back-turned ID needs a person re-identification model, which this
# project does not have.
NAME_FRESH_SEC = 6.0            # show "?" only when the name really is
                                # stale. MUST stay well above
                                # REVERIFY_SEC (2.5) plus embed latency,
                                # or every re-verify cycle flashes a grey
                                # "?" on a perfectly tracked face.

# =====================================================================
#  VOICE ANNOUNCEMENTS (Piper, offline)
#
#  ONE-TIME SETUP ON THE PI:
#    1. Install Piper (pick one):
#         pipx install piper-tts
#       or
#         pip install piper-tts --break-system-packages
#    2. Download the en_US-amy-medium voice (two files, same folder):
#         mkdir -p ~/piper_voices && cd ~/piper_voices
#         wget https://huggingface.co/rhasspy/piper-voices/resolve/main/en/en_US/amy/medium/en_US-amy-medium.onnx
#         wget https://huggingface.co/rhasspy/piper-voices/resolve/main/en/en_US/amy/medium/en_US-amy-medium.onnx.json
#    3. Make sure a WAV plays out your speaker:
#         speaker-test -t wav -c2   (Ctrl-C to stop), or aplay some.wav
#       If it comes out the wrong jack: sudo raspi-config > System > Audio.
#
#  The script renders "<Name> detected." to a cached WAV the first time
#  each name is needed, then just replays the file. If Piper or the
#  voice model is missing, announcements are disabled with a clear
#  message and the rest of the program runs normally.
# =====================================================================
ANNOUNCE_ENABLED = True
PIPER_VOICE = os.path.expanduser("~/piper_voices/en_US-amy-medium.onnx")
TTS_CACHE_DIR = "tts_cache"
ANNOUNCE_MIN_CONFIDENCE = 60.0  # only speak above this confidence
ANNOUNCE_GONE_SEC = 5.0         # a name must be absent this long before
                                # it can be announced again on return
ANNOUNCE_PHRASE = "{name} detected."

# ======================= LOAD ENCODINGS ==============================
print("[INFO] loading encodings...")

def l2norm_rows(a):
    a = np.asarray(a, dtype=np.float32)
    n = np.linalg.norm(a, axis=1, keepdims=True)
    n[n == 0] = 1.0
    return a / n

with open("encodings_close.pickle", "rb") as f:
    data_close = pickle.loads(f.read())
known_face_encodings_close = l2norm_rows(data_close["encodings"])
known_face_names_close = list(data_close["names"])
known_face_names_close_arr = np.array(known_face_names_close)
print(f"[INFO] close: {len(set(known_face_names_close))} people, "
      f"{len(known_face_names_close)} encodings")

if os.path.exists("encodings_far.pickle"):
    with open("encodings_far.pickle", "rb") as f:
        data_far = pickle.loads(f.read())
    known_face_encodings_far = l2norm_rows(data_far["encodings"])
    known_face_names_far = list(data_far["names"])
    known_face_names_far_arr = np.array(known_face_names_far)
    HAS_FAR_ENCODINGS = True
    print(f"[INFO] far: {len(set(known_face_names_far))} people, "
          f"{len(known_face_names_far)} encodings")
else:
    known_face_encodings_far = known_face_encodings_close
    known_face_names_far = known_face_names_close
    known_face_names_far_arr = known_face_names_close_arr
    HAS_FAR_ENCODINGS = False
    print("[WARN] encodings_far.pickle not found - using close "
          "encodings for all distances")

# ========================= INSIGHTFACE ===============================
print(f"[INFO] loading InsightFace ({MODEL_PACK})...")
_modules = ["detection", "recognition"]
if AGE_ENABLED:
    _modules.append("genderage")
app = FaceAnalysis(
    name=MODEL_PACK,
    allowed_modules=_modules,
    providers=["CPUExecutionProvider"]
)
app.prepare(ctx_id=-1, det_size=DET_SIZE, det_thresh=DET_CONFIDENCE)
rec_model = app.models.get("recognition")
if rec_model is None:
    raise RuntimeError("Recognition model missing from pack "
                       f"'{MODEL_PACK}'")
ga_model = app.models.get("genderage") if AGE_ENABLED else None
_ga_warned = False              # one-shot flag for genderage errors

def warmup_models():
    """Run one throwaway inference through every model at startup.

    ONNX Runtime builds and optimises its graph on the FIRST inference,
    and allocates its arenas then too. Without this, the first real face
    pays that cost, which is why the very first box and the very first
    name take noticeably longer than the rest. Doing it here on a blank
    image moves the cost to load time, where nobody is waiting on it."""
    t0 = time.time()
    blank = np.zeros((DET_SIZE[1], DET_SIZE[0], 3), dtype=np.uint8)
    try:
        app.det_model.detect(blank, max_num=0, metric="default")
    except Exception as e:
        print(f"[WARN] detector warmup failed: {e}")
    # Feed the recognition/genderage models a synthetic face-shaped box
    # so their sessions initialise too.
    try:
        f = Face(bbox=np.array([100, 100, 200, 240], dtype=np.float32),
                 kps=np.array([[130, 150], [170, 150], [150, 180],
                               [132, 205], [168, 205]], dtype=np.float32),
                 det_score=0.99)
        rec_model.get(blank, f)
        if ga_model is not None:
            ga_model.get(blank, f)
    except Exception as e:
        print(f"[WARN] recognition warmup failed: {e}")
    print(f"[INFO] models warmed up in {time.time() - t0:.1f}s "
          "(first real face is now fast)")
if AGE_ENABLED and ga_model is None:
    print("[WARN] genderage model not in pack - age/gender OFF")
warmup_models()
print("[INFO] InsightFace ready")

def detect_only(rgb):
    """Fast pass: SCRFD detection only. Returns (bboxes Nx5, kpss)."""
    bboxes, kpss = app.det_model.detect(rgb, max_num=0, metric="default")
    return bboxes, kpss

def compute_embedding(rgb, bbox, kps, det_score):
    """Heavy pass: ArcFace 512-D embedding for one detected face, plus
    age/gender if that model is loaded (same aligned face, so running
    it here costs one extra small inference instead of a second pass).
    Returns (embedding, age, gender) with age/gender None if disabled.
    NOTE: color handling matches v1 and your enrollment pipeline.
    Do not change the BGR->RGB conversion unless you re-enroll."""
    face = Face(bbox=np.asarray(bbox, dtype=np.float32),
                kps=np.asarray(kps, dtype=np.float32),
                det_score=float(det_score))
    rec_model.get(rgb, face)
    age = gender = None
    if ga_model is not None and (bbox[3] - bbox[1]) >= AGE_MIN_FACE_PX:
        global _ga_warned
        try:
            # COLOR: InsightFace expects BGR (its models swapRB internally).
            # This pipeline passes RGB, inherited from v1, and recognition
            # depends on that staying consistent with enrollment, so it is
            # left alone. But genderage is an ABSOLUTE prediction, not a
            # comparison, so wrong channel order produces wildly wrong
            # ages. Hand this model the BGR it actually wants.
            ga_model.get(cv2.cvtColor(rgb, cv2.COLOR_RGB2BGR), face)
            age = int(face.age) if face.age is not None else None
            # InsightFace: gender 1 = male, 0 = female
            gender = ("M" if int(face.gender) == 1 else "F") \
                if face.gender is not None else None
        except Exception as e:
            # Report once. Silently swallowing this made a blank age
            # field impossible to diagnose.
            if not _ga_warned:
                print(f"[WARN] genderage inference failed ({e}) - "
                      "age/gender will stay blank")
                _ga_warned = True
    return face.embedding, age, gender

def match_face(embedding, active_encodings, active_names_arr, threshold):
    """Cosine matching with a margin test against other identities."""
    if len(active_encodings) == 0:
        return "Unknown", 0.0
    emb = embedding / np.linalg.norm(embedding)
    sims = active_encodings @ emb
    best_idx = int(np.argmax(sims))
    best_score = float(sims[best_idx])
    best_name = active_names_arr[best_idx]
    other_mask = active_names_arr != best_name
    best_other = float(sims[other_mask].max()) if other_mask.any() else -1.0
    confidence = round(best_score * 100, 1)
    if best_score >= threshold and (best_score - best_other) >= MATCH_MARGIN:
        return str(best_name), confidence
    return "Unknown", confidence

# =========================== CAMERA ==================================
picam2 = Picamera2()

try:
    SENSOR_NATIVE = tuple(picam2.sensor_resolution)
except Exception:
    SENSOR_NATIVE = tuple(picam2.camera_properties["PixelArraySize"])
SENSOR_BINNED = (SENSOR_NATIVE[0] // 2, SENSOR_NATIVE[1] // 2)
SENSOR_FULL = (min(SENSOR_NATIVE[0], SENSOR_MAX_REQ[0]),
               min(SENSOR_NATIVE[1], SENSOR_MAX_REQ[1]))
SENSOR_SIZES = {"BINNED": SENSOR_BINNED, "FULL": SENSOR_FULL}

# Camera orientation. These feed BOTH the camera Transform and the
# optical-axis math used by height. Keep them in sync by only ever
# changing them here.
HFLIP = True
VFLIP = True

def make_config(sensor_size, buffers):
    """The 'sensor' hint selects the readout mode WITHOUT allocating
    raw buffers to the app. On a Camera Module 3 in full mode this
    saves roughly 60 MB of contiguous memory versus requesting a raw
    stream. Falls back to a raw request on older picamera2."""
    common = dict(
        main={"size": (MAIN_W, MAIN_H), "format": "RGB888"},
        lores={"size": (LORES_W, LORES_H), "format": "YUV420"},
        transform=Transform(hflip=HFLIP, vflip=VFLIP),
        buffer_count=buffers
    )
    try:
        return picam2.create_preview_configuration(
            sensor={"output_size": sensor_size}, **common)
    except TypeError:
        return picam2.create_preview_configuration(
            raw={"size": sensor_size}, **common)

CONFIGS = {"BINNED": make_config(SENSOR_BINNED, BUFFERS_BINNED),
           "FULL": make_config(SENSOR_FULL, BUFFERS_FULL)}

current_sensor = "BINNED"
sensor_pref = "AUTO"            # AUTO / BINNED / FULL, key H cycles
last_sensor_switch = 0.0

picam2.configure(CONFIGS[current_sensor])
picam2.start()

def af_kick():
    try:
        picam2.set_controls({
            "AfMode": controls.AfModeEnum.Auto,
            "AfTrigger": controls.AfTriggerEnum.Start,
            "AfSpeed": controls.AfSpeedEnum.Fast,
            "AfRange": controls.AfRangeEnum.Full
        })
        time.sleep(0.4)
        picam2.set_controls({
            "AfMode": controls.AfModeEnum.Continuous,
            "AfSpeed": controls.AfSpeedEnum.Fast,
            "AfRange": controls.AfRangeEnum.Full
        })
    except Exception:
        pass  # sensor without autofocus

time.sleep(1.5)
af_kick()

# ============================ STATE ==================================
zoom_factor = 1.5
encoding_mode = "auto"          # auto / close / far, key E cycles

result_lock = Lock()
face_locations = []             # MAIN-frame coords (top,right,bottom,left)
face_names = []
face_confidences = []
face_zones = []
face_ages = []                  # per-face (age, gender) or None
info_mode = INFO_MODE_DEFAULT   # basic / full, toggled with I
last_result_time = 0.0

auto_zoom_enabled = True
auto_zoom_target = 1.5
auto_zoom_current = 1.5
pan_current_x = 0.5
pan_current_y = 0.5
pan_target_x_abs = 0.5          # absolute sensor-space pan targets,
pan_target_y_abs = 0.5          # stepped once per worker result
consumed_result_time = 0.0
last_applied_zoom = zoom_factor
last_applied_pan_x = 0.5
last_applied_pan_y = 0.5

zones = []                      # display coords
selecting = False
select_start = None
select_current = None
mouse_pos = (DISPLAY_W // 2, DISPLAY_H // 2)
selection_mode = False
frozen_main = None
frozen_display = None

frame_count = 0
start_time = time.time()
fps = 0
processing = False
latest_main_frame = None
latest_frame_meta = (zoom_factor, 0.5, 0.5, 0.0)  # (zoom, panx, pany,
                                                  # capture_time) for the
                                                  # frame handed to the
                                                  # worker
result_meta = (zoom_factor, 0.5, 0.5, 0.0)        # same, for the latest
                                                  # published results
main_frame_lock = Lock()

# Screenshots. L saves exactly what is on screen (boxes, labels, HUD).
SCREENSHOT_DIR = "screenshot"
screenshot_flash_until = 0.0                  # brief on-screen confirm

tracked_faces = []
last_reseed_time = 0.0

identity_cache = []             # worker-only: {cx,cy,h,name,conf,t,seen}

# ============================ ZOOM ===================================
def apply_zoom(z, cxf=0.5, cyf=0.5):
    """ScalerCrop in full pixel-array coordinates (valid across sensor
    modes). cxf/cyf: fraction of the sensor to center in the crop."""
    fw, fh = picam2.camera_properties["PixelArraySize"]
    cw = int(fw / z)
    ch = int(fh / z)
    cx = int(cxf * fw - cw / 2)
    cy = int(cyf * fh - ch / 2)
    cx = max(0, min(cx, fw - cw))
    cy = max(0, min(cy, fh - ch))
    picam2.set_controls({"ScalerCrop": (cx, cy, cw, ch)})

def update_zoom_full(z, cxf=0.5, cyf=0.5):
    apply_zoom(z, cxf, cyf)
    af_kick()

def get_active_encodings():
    """Returns (encodings, names_arr, using_far)."""
    if encoding_mode == "close":
        return known_face_encodings_close, known_face_names_close_arr, False
    if encoding_mode == "far":
        if HAS_FAR_ENCODINGS:
            return known_face_encodings_far, known_face_names_far_arr, True
        return known_face_encodings_close, known_face_names_close_arr, False
    if HAS_FAR_ENCODINGS and zoom_factor >= FAR_ZOOM_THRESHOLD:
        return known_face_encodings_far, known_face_names_far_arr, True
    return known_face_encodings_close, known_face_names_close_arr, False

def cycle_encoding_mode():
    global encoding_mode
    if encoding_mode == "auto":
        encoding_mode = "close"
    elif encoding_mode == "close":
        encoding_mode = "far" if HAS_FAR_ENCODINGS else "auto"
    else:
        encoding_mode = "auto"
    print(f"[INFO] Encoding mode: {encoding_mode.upper()}")

# ===================== SENSOR MODE SWITCHING =========================
def switch_sensor(target):
    """Reconfigure the camera to a different raw sensor mode. Takes
    about a second. FULL mode gives max detail for ScalerCrop at high
    zoom; on Camera Module 3 the sensor tops out near 14 fps there."""
    global current_sensor, last_sensor_switch, tracked_faces, sensor_pref
    global cache_reset
    if target == current_sensor:
        return
    print(f"[INFO] Sensor mode -> {target} ({SENSOR_SIZES[target]})")
    picam2.stop()
    try:
        picam2.configure(CONFIGS[target])
        picam2.start()
    except Exception as e:
        print(f"[WARN] {target} mode failed ({e}); staying BINNED. "
              "Lower MAIN_W/MAIN_H or raise CMA if this persists.")
        target = "BINNED"
        sensor_pref = "BINNED"
        picam2.configure(CONFIGS[target])
        picam2.start()
    current_sensor = target
    last_sensor_switch = time.time()
    time.sleep(0.25)
    apply_zoom(zoom_factor, pan_current_x, pan_current_y)
    af_kick()
    tracked_faces = []          # old templates no longer match
    cache_reset = True          # cached identities point at old coords

def desired_sensor():
    if sensor_pref in ("BINNED", "FULL"):
        return sensor_pref
    if zoom_factor >= ZOOM_FULLRES_ON:
        return "FULL"
    if zoom_factor <= ZOOM_FULLRES_OFF:
        return "BINNED"
    return current_sensor       # hysteresis band, hold

def maybe_switch_sensor():
    target = desired_sensor()
    if target != current_sensor and \
       time.time() - last_sensor_switch > SENSOR_SWITCH_COOLDOWN:
        switch_sensor(target)

def cycle_sensor_pref():
    global sensor_pref
    order = ["AUTO", "BINNED", "FULL"]
    sensor_pref = order[(order.index(sensor_pref) + 1) % 3]
    print(f"[INFO] Sensor preference: {sensor_pref}")

apply_zoom(zoom_factor)

# ======================== COORD HELPERS ==============================
SX = DISPLAY_W / MAIN_W         # main -> display
SY = DISPLAY_H / MAIN_H

def display_to_main(x, y):
    return int(x * MAIN_W / DISPLAY_W), int(y * MAIN_H / DISPLAY_H)

def normalize_zone(z):
    x1, y1, x2, y2 = z
    return (min(x1, x2), min(y1, y2), max(x1, x2), max(y1, y2))

def crop_zone_from_main(frame, zone):
    """Crop a display-coordinate zone out of a MAIN-resolution frame.
    Returns (crop, mx1, my1) or (None, 0, 0)."""
    x1, y1, x2, y2 = zone
    mx1, my1 = display_to_main(x1, y1)
    mx2, my2 = display_to_main(x2, y2)
    mx1 = max(0, mx1); my1 = max(0, my1)
    mx2 = min(MAIN_W, mx2); my2 = min(MAIN_H, my2)
    if mx2 <= mx1 or my2 <= my1:
        return None, 0, 0
    return frame[my1:my2, mx1:mx2], mx1, my1

# ======================== MOUSE CALLBACK =============================
def mouse_callback(event, x, y, flags, param):
    global selecting, select_start, select_current, zones, mouse_pos
    mouse_pos = (x, y)
    if not selection_mode:
        return
    if event == cv2.EVENT_LBUTTONDOWN:
        selecting = True
        select_start = (x, y)
        select_current = (x, y)
    elif event == cv2.EVENT_MOUSEMOVE and selecting:
        select_current = (x, y)
    elif event == cv2.EVENT_LBUTTONUP and selecting:
        selecting = False
        if select_start and select_current:
            zone = normalize_zone((*select_start, *select_current))
            w = zone[2] - zone[0]
            h = zone[3] - zone[1]
            if w > 20 and h > 20:
                if len(zones) < MAX_ZONES:
                    zones.append(zone)
                    print(f"Zone {len(zones)} set")
                else:
                    print("Max 3 zones - press C to clear")
        select_start = None
        select_current = None
    elif event == cv2.EVENT_RBUTTONDOWN:
        for i in range(len(zones) - 1, -1, -1):
            x1, y1, x2, y2 = zones[i]
            if x1 <= x <= x2 and y1 <= y <= y2:
                zones.pop(i)
                print(f"Zone {i + 1} deleted")
                break

if not HEADLESS:
    cv2.namedWindow("Face Recognition")
    cv2.setMouseCallback("Face Recognition", mouse_callback)
else:
    print("[INFO] headless: no display, browser stream only")

# ===================== IDENTITY CACHE (worker) =======================
cache_reset = False             # set by the main thread after camera
                                # moves or zone edits; worker clears
                                # the cache at the next pass
cache_lock = Lock()             # identity_cache is now touched by both
                                # the detect thread and the embed thread
embed_q = Queue(maxsize=2)      # detect -> embed handoff. Small on
                                # purpose: a stale face is not worth
                                # embedding, so old jobs get dropped

def cache_match(cx, cy, fh):
    """A detection only inherits a cached identity when close AND of
    similar size. Without the size check, a false detection on a wall
    near a person could pick up their name without any embedding.

    The tolerance has to cover how far a face MOVES between detection
    passes, not just detector jitter. At ~2.5 passes/sec a brisk walk
    covers ~240px, which the old 180px tolerance missed, so the name
    dropped out every time you moved. The size check is what keeps this
    safe despite the generous radius."""
    best = None
    best_d = max(CACHE_MATCH_MIN_PX, fh * CACHE_MATCH_FACE_FRAC)
    for e in identity_cache:
        ratio = fh / max(e["h"], 1.0)
        if ratio < 0.5 or ratio > 2.0:
            continue
        d = ((e["cx"] - cx) ** 2 + (e["cy"] - cy) ** 2) ** 0.5
        if d < best_d:
            best_d = d
            best = e
    return best

def cache_prune(now):
    global identity_cache
    identity_cache = [e for e in identity_cache
                      if now - e["seen"] <= CACHE_SEEN_TTL]

# ====================== RECOGNITION WORKER ===========================
def recognition_worker():
    """Two-phase pipeline per pass.

    Phase 1 (fast): SCRFD detection on the whole frame/zones. Boxes are
    published immediately with the last known name from the identity
    cache, or a scanning placeholder for brand-new faces. This is what
    makes the box appear on your face quickly instead of waiting for
    the slow embedding step.

    Phase 2 (slow): ArcFace embeddings for faces that are new or stale,
    up to MAX_EMBEDS_PER_PASS. Names are then republished. So the box
    tracks in near real time and the label sharpens a moment later."""
    global face_locations, face_names, face_confidences, face_zones
    global processing, last_result_time, cache_reset, result_meta
    global face_ages

    def publish(locs, names, confs, fzones, ages, meta):
        global face_locations, face_names, face_confidences, face_zones
        global result_meta, last_result_time, face_ages
        with result_lock:
            face_locations = list(locs)
            face_names = list(names)
            face_confidences = list(confs)
            face_zones = list(fzones)
            face_ages = list(ages)
            result_meta = meta
            last_result_time = time.time()

    while True:
        if not processing:
            time.sleep(0.005)
            continue

        if cache_reset:
            with cache_lock:
                identity_cache.clear()
            cache_reset = False

        with main_frame_lock:
            frame = latest_main_frame.copy() if latest_main_frame is not None else None
            frame_meta = latest_frame_meta

        if frame is None:
            processing = False
            continue

        active_encodings, active_names_arr, using_far = get_active_encodings()
        base_threshold = ZONE_THRESHOLD if using_far else SIMILARITY_THRESHOLD

        now = time.time()
        locations = []
        names = []
        confs = []
        zones_found = []
        ages = []
        jobs = []           # deferred embed jobs

        def add_detection(rgb, bbox_local, kps_local, det_score,
                          off_x, off_y, inv_scale, zone_idx, threshold):
            lx1, ly1, lx2, ly2 = bbox_local
            mx1 = off_x + lx1 * inv_scale
            my1 = off_y + ly1 * inv_scale
            mx2 = off_x + lx2 * inv_scale
            my2 = off_y + ly2 * inv_scale
            mx1 = max(0.0, mx1); my1 = max(0.0, my1)
            mx2 = min(float(MAIN_W), mx2); my2 = min(float(MAIN_H), my2)
            if mx2 <= mx1 or my2 <= my1:
                return
            fh = my2 - my1
            cx = (mx1 + mx2) / 2
            cy = (my1 + my2) / 2

            with cache_lock:
                entry = cache_match(cx, cy, fh)
                if entry is not None:
                    entry["cx"], entry["cy"], entry["h"] = cx, cy, fh
                    entry["seen"] = now
                    name, conf = entry["name"], entry["conf"]
                    ag = entry.get("ag")
                else:
                    name, conf = PENDING, 0.0    # not yet embedded
                    ag = None

            idx = len(locations)
            locations.append((int(my1), int(mx2), int(my2), int(mx1)))
            names.append(name)
            confs.append(conf)
            zones_found.append(zone_idx)
            ages.append(ag)

            with cache_lock:
                fresh = (entry is not None
                         and (now - entry["t"]) <= REVERIFY_SEC)
            if (not fresh and fh >= MIN_FACE_EMBED_PX
                    and kps_local is not None):
                jobs.append({
                    "idx": idx, "rgb": rgb, "bbox": bbox_local,
                    "kps": kps_local, "score": det_score,
                    "cx": cx, "cy": cy, "fh": fh,
                    "entry": entry, "threshold": threshold,
                })

        # ---- Phase 1: detection ----
        if zones:
            for zone_idx, zone in enumerate(zones):
                crop, off_x, off_y = crop_zone_from_main(frame, zone)
                if crop is None or crop.size == 0:
                    continue
                up = 2 if min(crop.shape[0], crop.shape[1]) < ZONE_UPSCALE_MIN else 1
                if up > 1:
                    crop = cv2.resize(crop, None, fx=up, fy=up,
                                      interpolation=cv2.INTER_CUBIC)
                rgb = cv2.cvtColor(crop, cv2.COLOR_BGR2RGB)
                bboxes, kpss = detect_only(rgb)
                for i in range(len(bboxes)):
                    x1, y1, x2, y2, score = bboxes[i]
                    kps = kpss[i] if kpss is not None else None
                    add_detection(rgb, (x1, y1, x2, y2), kps, score,
                                  off_x, off_y, 1.0 / up,
                                  zone_idx, ZONE_THRESHOLD)
        else:
            rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            bboxes, kpss = detect_only(rgb)
            for i in range(len(bboxes)):
                x1, y1, x2, y2, score = bboxes[i]
                kps = kpss[i] if kpss is not None else None
                add_detection(rgb, (x1, y1, x2, y2), kps, score,
                              0, 0, 1.0, -1, base_threshold)

        publish(locations, names, confs, zones_found, ages, frame_meta)

        # ---- Hand the slow work to the embed thread ----
        # Embeddings used to run right here, which meant the next
        # detection could not start until they finished. Box refresh was
        # therefore gated by the SLOWEST step, so boxes updated about
        # once a second. Now detection loops at detection speed and the
        # embed thread fills in names alongside it. Names land a beat
        # later via the identity cache, which is the right trade: a box
        # in the right place immediately beats a name a moment sooner.
        jobs.sort(key=lambda j: j["fh"], reverse=True)
        for job in jobs[:MAX_EMBEDS_PER_PASS]:
            job["encodings"] = active_encodings
            job["names_arr"] = active_names_arr
            job["t"] = now
            try:
                embed_q.put_nowait(job)
            except Full:
                pass        # embedder busy; this face gets the next pass

        with cache_lock:
            cache_prune(now)
        processing = False

def embed_worker():
    """Runs ArcFace (and genderage) off the detection thread, writing
    results into the identity cache. The detect loop reads that cache,
    so names appear on the next detection pass."""
    while True:
        job = embed_q.get()
        try:
            emb, age, gender = compute_embedding(job["rgb"], job["bbox"],
                                                 job["kps"], job["score"])
            if emb is None:
                continue
            name, conf = match_face(emb, job["encodings"],
                                    job["names_arr"], job["threshold"])
            ag = (age, gender) if age is not None else None
            now = time.time()
            with cache_lock:
                # Always re-match under the lock rather than trusting the
                # entry the detect thread captured. That reference can be
                # seconds old by now, and cache_prune rebuilds the list,
                # so a stale reference would take the result into a dict
                # no longer in the cache and the name would be lost.
                entry = cache_match(job["cx"], job["cy"], job["fh"])
                if entry is None:
                    identity_cache.append({
                        "cx": job["cx"], "cy": job["cy"], "h": job["fh"],
                        "name": name, "conf": conf, "t": now, "seen": now,
                        "ag": ag})
                else:
                    entry["name"] = name
                    entry["conf"] = conf
                    entry["t"] = now
                    if ag is not None:
                        entry["ag"] = ag
        except Exception as e:
            print(f"[WARN] embed failed: {e}")

Thread(target=embed_worker, daemon=True).start()

worker_thread = Thread(target=recognition_worker, daemon=True)
worker_thread.start()

# ===================== VOICE ANNOUNCER (Piper) =======================
announce_queue = Queue()
present_names = {}          # name -> last time seen present (main thread)
announced_names = set()     # names currently considered present
_piper_ok = False

def _piper_available():
    """True if the piper binary and the voice model are both present."""
    if not ANNOUNCE_ENABLED:
        return False
    if not os.path.exists(PIPER_VOICE):
        print(f"[AUDIO] voice model not found at {PIPER_VOICE} - "
              "announcements OFF (see setup notes near the top)")
        return False
    if not os.path.exists(PIPER_VOICE + ".json"):
        print(f"[AUDIO] {PIPER_VOICE}.json missing - announcements OFF")
        return False
    from shutil import which
    if which("piper") is None:
        print("[AUDIO] 'piper' not on PATH - announcements OFF "
              "(pipx install piper-tts)")
        return False
    if which("aplay") is None:
        print("[AUDIO] 'aplay' not found - announcements OFF "
              "(sudo apt install alsa-utils)")
        return False
    return True

def _safe_filename(name):
    return "".join(c if c.isalnum() else "_" for c in name)

def _render_phrase(name):
    """Return a path to a cached WAV of '<name> detected.', rendering it
    with Piper on first use. None on failure."""
    os.makedirs(TTS_CACHE_DIR, exist_ok=True)
    path = os.path.join(TTS_CACHE_DIR, _safe_filename(name) + ".wav")
    if os.path.exists(path):
        return path
    phrase = ANNOUNCE_PHRASE.format(name=name)
    try:
        subprocess.run(
            ["piper", "--model", PIPER_VOICE, "--output_file", path],
            input=phrase.encode("utf-8"),
            stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL,
            check=True, timeout=30)
        print(f"[AUDIO] rendered '{phrase}' -> {path}")
        return path
    except Exception as e:
        print(f"[AUDIO] Piper render failed for '{name}': {e}")
        return None

def audio_worker():
    """Plays queued names one at a time so speech never overlaps and
    never blocks the video loop."""
    while True:
        name = announce_queue.get()
        if name is None:
            continue
        path = _render_phrase(name)
        if path is None:
            continue
        try:
            subprocess.run(["aplay", "-q", path],
                           stdout=subprocess.DEVNULL,
                           stderr=subprocess.DEVNULL, timeout=15)
        except Exception as e:
            print(f"[AUDIO] playback failed for '{name}': {e}")

_piper_ok = _piper_available()
if _piper_ok:
    Thread(target=audio_worker, daemon=True).start()
    print("[AUDIO] announcements ON (Piper, en_US-amy-medium)")

def update_announcements():
    """Called each frame. Builds the set of confident, known names on
    screen right now, announces any that just arrived, and forgets any
    that have been gone longer than ANNOUNCE_GONE_SEC. A brief
    confidence dip keeps the name present (we only refresh 'seen' when
    it is confidently identified), so it does not re-announce."""
    if not _piper_ok:
        return
    now = time.time()

    # Names confidently on screen this frame.
    seen_now = set()
    for t in tracked_faces:
        name = track_display_name(t)
        if name is None or name == "Unknown":
            continue
        if t["confidence"] >= ANNOUNCE_MIN_CONFIDENCE:
            seen_now.add(name)

    # Refresh last-seen time for anyone confidently visible.
    for name in seen_now:
        present_names[name] = now

    # Announce genuinely new arrivals.
    for name in seen_now:
        if name not in announced_names:
            announced_names.add(name)
            announce_queue.put(name)

    # Drop anyone not seen for the gone timeout, so they can announce
    # again next time they appear.
    for name in list(announced_names):
        if now - present_names.get(name, 0) > ANNOUNCE_GONE_SEC:
            announced_names.discard(name)
            present_names.pop(name, None)

# ========================== AUTO ZOOM ================================
def compute_auto_zoom(locs, current_z):
    if not locs:
        return max(1.5, current_z - 0.05)
    largest = max(locs, key=lambda b: (b[2] - b[0]) * (b[1] - b[3]))
    top, right, bottom, left = largest
    ratio = ((bottom - top) * (right - left)) / (MAIN_H * MAIN_W)
    if ratio < 0.03:
        return min(current_z + 0.15, 6.0)
    if ratio < 0.06:
        return min(current_z + 0.05, 6.0)
    if ratio > 0.25:
        return max(current_z - 0.1, 1.5)
    return current_z

def compute_auto_pan(locs):
    """Largest face center as a fraction (0..1) of the MAIN frame."""
    if not locs:
        return None
    largest = max(locs, key=lambda b: (b[2] - b[0]) * (b[1] - b[3]))
    top, right, bottom, left = largest
    cx = (left + right) / 2
    cy = (top + bottom) / 2
    return cx / MAIN_W, cy / MAIN_H

# =============== FAST BOX TRACKING (template matching) ===============
def get_identified_faces():
    """Latest worker results converted MAIN -> DISPLAY coords."""
    with result_lock:
        age = time.time() - last_result_time
        if age > IDENTITY_STALE_SEC:
            return []
        locs = list(face_locations)
        names = list(face_names)
        confs = list(face_confidences)
        fzones = list(face_zones)
        fages = list(face_ages)
        meta_zoom = result_meta[0] if result_meta else zoom_factor

    if len(fages) != len(locs):
        fages = [None] * len(locs)

    identities = []
    for (top, right, bottom, left), name, conf, zone_idx, ag in zip(
            locs, names, confs, fzones, fages):
        # Keep the TIGHT SCRFD box for tracking. The template is
        # grabbed from this region, so it stays on face texture and
        # never picks up the wall. Expansion happens only at draw time.
        top_d = int(top * SY)
        bottom_d = int(bottom * SY)
        left_d = int(left * SX)
        right_d = int(right * SX)
        identities.append({
            "box": (top_d, right_d, bottom_d, left_d),
            # Raw detector output in MAIN pixels. Distance and height are
            # measured from THIS, never from the display-scaled box: the
            # display path adds an aspect/scaling factor, and the tracked
            # box is size-eased so it lags the true face size.
            "m_box": (float(top), float(right), float(bottom), float(left)),
            "m_zoom": meta_zoom,
            "cx": (left_d + right_d) / 2,
            "cy": (top_d + bottom_d) / 2,
            "name": name,
            "confidence": conf,
            "zone_idx": zone_idx,
            "ag": ag,
        })
    return identities

def expand_box(box):
    """Grow a tight face box to frame the whole head, for DISPLAY only.
    Never fed back into the tracker template."""
    top, right, bottom, left = box
    bw = right - left
    bh = bottom - top
    left = max(0, int(left - bw * BOX_EXPAND_X / 2))
    right = min(DISPLAY_W - 1, int(right + bw * BOX_EXPAND_X / 2))
    top = max(0, int(top - bh * BOX_EXPAND_TOP))
    bottom = min(DISPLAY_H - 1, int(bottom + bh * BOX_EXPAND_BOTTOM))
    return top, right, bottom, left

def update_tracks(gray_frame, dt):
    """Move every box to where its face is right now.

    Velocity is kept in px/SECOND, so prediction and the speed readout
    stay correct even when the frame rate wobbles.

    On a failed match the box COASTS: it keeps gliding at its last known
    velocity for up to COAST_MAX_FRAMES instead of freezing. Freezing was
    what made the box stutter, because a frozen box also killed the
    prediction, which pushed the search window further behind the face
    and caused the next match to fail too."""
    global tracked_faces
    h, w = gray_frame.shape[:2]
    if dt <= 0:
        dt = 1 / 30.0
    for t in tracked_faces:
        top, right, bottom, left = t["box"]
        vx = t.get("vx", 0.0)           # px/sec
        vy = t.get("vy", 0.0)
        # Predict where the face is heading and center the search there,
        # so a fast head move does not outrun the search window.
        ox = vx * dt * PREDICT_GAIN
        oy = vy * dt * PREDICT_GAIN
        m = TRACK_SEARCH_MARGIN
        sx1 = int(max(0, left + ox - m)); sy1 = int(max(0, top + oy - m))
        sx2 = int(min(w, right + ox + m)); sy2 = int(min(h, bottom + oy + m))
        search = gray_frame[sy1:sy2, sx1:sx2]
        tmpl = t["template"]
        th, tw = tmpl.shape[:2]

        matched = False
        if not (th == 0 or tw == 0 or search.shape[0] < th
                or search.shape[1] < tw):
            res = cv2.matchTemplate(search, tmpl, cv2.TM_CCOEFF_NORMED)
            _, max_val, _, max_loc = cv2.minMaxLoc(res)
            # Already coasting means we are committed to this face, so
            # accept a weaker match to regain lock rather than stall.
            bar = MATCH_THRESHOLD_COAST if t.get("coast", 0) > 0 \
                else MATCH_THRESHOLD
            if max_val >= bar:
                matched = True
                new_left = sx1 + max_loc[0]
                new_top = sy1 + max_loc[1]
                new_right = new_left + tw
                new_bottom = new_top + th
                ocx, ocy = (left + right) / 2.0, (top + bottom) / 2.0
                ncx = (new_left + new_right) / 2.0
                ncy = (new_top + new_bottom) / 2.0
                mvx = (ncx - ocx) / dt          # px/sec
                mvy = (ncy - ocy) / dt
                t["vx"] = vx + (mvx - vx) * VEL_SMOOTH
                t["vy"] = vy + (mvy - vy) * VEL_SMOOTH
                t["box"] = (new_top, new_right, new_bottom, new_left)
                t["coast"] = 0
                t["coast_px"] = 0.0

                # Refresh rate follows motion. Moving fast, the face is
                # blurred, so adopt the blurred patch quickly or the next
                # match fails. Standing still, adopt slowly so the
                # template cannot drift off-face.
                spd = (t["vx"] ** 2 + t["vy"] ** 2) ** 0.5
                k = min(1.0, spd / BLEND_SPEED_FULL)
                blend = TEMPLATE_BLEND_STILL + \
                    (TEMPLATE_BLEND_MOVING - TEMPLATE_BLEND_STILL) * k
                new_tmpl = gray_frame[new_top:new_bottom, new_left:new_right]
                if (new_tmpl.size > 0 and new_tmpl.std() >= MIN_TEMPLATE_STD
                        and new_tmpl.shape == tmpl.shape):
                    t["template"] = cv2.addWeighted(tmpl, 1.0 - blend,
                                                    new_tmpl, blend, 0)

        if not matched:
            c = t.get("coast", 0) + 1
            t["coast"] = c
            dx = vx * dt
            dy = vy * dt
            travelled = t.get("coast_px", 0.0) + (dx * dx + dy * dy) ** 0.5
            budget = max(COAST_MAX_PX, (right - left) * COAST_MAX_FACE_FRAC)
            if (c <= COAST_MAX_FRAMES and travelled <= budget
                    and (abs(vx) > 1 or abs(vy) > 1)):
                # Dead reckoning: keep gliding so the box stays on a
                # moving face through motion blur.
                t["coast_px"] = travelled
                nl = int(max(0, min(w - (right - left), left + dx)))
                nt = int(max(0, min(h - (bottom - top), top + dy)))
                t["box"] = (nt, nl + (right - left), nt + (bottom - top), nl)
                t["vx"] = vx * COAST_VEL_DECAY
                t["vy"] = vy * COAST_VEL_DECAY
            else:
                # Out of coast budget. Stop moving and wait for the
                # detector rather than drifting off into the scene.
                t["vx"] = vx * 0.5
                t["vy"] = vy * 0.5

def track_iou(a, b):
    """Overlap between two track boxes, 0 to 1."""
    at, ar, ab, al = a["box"]
    bt, br, bb, bl = b["box"]
    ix1, iy1 = max(al, bl), max(at, bt)
    ix2, iy2 = min(ar, br), min(ab, bb)
    iw, ih = max(0, ix2 - ix1), max(0, iy2 - iy1)
    inter = iw * ih
    if inter <= 0:
        return 0.0
    aa = max(1, (ar - al) * (ab - at))
    ba = max(1, (br - bl) * (bb - bt))
    return inter / float(aa + ba - inter)

def tracks_collide(a, b):
    """True if these two tracks are really the same person. Either they
    overlap, or their centers sit closer than a face width apart, which
    catches a box parked beside someone rather than on them."""
    if track_iou(a, b) > TRACK_MERGE_IOU:
        return True
    at, ar, ab, al = a["box"]
    bt, br, bb, bl = b["box"]
    acx, acy = (al + ar) / 2.0, (at + ab) / 2.0
    bcx, bcy = (bl + br) / 2.0, (bt + bb) / 2.0
    d = ((acx - bcx) ** 2 + (acy - bcy) ** 2) ** 0.5
    size = (((ar - al) + (br - bl)) / 2.0 +
            ((ab - at) + (bb - bt)) / 2.0) / 2.0
    return d < size * TRACK_MERGE_CENTER_FRAC

def suppress_duplicate_tracks():
    """Collapse duplicate boxes for one person. Keeps whichever track the
    detector confirmed most recently, since that is the one actually on
    the face. The loser is dropped, not merged, because its template is
    the drifted one we want gone.

    Two rules:
      1. Boxes that overlap or sit close together are the same face.
      2. Boxes carrying the SAME NAME are the same person no matter how
         far apart they are. A track that lost lock and coasted away
         still wears the name, so proximity alone never catches it.
    """
    global tracked_faces
    if len(tracked_faces) < 2:
        return
    now = time.time()
    def rank(t):
        return (t.get("id_time", 0.0), len(t.get("hist", ())),
                -t.get("misses", 0))
    order = sorted(range(len(tracked_faces)),
                   key=lambda i: rank(tracked_faces[i]), reverse=True)
    dropped = set()
    for pos, i in enumerate(order):
        if i in dropped:
            continue
        ni = track_display_name(tracked_faces[i])
        for j in order[pos + 1:]:
            if j in dropped:
                continue
            tj = tracked_faces[j]
            # Same name means same person ONLY if this copy is not
            # currently being confirmed by the detector. A real duplicate
            # (a track that lost lock and coasted off) goes unconfirmed
            # and stale. Two real people who happen to be misidentified
            # as each other are BOTH live, and dropping one would delete
            # a real person's box, so live tracks are always kept.
            stale = (tj.get("misses", 0) >= SAME_NAME_STALE_MISSES
                     or now - tj.get("seen_time", 0.0) > SAME_NAME_STALE_SEC)
            same_name = (ni is not None and ni != "Unknown"
                         and track_display_name(tj) == ni and stale)
            if same_name or tracks_collide(tracked_faces[i], tj):
                dropped.add(j)
    if dropped:
        tracked_faces = [t for i, t in enumerate(tracked_faces)
                         if i not in dropped]

def expire_stale_tracks():
    """Drop tracks the detector has not confirmed in TRACK_UNCONFIRMED_SEC.
    Bounds ghost boxes in real time rather than in detector passes."""
    global tracked_faces
    now = time.time()
    tracked_faces = [
        t for t in tracked_faces
        if now - t.get("seen_time", now) <= TRACK_UNCONFIRMED_SEC
    ]

def reseed_tracks(gray_frame, identities, lag=0.0):
    """Fold a (possibly lagged) detector pass into the live tracks.

    Detection is slow, so its POSITION is stale. This function does NOT
    move the box to the detection. The template tracker already owns
    position from the live frame. Detection only:
      - keeps a matched track alive (resets its miss counter),
      - votes the identity/name,
      - eases the box SIZE toward the detected face size, keeping the
        tracker's current center. This is what makes the box frame the
        whole face as you move closer or further, fixing 'only part of
        my face', without dragging the box back to where you were.
    A detection spawns a NEW track only when it is far from every
    existing track, so a lagged detection of a person already tracked
    can no longer create a phantom box behind them."""
    global tracked_faces
    h, w = gray_frame.shape[:2]
    used_tracks = set()
    n_before = len(tracked_faces)

    for ident in identities:
        cx, cy = ident["cx"], ident["cy"]
        matched = None
        best_dist = RESEED_SEARCH_MARGIN
        for i in range(n_before):
            if i in used_tracks:
                continue
            top, right, bottom, left = tracked_faces[i]["box"]
            tcx, tcy = (left + right) / 2, (top + bottom) / 2
            d = ((tcx - cx) ** 2 + (tcy - cy) ** 2) ** 0.5
            if d < best_dist:
                best_dist = d
                matched = i

        if matched is not None:
            t = tracked_faces[matched]
            bt, br, bb, bl = t["box"]
            tcx, tcy = (bl + br) / 2.0, (bt + bb) / 2.0

            # Lag compensation. The detection reports where the face was
            # at capture time, `lag` seconds ago. Project it forward by
            # the tracker's measured velocity to get where the face is
            # NOW, then snap onto that. Blending only partway (the old
            # behaviour) left a residual offset every pass, and since the
            # template kept drifting between passes, the box settled onto
            # a neck or shoulder instead of the face.
            dcx, dcy = cx, cy
            if LAG_COMPENSATE and 0 < lag <= LAG_MAX_SEC:
                sx = t.get("vx", 0.0) * lag
                sy = t.get("vy", 0.0) * lag
                sx = max(-LAG_MAX_SHIFT_PX, min(LAG_MAX_SHIFT_PX, sx))
                sy = max(-LAG_MAX_SHIFT_PX, min(LAG_MAX_SHIFT_PX, sy))
                dcx += sx
                dcy += sy

            tcx += (dcx - tcx) * SNAP_BLEND
            tcy += (dcy - tcy) * SNAP_BLEND

            cur_w, cur_h = br - bl, bb - bt
            dt, dr, db, dl = ident["box"]
            det_w, det_h = dr - dl, db - dt
            new_w = cur_w + (det_w - cur_w) * SIZE_BLEND
            new_h = cur_h + (det_h - cur_h) * SIZE_BLEND
            left = max(0, int(tcx - new_w / 2))
            right = min(w, int(tcx + new_w / 2))
            top = max(0, int(tcy - new_h / 2))
            bottom = min(h, int(tcy + new_h / 2))
            if right > left and bottom > top:
                t["box"] = (top, right, bottom, left)
                tmpl = gray_frame[top:bottom, left:right].copy()
                if tmpl.size > 0 and tmpl.std() >= MIN_TEMPLATE_STD:
                    t["template"] = tmpl
            if ident["name"] != PENDING:
                t["hist"].append(ident["name"])
                t["confidence"] = ident["confidence"]
                t["id_time"] = time.time()   # for identity persistence
            if ident.get("m_box") is not None:
                t["m_box"] = ident["m_box"]
                t["m_zoom"] = ident["m_zoom"]
            if ident.get("ag") is not None:
                t["ag"] = ident["ag"]
                _a, _g = ident["ag"]
                if _a is not None:
                    t.setdefault("age_hist",
                                 deque(maxlen=AGE_VOTE_LEN)).append(_a)
                if _g:
                    t.setdefault("gender_hist",
                                 deque(maxlen=AGE_VOTE_LEN)).append(_g)
            t["zone_idx"] = ident["zone_idx"]
            t["misses"] = 0
            t["coast"] = 0          # detector confirmed it, lock is real
            t["coast_px"] = 0.0
            t["seen_time"] = time.time()
            used_tracks.add(matched)
        else:
            # Only spawn if this detection is not just a lagged copy of
            # an existing track sitting nearby.
            dt_, dr_, db_, dl_ = ident["box"]
            guard = max(SPAWN_MIN_DIST, (db_ - dt_) * SPAWN_FACE_FRAC)
            near = False
            for i in range(len(tracked_faces)):
                top, right, bottom, left = tracked_faces[i]["box"]
                tcx, tcy = (left + right) / 2, (top + bottom) / 2
                if ((tcx - cx) ** 2 + (tcy - cy) ** 2) ** 0.5 < guard:
                    near = True
                    break
            if near:
                continue
            top, right, bottom, left = ident["box"]
            top, left = max(0, top), max(0, left)
            right, bottom = min(w, right), min(h, bottom)
            if right <= left or bottom <= top:
                continue
            tmpl = gray_frame[top:bottom, left:right].copy()
            if tmpl.size == 0 or tmpl.std() < MIN_TEMPLATE_STD:
                continue
            tracked_faces.append({
                "box": (top, right, bottom, left),
                "template": tmpl,
                "hist": deque(
                    ([] if ident["name"] == PENDING else [ident["name"]]),
                    maxlen=NAME_VOTE_LEN),
                "confidence": ident["confidence"],
                "zone_idx": ident["zone_idx"],
                "misses": 0,
                "coast": 0,
                "seen_time": time.time(),
                "m_box": ident.get("m_box"),
                "m_zoom": ident.get("m_zoom", zoom_factor),
                "vx": 0.0,
                "vy": 0.0,
                "ag": ident.get("ag"),
                "id_time": time.time() if ident["name"] != PENDING else 0.0,
            })

    for i in range(n_before):
        if i not in used_tracks:
            tracked_faces[i]["misses"] = tracked_faces[i].get("misses", 0) + 1
    tracked_faces = [t for t in tracked_faces
                     if t.get("misses", 0) <= TRACK_MAX_MISSES]

def track_display_name(t):
    """Voted name, or None while the first identity is still resolving."""
    if not t["hist"]:
        return None
    return Counter(t["hist"]).most_common(1)[0][0]

# ========================= DRAW RESULTS ==============================
_last_draw_time = 0.0

def draw_results(frame):
    global last_reseed_time, tracked_faces, _last_draw_time

    gray_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    now_draw = time.time()
    dt = now_draw - _last_draw_time if _last_draw_time else 0.0
    _last_draw_time = now_draw

    with result_lock:
        age = time.time() - last_result_time
        current_result_time = last_result_time

    if age > IDENTITY_STALE_SEC:
        tracked_faces = []

    update_tracks(gray_frame, dt)

    if current_result_time != last_reseed_time:
        identities = get_identified_faces()
        # How old is this detection? Used to project it forward onto the
        # face's current position instead of snapping to a stale one.
        with result_lock:
            cap_t = result_meta[3] if len(result_meta) > 3 else 0.0
        lag = (time.time() - cap_t) if cap_t else 0.0
        reseed_tracks(gray_frame, identities, lag)
        last_reseed_time = current_result_time

    # Collapse duplicate boxes on one person, then drop anything the
    # detector has not confirmed lately. Runs every frame so a ghost is
    # gone in a frame, not after several slow detector passes.
    suppress_duplicate_tracks()
    expire_stale_tracks()

    _, _, using_far = get_active_encodings()

    for t in tracked_faces:
        # Track a tight face box, draw an expanded one framing the head.
        top, right, bottom, left = expand_box(t["box"])
        name = track_display_name(t)
        confidence, zone_idx = t["confidence"], t["zone_idx"]

        pending = name is None       # no identity resolved yet

        # Is the name still fresh? A track keeps its voted name after the
        # detector stops confirming it (when you turn away, say), which
        # is what makes the label persist. But the stored confidence is
        # then stale, so past NAME_FRESH_SEC we show "?" instead of a
        # number that is no longer true.
        held = (not pending and t.get("id_time", 0.0) > 0.0
                and (time.time() - t["id_time"]) > NAME_FRESH_SEC)

        if zone_idx >= 0:
            color = ZONE_COLORS[zone_idx]
        elif pending:
            color = (150, 150, 150)
        elif held:
            color = (140, 140, 140)     # known, but not freshly confirmed
        elif name == "Unknown":
            color = (60, 60, 255)
        elif confidence >= 70:
            color = (0, 220, 80)
        else:
            color = (0, 160, 255)

        cv2.rectangle(frame, (left, top), (right, bottom), color, 2)
        corner = 14
        for (x, y, dx, dy) in [(left, top, 1, 1), (right, top, -1, 1),
                               (left, bottom, 1, -1), (right, bottom, -1, -1)]:
            cv2.line(frame, (x, y), (x + dx * corner, y), color, 2)
            cv2.line(frame, (x, y), (x, y + dy * corner), color, 2)

        # Metrics from the tight (unexpanded) face box, EMA-smoothed.
        dist_str = ""
        if DIST_ENABLED:
            dcm = track_distance_cm(t)
            if dcm is not None:
                prev = t.get("dist_cm")
                t["dist_cm"] = dcm if prev is None else \
                    prev + (dcm - prev) * DIST_SMOOTH
                dist_str = fmt_distance(t["dist_cm"])
        if SPEED_ENABLED:
            update_speed(t, dt)
        if HEIGHT_ENABLED:
            update_height(t)

        if pending:
            if dist_str:
                draw_tag_stack(frame, left, top, bottom,
                               [(dist_str, 0.5)], (110, 110, 110))
            continue

        if name == "Unknown":
            label = "UNKNOWN"
        else:
            far_tag = " [FAR]" if using_far else ""
            zone_tag = f" [{ZONE_NAMES[zone_idx]}]" if zone_idx >= 0 else ""
            conf_tag = "?" if held else f"  {confidence}%"
            label = f"{name.upper()}{conf_tag}{far_tag}{zone_tag}"
        if dist_str:
            label += f"  {dist_str}"

        lines = [(label, 0.5)]
        # Second line: speed, height, age/gender. Toggle with I.
        if info_mode == "full":
            bits = [b for b in (fmt_speed(t), fmt_height(t), fmt_ag(t)) if b]
            if bits:
                lines.append(("  ".join(bits), 0.44))
        draw_tag_stack(frame, left, top, bottom, lines, color)

    return frame

def draw_tag_stack(frame, left, top, bottom, lines, color):
    """Draw label lines near a box, always fully on screen.

    Tries above the box, then below, then inside it. A big zoomed-in box
    can reach both edges of a small screen, so "above" and "below" both
    run off; drawing inside is the only place left. Also clamps x, since
    a box near the right edge would otherwise push text off the side.

    lines: list of (text, font_scale). Drawn top to bottom.
    """
    if not lines:
        return
    FONT = cv2.FONT_HERSHEY_SIMPLEX
    PAD, GAP = 6, 3
    sizes = []
    for text, scale in lines:
        (tw, th), _ = cv2.getTextSize(text, FONT, scale, 1)
        sizes.append((tw, th))
    block_h = sum(th + PAD for _, th in sizes) + GAP * (len(lines) - 1)
    widest = max(tw for tw, _ in sizes)

    if top - block_h - 4 >= 0:
        y = top - block_h - 4                 # above the box
        inside = False
    elif bottom + block_h + 4 <= DISPLAY_H:
        y = bottom + 4                        # below the box
        inside = False
    else:
        y = max(2, top + 4)                   # nowhere outside: go inside
        inside = True

    x = max(2, min(int(left), DISPLAY_W - widest - 10))

    for (text, scale), (tw, th) in zip(lines, sizes):
        h = th + PAD
        if y + h > DISPLAY_H:
            break                             # ran out of screen
        bg = (0, 0, 0) if inside else color
        fg = color if inside else (0, 0, 0)
        cv2.rectangle(frame, (x, y), (x + tw + 8, y + h), bg, -1)
        if inside:
            cv2.rectangle(frame, (x, y), (x + tw + 8, y + h), color, 1)
        cv2.putText(frame, text, (x + 4, y + th + 1), FONT, scale, fg, 1,
                    cv2.LINE_AA)
        y += h + GAP

# ========================== DRAW ZONES ===============================
def draw_zones(frame):
    if not zones and not (selecting and select_start and select_current):
        return frame
    overlay = frame.copy()
    cv2.rectangle(overlay, (0, 0), (DISPLAY_W, DISPLAY_H), (0, 0, 0), -1)
    cv2.addWeighted(overlay, 0.45, frame, 0.55, 0, frame)
    for i, zone in enumerate(zones):
        x1, y1, x2, y2 = zone
        color = ZONE_COLORS[i]
        zone_region = frame[y1:y2, x1:x2].copy()
        bright = cv2.convertScaleAbs(zone_region, alpha=1.4, beta=20)
        frame[y1:y2, x1:x2] = bright
        cv2.rectangle(frame, (x1, y1), (x2, y2), color, 2)
        corner = 16
        for (x, y, dx, dy) in [(x1, y1, 1, 1), (x2, y1, -1, 1),
                               (x1, y2, 1, -1), (x2, y2, -1, -1)]:
            cv2.line(frame, (x, y), (x + dx * corner, y), color, 3)
            cv2.line(frame, (x, y), (x, y + dy * corner), color, 3)
        label = f"{ZONE_NAMES[i]} - SCANNING"
        cv2.putText(frame, label, (x1 + 8, y1 + 20),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.5, color, 1, cv2.LINE_AA)
        pulse = int(abs(np.sin(time.time() * 3)) * 5) + 3
        cv2.circle(frame, (x1 + len(label) * 7 + 20, y1 + 16),
                   pulse, color, -1)
    if selecting and select_start and select_current:
        cv2.rectangle(frame, select_start, select_current, (0, 220, 255), 1)
    return frame

# ================== SELECTION UI (high-res freeze) ===================
LOUPE_SRC_HALF = 44             # display px sampled around the cursor
LOUPE_OUT = 200                 # rendered loupe size
PREVIEW_MAX_W = 300
PREVIEW_MAX_H = 210

def draw_crosshair(frame, pos):
    x, y = pos
    cv2.line(frame, (0, y), (DISPLAY_W, y), (90, 90, 90), 1, cv2.LINE_AA)
    cv2.line(frame, (x, 0), (x, DISPLAY_H), (90, 90, 90), 1, cv2.LINE_AA)

def draw_loupe(frame, pos):
    """Magnifier sampled from the full-res frozen MAIN frame, so the
    loupe shows true native detail, not upscaled display pixels."""
    if frozen_main is None:
        return
    x, y = pos
    mx1, my1 = display_to_main(x - LOUPE_SRC_HALF, y - LOUPE_SRC_HALF)
    mx2, my2 = display_to_main(x + LOUPE_SRC_HALF, y + LOUPE_SRC_HALF)
    mx1 = max(0, mx1); my1 = max(0, my1)
    mx2 = min(MAIN_W, mx2); my2 = min(MAIN_H, my2)
    if mx2 - mx1 < 8 or my2 - my1 < 8:
        return
    patch = frozen_main[my1:my2, mx1:mx2]
    loupe = cv2.resize(patch, (LOUPE_OUT, LOUPE_OUT),
                       interpolation=cv2.INTER_NEAREST)
    px = x + 28
    py = y + 28
    if px + LOUPE_OUT > DISPLAY_W:
        px = x - 28 - LOUPE_OUT
    if py + LOUPE_OUT > DISPLAY_H:
        py = y - 28 - LOUPE_OUT
    px = max(0, px); py = max(0, py)
    frame[py:py + LOUPE_OUT, px:px + LOUPE_OUT] = loupe
    cv2.rectangle(frame, (px, py), (px + LOUPE_OUT, py + LOUPE_OUT),
                  (255, 255, 255), 1)
    c = LOUPE_OUT // 2
    cv2.line(frame, (px + c - 8, py + c), (px + c + 8, py + c),
             (0, 220, 255), 1)
    cv2.line(frame, (px + c, py + c - 8), (px + c, py + c + 8),
             (0, 220, 255), 1)

def draw_drag_dimensions(frame):
    if not (selecting and select_start and select_current):
        return
    x1, y1, x2, y2 = normalize_zone((*select_start, *select_current))
    wd, hd = x2 - x1, y2 - y1
    wm = int(wd * MAIN_W / DISPLAY_W)
    hm = int(hd * MAIN_H / DISPLAY_H)
    label = f"{wd}x{hd}  ({wm}x{hm} px src)"
    tx = min(select_current[0] + 12, DISPLAY_W - 220)
    ty = max(select_current[1] - 10, 18)
    cv2.putText(frame, label, (tx, ty),
                cv2.FONT_HERSHEY_SIMPLEX, 0.45, (0, 220, 255), 1, cv2.LINE_AA)

def draw_zone_preview(frame):
    """Native-resolution preview of the newest zone, cropped from the
    frozen MAIN frame."""
    if not zones or frozen_main is None:
        return
    idx = len(zones) - 1
    crop, _, _ = crop_zone_from_main(frozen_main, zones[idx])
    if crop is None or crop.size == 0:
        return
    ch, cw = crop.shape[:2]
    scale = min(PREVIEW_MAX_W / cw, PREVIEW_MAX_H / ch, 1.0)
    pw, ph = max(1, int(cw * scale)), max(1, int(ch * scale))
    preview = cv2.resize(crop, (pw, ph), interpolation=cv2.INTER_AREA)
    x0 = DISPLAY_W - pw - 12
    y0 = 96
    frame[y0:y0 + ph, x0:x0 + pw] = preview
    color = ZONE_COLORS[idx]
    cv2.rectangle(frame, (x0, y0), (x0 + pw, y0 + ph), color, 2)
    caption = f"{ZONE_NAMES[idx]} PREVIEW  {cw}x{ch}px"
    cv2.putText(frame, caption, (x0, y0 - 8),
                cv2.FONT_HERSHEY_SIMPLEX, 0.42, color, 1, cv2.LINE_AA)

def draw_selection_header(frame):
    cv2.putText(frame, "ZONE SELECTION MODE", (DISPLAY_W // 2 - 130, 30),
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 220, 255), 2, cv2.LINE_AA)
    cv2.putText(frame,
                f"Drag to draw a zone   ({len(zones)}/{MAX_ZONES} set)   "
                f"Right-click deletes a zone",
                (DISPLAY_W // 2 - 250, 55),
                cv2.FONT_HERSHEY_SIMPLEX, 0.45, (200, 200, 200), 1,
                cv2.LINE_AA)
    cv2.putText(frame, "S or ESC confirms   C clears all",
                (DISPLAY_W // 2 - 130, 75),
                cv2.FONT_HERSHEY_SIMPLEX, 0.45, (200, 200, 200), 1,
                cv2.LINE_AA)

def render_selection_frame():
    canvas = frozen_display.copy()
    if zones or (selecting and select_start):
        canvas = draw_zones(canvas)
    else:
        overlay = canvas.copy()
        cv2.rectangle(overlay, (0, 0), (DISPLAY_W, DISPLAY_H), (0, 0, 0), -1)
        cv2.addWeighted(overlay, 0.25, canvas, 0.75, 0, canvas)
    draw_crosshair(canvas, mouse_pos)
    draw_drag_dimensions(canvas)
    draw_zone_preview(canvas)
    draw_loupe(canvas, mouse_pos)
    draw_selection_header(canvas)
    return canvas

# ============================= HUD ===================================
def draw_hud(frame, current_fps, zoom, auto_zoom):
    h, w = frame.shape[:2]
    overlay = frame.copy()
    cv2.rectangle(overlay, (0, 0), (w, 50), (0, 0, 0), -1)
    cv2.addWeighted(overlay, 0.55, frame, 0.45, 0, frame)

    fps_color = (0, 220, 80) if current_fps >= 8 else \
                (0, 160, 255) if current_fps >= 4 else (60, 60, 255)
    cv2.putText(frame, f"FPS {current_fps:.0f}", (w - 90, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, fps_color, 2, cv2.LINE_AA)
    cv2.putText(frame, f"ZOOM  {zoom:.1f}x", (10, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 220, 80), 2, cv2.LINE_AA)

    az_color = (0, 220, 80) if auto_zoom else (100, 100, 100)
    cv2.putText(frame, "AUTO" if auto_zoom else "MANUAL", (170, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.55, az_color, 1, cv2.LINE_AA)

    sensor_color = (0, 160, 255) if current_sensor == "FULL" else (0, 220, 80)
    cv2.putText(frame, f"SNSR:{current_sensor} [{sensor_pref}]", (260, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.48, sensor_color, 1, cv2.LINE_AA)

    _, _, using_far = get_active_encodings()
    enc_label = "ENC:FAR" if using_far else "ENC:CLOSE"
    enc_color = (0, 160, 255) if using_far else (0, 220, 80)
    cv2.putText(frame, enc_label + f" [{encoding_mode.upper()}]", (480, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.48, enc_color, 1, cv2.LINE_AA)

    zone_status = f"ZONES {len(zones)}/{MAX_ZONES}" if zones else "FULL FRAME"
    zone_color = (0, 220, 255) if zones else (120, 120, 120)
    cv2.putText(frame, zone_status, (680, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.5, zone_color, 1, cv2.LINE_AA)

    with result_lock:
        known = sum(1 for n in face_names if n != "Unknown")
        unknown = sum(1 for n in face_names if n == "Unknown")
    cv2.putText(frame, f"K:{known} U:{unknown}", (810, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.5, (180, 180, 180), 1,
                cv2.LINE_AA)

    if DIST_ENABLED:
        dist_lbl = "DIST:CAL" if DIST_CALIBRATED else "DIST:~"
        dist_color = (0, 220, 80) if DIST_CALIBRATED else (0, 160, 255)
        cv2.putText(frame, dist_lbl, (910, 32),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.48, dist_color, 1,
                    cv2.LINE_AA)

    # Detection lag. This is how far behind the detector runs, and it is
    # what lag compensation projects away. Big numbers here mean the
    # recognition pass is slow, which is the root of most box problems.
    if info_mode == "full":
        with result_lock:
            cap_t = result_meta[3] if len(result_meta) > 3 else 0.0
        if cap_t:
            lag_ms = (time.time() - cap_t) * 1000.0
            lag_color = (0, 220, 80) if lag_ms < 400 else \
                        (0, 160, 255) if lag_ms < 900 else (60, 60, 255)
            cv2.putText(frame, f"LAG {lag_ms:.0f}ms", (10, 68),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.45, lag_color, 1,
                        cv2.LINE_AA)

    # Optical axis. Everything on this line is at exactly lens height,
    # which is what CAM_HEIGHT_CM must equal. Handy two ways: measure
    # whatever the line lands on to get camera height, or check the line
    # stays on a level surface to confirm the camera is not tilted.
    if info_mode == "full" and HEIGHT_ENABLED:
        cy = int(optical_axis_y())
        if 0 <= cy < h:
            for x in range(0, w, 24):
                cv2.line(frame, (x, cy), (x + 12, cy), (120, 120, 120), 1)
            cv2.putText(frame, f"lens height {CAM_HEIGHT_CM:.0f}cm",
                        (10, max(12, cy - 8)), cv2.FONT_HERSHEY_SIMPLEX,
                        0.42, (120, 120, 120), 1, cv2.LINE_AA)
        else:
            # Panned so far the axis is off-screen. Height still computes,
            # but say so rather than draw a line that is not there.
            cv2.putText(frame, "lens axis off-screen (panned)", (10, 88),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.42, (0, 160, 255), 1,
                        cv2.LINE_AA)

    # Pan mini-map: the outer box is the whole sensor, the inner box is
    # what you are looking at. Without this, panning around a scene you
    # cannot fully see is disorienting.
    if zoom_factor > 1.05:
        mw, mh = 92, 52
        mx, my = w - mw - 12, 58
        cv2.rectangle(frame, (mx, my), (mx + mw, my + mh), (90, 90, 90), 1)
        vw = max(4, int(mw / max(zoom_factor, 1e-3)))
        vh = max(4, int(mh / max(zoom_factor, 1e-3)))
        vx = int(mx + last_applied_pan_x * mw - vw / 2)
        vy = int(my + last_applied_pan_y * mh - vh / 2)
        vx = max(mx, min(vx, mx + mw - vw))
        vy = max(my, min(vy, my + mh - vh))
        vcol = (0, 220, 80) if not auto_zoom_enabled else (0, 160, 255)
        cv2.rectangle(frame, (vx, vy), (vx + vw, vy + vh), vcol, 1)
        cv2.putText(frame, "view", (mx, my - 4),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.35, (90, 90, 90), 1,
                    cv2.LINE_AA)

    overlay2 = frame.copy()
    cv2.rectangle(overlay2, (0, h - 36), (w, h), (0, 0, 0), -1)
    cv2.addWeighted(overlay2, 0.55, frame, 0.45, 0, frame)
    cv2.putText(frame,
                "=  zoom in    -  zoom out    A  auto zoom    "
                "H  sensor    E  enc    D  calib dist    K  calib height    "
                "I  info    R  reload    L  shot    S  zones    C  clear    Q  quit",
                (10, h - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.36,
                (150, 150, 150), 1, cv2.LINE_AA)
    return frame

# ========================= DISTANCE =================================
def _load_dist_c():
    """Loads the distance constant and, if present, a previously solved
    camera height. Old files with only C still load fine."""
    global CAM_HEIGHT_CM
    if os.path.exists(DIST_CALIB_FILE):
        try:
            with open(DIST_CALIB_FILE) as f:
                data = json.load(f)
            ver = int(data.get("version", 1))
            if ver != DIST_CALIB_VERSION:
                print(f"[DIST] {DIST_CALIB_FILE} is v{ver}, this build needs "
                      f"v{DIST_CALIB_VERSION} (the units changed). Ignoring "
                      "it. Please re-calibrate: stand at "
                      f"{DIST_CALIB_CM:.0f} cm and press D, then press K.")
                return DIST_C_DEFAULT, False
            c = float(data["C"])
            if "CAM_H" in data:
                CAM_HEIGHT_CM = float(data["CAM_H"])
                print(f"[DIST] loaded camera height {CAM_HEIGHT_CM:.1f} cm")
            print(f"[DIST] loaded calibration C={c:.1f} from {DIST_CALIB_FILE}")
            return c, True
        except Exception as e:
            print(f"[DIST] calibration load failed ({e}), using default")
    return DIST_C_DEFAULT, False

def _save_calib():
    try:
        with open(DIST_CALIB_FILE, "w") as f:
            json.dump({"version": DIST_CALIB_VERSION, "C": DIST_C,
                       "CAM_H": CAM_HEIGHT_CM}, f)
        return True
    except Exception as e:
        print(f"[DIST] save failed: {e}")
        return False

DIST_C, DIST_CALIBRATED = _load_dist_c()

def face_size_metric(box):
    """Zoom-independent apparent face size, in DISPLAY pixels.

    This is the ORIGINAL, accurate version. It measures the tracked box
    directly and DIST_C is calibrated in the same display-pixel units,
    so the units cancel and it is self-consistent. An earlier "fix"
    converted this to main pixels for distance but left the tracked box
    in display pixels, which introduced a mismatch and made every
    reading too short. Reverted.

    sqrt(width*height) so a head tilt (mostly height) or a turn (mostly
    width) moves the estimate less than either dimension alone.
    Normalized by the digital zoom so 2x zoom does not read as closer.
    """
    top, right, bottom, left = box
    w = max(1, right - left)
    h = max(1, bottom - top)
    return (w * h) ** 0.5 / max(zoom_factor, 1e-3)

def track_distance_cm(t):
    m = face_size_metric(t["box"])
    return DIST_C / m if m > 0 else None

def fmt_distance(cm):
    if cm is None:
        return ""
    if DIST_UNITS == "ft":
        val = cm / 30.48
        unit = "ft"
    else:
        val = cm / 100.0
        unit = "m"
    tilde = "" if DIST_CALIBRATED else "~"
    return f"{tilde}{val:.1f}{unit}"

def focal_px():
    """Effective focal length in display px at zoom 1, derived from the
    distance constant: DIST_C = focal * real_face_size."""
    return DIST_C / FACE_REAL_CM

def update_speed(t, dt):
    """Real-world speed in ft/s, combining lateral motion (across the
    frame) with radial motion (toward or away from the camera).

    Lateral uses the face as an on-screen ruler: a face spans about
    FACE_REAL_CM, so cm-per-pixel = FACE_REAL_CM / face_px. Zoom
    cancels because both are measured on the same image, which means
    this stays correct while the camera is zooming.
    Radial is simply the rate of change of the distance estimate.

    The tracker stores velocity in px/sec already, so no dt division
    here. Accuracy depends on the box holding lock, which is why the
    tracker coasts through failed matches instead of freezing."""
    if dt <= 0:
        return
    top, right, bottom, left = t["box"]
    face_px = ((right - left) * (bottom - top)) ** 0.5
    if face_px <= 1:
        return
    cm_per_px = FACE_REAL_CM / face_px

    # Lateral: tracker velocity is already px/sec.
    lat_cm_s = ((t.get("vx", 0.0) ** 2 + t.get("vy", 0.0) ** 2) ** 0.5) \
        * cm_per_px

    # Radial: change in estimated distance over time.
    rad_cm_s = 0.0
    d_now = t.get("dist_cm")
    d_prev = t.get("_d_prev")
    if d_now is not None and d_prev is not None:
        rad_cm_s = abs(d_now - d_prev) / dt
    if d_now is not None:
        t["_d_prev"] = d_now

    total_cm_s = (lat_cm_s ** 2 + rad_cm_s ** 2) ** 0.5
    ft_s = total_cm_s / 30.48
    prev = t.get("speed")
    t["speed"] = ft_s if prev is None else prev + (ft_s - prev) * SPEED_SMOOTH

def fmt_speed(t):
    s = t.get("speed")
    if s is None:
        return ""
    if s < SPEED_MIN_SHOW:
        return "still"
    return f"{s:.1f}ft/s"

def optical_axis_frac():
    """Optical axis position as a fraction of frame height (0..1).

    The axis is the SENSOR CENTRE, not the image centre. They coincide
    only when the crop sits at the middle of the sensor. Auto-zoom pans
    the crop, which slides the axis away from the middle of the frame,
    and the offset grows with zoom.

    This mirrors apply_zoom()'s math exactly, including the clamp at the
    sensor edges, then applies the vertical flip from the camera
    Transform. Getting any of those three wrong throws height off by a
    foot or more once zoomed.
    """
    try:
        fw, fh = picam2.camera_properties["PixelArraySize"]
    except Exception:
        return 0.5
    z = max(zoom_factor, 1e-3)
    ch = int(fh / z)
    cy = int(last_applied_pan_y * fh - ch / 2)
    cy = max(0, min(cy, fh - ch))        # same clamp apply_zoom uses
    if ch <= 0:
        return 0.5
    v = (fh / 2.0 - cy) / float(ch)      # sensor centre within the crop
    if VFLIP:
        v = 1.0 - v                      # the image is flipped vertically
    return v

def optical_axis_y():
    """Optical axis in DISPLAY pixels (for drawing the overlay line)."""
    return DISPLAY_H * optical_axis_frac()

def optical_axis_y_main():
    """Optical axis in MAIN pixels (for the height maths)."""
    return MAIN_H * optical_axis_frac()

def update_height(t):
    """Standing height from the pinhole projection of the head top.

    For a LEVEL camera at CAM_HEIGHT_CM, a point at distance d whose
    real height above the floor is Y projects dy px from the OPTICAL
    AXIS, where dy = zoom * focal * (Y - CAM_HEIGHT_CM) / d.
    Solve for Y. Assumes the camera is not tilted. A tilted camera
    makes this wrong, and the reading is only as good as CAM_HEIGHT_CM.
    """
    d = t.get("dist_cm")
    if d is None:
        return
    f = focal_px()
    if f <= 0:
        return
    # Work entirely in MAIN pixels from the raw detector box, matching
    # the units of focal_px(). Going via the display box would reintroduce
    # the aspect/scaling factor that made distance unreliable.
    m_box = t.get("m_box")
    if m_box is None:
        return
    mt, mr, mb, ml = m_box
    z = max(t.get("m_zoom", zoom_factor), 1e-3)
    # Head top is above the detector box (which starts at the eyebrows).
    # Not clamped: clamping would pretend a head near the top edge sits
    # at y=0 and silently under-read height.
    head_top = mt - (mb - mt) * BOX_EXPAND_TOP
    # If the detected face is jammed against the top edge the head really
    # is cut off, so its position is unknown. Say nothing rather than
    # guess low.
    if mt <= 1:
        return
    dy = optical_axis_y_main() - head_top     # +ve when above the axis
    y_cm = CAM_HEIGHT_CM + dy * d / (z * f)
    if not (30.0 < y_cm < 260.0):         # reject nonsense
        return
    prev = t.get("height_cm")
    t["height_cm"] = y_cm if prev is None else \
        prev + (y_cm - prev) * HEIGHT_SMOOTH

def fmt_height(t):
    cm = t.get("height_cm")
    if cm is None:
        return ""
    if DIST_UNITS == "ft":
        inches = cm / 2.54
        ft = int(inches // 12)
        inch = int(round(inches - ft * 12))
        if inch == 12:
            ft += 1
            inch = 0
        return f"{ft}'{inch}\""
    return f"{cm / 100.0:.2f}m"

def fmt_ag(t):
    """Median age and majority gender over the last AGE_VOTE_LEN reads.
    A single genderage inference jitters by several years, so voting
    gives a number that holds still enough to be worth showing."""
    ah = t.get("age_hist")
    gh = t.get("gender_hist")
    if not ah:
        ag = t.get("ag")
        if not ag or ag[0] is None:
            return ""
        return f"{ag[1] or ''}{ag[0]}"
    vals = sorted(ah)
    age = vals[len(vals) // 2]
    gender = Counter(gh).most_common(1)[0][0] if gh else ""
    return f"{gender}{age}"

def calibrate_cam_height():
    """Solve CAM_HEIGHT_CM from a person of known height (MY_HEIGHT_CM).

    The height formula is:
        Y = CAM_HEIGHT_CM + dy * d / (zoom * focal)
    Everything except CAM_HEIGHT_CM is measured, and Y is known, so:
        CAM_HEIGHT_CM = Y - dy * d / (zoom * focal)

    Requires the distance calibration (D) first, since focal comes from
    it. Stand upright on the same floor the camera sits on, facing the
    camera, then press K."""
    global CAM_HEIGHT_CM
    if not DIST_CALIBRATED:
        print("[HEIGHT] calibrate distance first: stand at "
              f"{DIST_CALIB_CM:.0f} cm and press D")
        return
    if not tracked_faces:
        print("[HEIGHT] calibrate: no face on screen")
        return
    t = max(tracked_faces, key=lambda x: (x["box"][2] - x["box"][0]) *
                                          (x["box"][1] - x["box"][3]))
    d = t.get("dist_cm")
    if d is None:
        print("[HEIGHT] calibrate: no distance reading yet")
        return
    f = focal_px()
    if f <= 0:
        print("[HEIGHT] calibrate: bad focal length")
        return
    # Raw main-frame box and its zoom, matching update_height() exactly.
    m_box = t.get("m_box")
    if m_box is None:
        print("[HEIGHT] calibrate: no detection for that face yet")
        return
    mt, mr, mb, ml = m_box
    if mt <= 1:
        print("[HEIGHT] calibrate: your head is cut off at the top of the "
              "frame. Step back or tilt down, then press K again.")
        return
    z = max(t.get("m_zoom", zoom_factor), 1e-3)
    head_top = mt - (mb - mt) * BOX_EXPAND_TOP
    dy = optical_axis_y_main() - head_top
    CAM_HEIGHT_CM = MY_HEIGHT_CM - dy * d / (z * f)
    # Clear cached heights so every track re-reads with the new value.
    for tr in tracked_faces:
        tr.pop("height_cm", None)
    ok = _save_calib()
    print(f"[HEIGHT] camera height solved: {CAM_HEIGHT_CM:.1f} cm "
          f"({CAM_HEIGHT_CM / 2.54:.1f} in) from a "
          f"{MY_HEIGHT_CM:.0f} cm person at {d:.0f} cm"
          + ("" if ok else " (not saved)"))
    if not (10.0 < CAM_HEIGHT_CM < 250.0):
        print("[HEIGHT] that value looks wrong. Check MY_HEIGHT_CM, and "
              "make sure the distance calibration is good.")

def calibrate_distance():
    """Set DIST_C from the largest tracked face, assuming it is standing
    at DIST_CALIB_CM. Saves so it persists across runs.

    Uses the RAW detector box, not the tracked one. The tracked box eases
    toward the detected size, so calibrating against it a moment too
    early baked a 20%+ error into DIST_C and every distance inherited it.
    """
    global DIST_C, DIST_CALIBRATED
    if not tracked_faces:
        print("[DIST] calibrate: no face on screen")
        return
    t = max(tracked_faces, key=lambda x: (x["box"][2] - x["box"][0]) *
                                          (x["box"][1] - x["box"][3]))
    # Guard only: refuse to calibrate against a face the detector has not
    # confirmed recently, so you cannot calibrate against where you were
    # a moment ago. The measurement itself uses the same display box the
    # readout uses, which is what makes it self-consistent.
    age = time.time() - t.get("seen_time", 0.0)
    if age > 1.5:
        print(f"[DIST] calibrate: last detection was {age:.1f}s ago. "
              "Face the camera, hold still, press D again")
        return
    m = face_size_metric(t["box"])
    if m <= 0:
        print("[DIST] calibrate: bad face size")
        return
    DIST_C = DIST_CALIB_CM * m
    DIST_CALIBRATED = True
    ok = _save_calib()
    print(f"[DIST] calibrated at {DIST_CALIB_CM:.0f} cm, C={DIST_C:.0f}"
          + (f", saved to {DIST_CALIB_FILE}" if ok else " (not saved)"))
    print(f"[DIST] next: set MY_HEIGHT_CM ({MY_HEIGHT_CM:.0f} cm now), "
          "stand upright in view, press K to solve camera height")

def shift_tracks_and_cache(dx_view, dy_view):
    """Slide existing tracks and cached identities to follow a pan.

    Panning does not make the scene unknown, it just moves it by a known
    amount, so throwing tracks and embeddings away (the old behaviour)
    was needless. It also made panning useless in practice: holding an
    arrow key repeats, which cleared the identity cache every frame, so
    recognition could never finish and you stayed unidentified.

    If the view moves right, content moves LEFT on screen by the same
    amount, hence the sign flip.
    """
    dx_disp = -dx_view * DISPLAY_W
    dy_disp = -dy_view * DISPLAY_H
    for t in tracked_faces:
        top, right, bottom, left = t["box"]
        t["box"] = (int(top + dy_disp), int(right + dx_disp),
                    int(bottom + dy_disp), int(left + dx_disp))
        # The template is still valid: it is the same face, just moved.
    dx_main = -dx_view * MAIN_W
    dy_main = -dy_view * MAIN_H
    with cache_lock:
        for e in identity_cache:
            e["cx"] += dx_main
            e["cy"] += dy_main

def manual_pan(dx_view, dy_view):
    """Slide the crop around inside the sensor. dx/dy are in units of the
    current VIEW (1.0 = one full screen width), so a press moves the
    picture the same amount on screen at any zoom.

    Sign convention is taken from the auto-pan code, which is known to
    work: there, a face at fx > 0.5 (right of centre) raises pan_x, so
    increasing pan_x moves the view RIGHT. Same for pan_y and down.
    """
    global pan_current_x, pan_current_y, auto_zoom_enabled
    global last_applied_pan_x, last_applied_pan_y
    global auto_zoom_current

    if auto_zoom_enabled:
        # Auto-pan would immediately steer back to a face and fight the
        # keys, so hand control over.
        auto_zoom_enabled = False
        auto_zoom_current = zoom_factor
        print("Auto zoom: OFF (manual pan)")

    if PAN_INVERT_X:
        dx_view = -dx_view
    if PAN_INVERT_Y:
        dy_view = -dy_view

    # A view is 1/zoom of the sensor, so a step of dx_view screens is
    # dx_view/zoom in sensor fractions.
    z = max(zoom_factor, 1e-3)
    before_x, before_y = pan_current_x, pan_current_y
    pan_current_x = pan_clamp(pan_current_x + dx_view / z)
    pan_current_y = pan_clamp(pan_current_y + dy_view / z)

    # Only shift by the pan that actually happened. At a sensor edge the
    # clamp eats some or all of it, and shifting by the requested amount
    # instead of the real one would slide the boxes off the faces.
    actual_dx = (pan_current_x - before_x) * z
    actual_dy = (pan_current_y - before_y) * z
    if actual_dx == 0.0 and actual_dy == 0.0:
        return                      # at the edge, nothing moved

    apply_zoom(zoom_factor, pan_current_x, pan_current_y)
    last_applied_pan_x = pan_current_x
    last_applied_pan_y = pan_current_y
    shift_tracks_and_cache(actual_dx, actual_dy)

def center_pan():
    """Snap the view back to the middle of the sensor."""
    global pan_current_x, pan_current_y
    global last_applied_pan_x, last_applied_pan_y
    z = max(zoom_factor, 1e-3)
    dx = (0.5 - pan_current_x) * z
    dy = (0.5 - pan_current_y) * z
    pan_current_x = 0.5
    pan_current_y = 0.5
    apply_zoom(zoom_factor, 0.5, 0.5)
    last_applied_pan_x = 0.5
    last_applied_pan_y = 0.5
    shift_tracks_and_cache(dx, dy)
    print("View re-centred")

def pan_limits():
    """How much room is left to pan, as a fraction of the sensor.
    Zero at 1x zoom, because the crop already covers everything."""
    z = max(zoom_factor, 1e-3)
    if z <= 1.0:
        return 0.0
    return (1.0 - 1.0 / z) / 2.0

def pan_clamp(v):
    """Clamp a pan value to what the crop can actually honour.

    apply_zoom() clamps the CROP at the sensor edges, so pan values
    beyond that range have no effect on screen. Storing them anyway
    causes windup: hold right at the edge and pan_x climbs to 1.0, then
    the first several left presses do nothing while it unwinds. Clamping
    to the legal range keeps the stored value and the picture in step.
    """
    lim = pan_limits()
    return max(0.5 - lim, min(0.5 + lim, v))

def scale_cache_for_zoom(z_old, z_new):
    """Rescale cached identity positions when the zoom changes.

    Zooming magnifies the view about its centre, so a face at main
    coords (cx, cy) moves outward from the centre by the zoom ratio.
    Rescaling the cache instead of clearing it means your name survives
    a zoom press, rather than needing a fresh 1-2s embedding every time
    you touch the zoom.
    """
    if z_old <= 0 or z_new <= 0:
        return
    r = z_new / z_old
    ccx, ccy = MAIN_W / 2.0, MAIN_H / 2.0
    with cache_lock:
        for e in identity_cache:
            e["cx"] = ccx + (e["cx"] - ccx) * r
            e["cy"] = ccy + (e["cy"] - ccy) * r
            e["h"] = e["h"] * r

def zoom_to(new_zoom):
    """Manual zoom that keeps looking where you are already looking.

    The old code slammed pan back to centre on every zoom press, so
    lining someone up with the pan keys and then zooming threw the aim
    away and re-centred on the middle of the sensor. Zoom should be
    about the current view, not the sensor centre.
    """
    global zoom_factor, auto_zoom_enabled, auto_zoom_current
    global pan_current_x, pan_current_y
    global last_applied_zoom, last_applied_pan_x, last_applied_pan_y
    global pan_target_x_abs, pan_target_y_abs, tracked_faces

    z_old = zoom_factor
    auto_zoom_enabled = False
    zoom_factor = new_zoom
    auto_zoom_current = zoom_factor

    # Keep the current aim. Re-clamp because zooming OUT shrinks the
    # legal pan range and the old value may now sit outside it.
    pan_current_x = pan_clamp(pan_current_x)
    pan_current_y = pan_clamp(pan_current_y)
    pan_target_x_abs = pan_current_x
    pan_target_y_abs = pan_current_y
    last_applied_zoom = zoom_factor
    last_applied_pan_x = pan_current_x
    last_applied_pan_y = pan_current_y

    update_zoom_full(zoom_factor, pan_current_x, pan_current_y)

    # Faces change size, so the templates are the wrong scale and the
    # tracks must rebuild. Identities are kept: rescaling the cache lets
    # the new tracks pick their names straight back up.
    scale_cache_for_zoom(z_old, zoom_factor)
    tracked_faces = []
    maybe_switch_sensor()

# ==================== PORTABLE / BROWSER STREAM ======================
if HEADLESS is None:
    HEADLESS = not os.environ.get("DISPLAY")

_jpeg_lock = Lock()
_latest_jpeg = None
web_keys = Queue()
web_zone_q = Queue()

def publish_frame(frame):
    """Encode the composited frame for the browser. No rate limit for
    this first full-quality test; STREAM_MAX_FPS caps the reader side."""
    global _latest_jpeg
    if not STREAM_ENABLED:
        return
    ok, buf = cv2.imencode(".jpg", frame,
                           [int(cv2.IMWRITE_JPEG_QUALITY), STREAM_QUALITY])
    if ok:
        with _jpeg_lock:
            _latest_jpeg = buf.tobytes()

PAGE = """<!DOCTYPE html><html><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Face Recognition</title><style>
body{background:#111;color:#ddd;font-family:system-ui,sans-serif;margin:0;padding:10px}
#wrap{max-width:1024px;margin:0 auto}
#vid{position:relative;display:inline-block;width:100%;touch-action:none}
img{width:100%;display:block;border:1px solid #333;border-radius:6px}
#sel{position:absolute;border:2px solid #0dd;background:rgba(0,220,220,.15);display:none;pointer-events:none}
.row{margin:8px 0}
button{background:#222;color:#ddd;border:1px solid #444;border-radius:6px;
padding:11px 13px;margin:3px;font-size:15px;cursor:pointer;min-width:48px}
button:active{background:#0a84ff;border-color:#0a84ff}
.lbl{color:#888;font-size:12px;margin-right:6px}
.pan{display:grid;grid-template-columns:repeat(3,52px);gap:4px;width:max-content}
</style></head><body><div id="wrap">
<div id="vid"><img id="f" src="/stream"><div id="sel"></div></div>
<div class="row"><span class="lbl">look around</span></div>
<div class="pan">
<span></span><button onclick="k('t')">&uarr;</button><span></span>
<button onclick="k('f')">&larr;</button><button onclick="k('v')">&#9679;</button><button onclick="k('g')">&rarr;</button>
<span></span><button onclick="k('b')">&darr;</button><span></span></div>
<div class="row"><span class="lbl">zoom</span>
<button onclick="k('=')">+</button><button onclick="k('-')">&minus;</button>
<button onclick="k('a')">auto</button><button onclick="k('h')">sensor</button></div>
<div class="row"><span class="lbl">calib</span>
<button onclick="k('d')">dist</button><button onclick="k('k')">height</button>
<button onclick="k('e')">enc</button><button onclick="k('i')">info</button></div>
<div class="row"><span class="lbl">tools</span>
<button onclick="k('r')">reload</button><button onclick="k('l')">shot</button>
<button onclick="k('c')">clear zones</button></div>
<div class="row"><span class="lbl">drag on the video to draw a scan zone</span></div>
</div><script>
function k(x){fetch('/key?k='+encodeURIComponent(x));}
var img=document.getElementById('f'),sel=document.getElementById('sel'),
    box=document.getElementById('vid'),sx=0,sy=0,drag=false;
function pos(e){var r=img.getBoundingClientRect();
  var t=e.touches?e.touches[0]:e;
  return [(t.clientX-r.left)/r.width,(t.clientY-r.top)/r.height];}
function down(e){var p=pos(e);sx=p[0];sy=p[1];drag=true;
  sel.style.display='block';sel.style.left=(sx*100)+'%';sel.style.top=(sy*100)+'%';
  sel.style.width='0';sel.style.height='0';e.preventDefault();}
function move(e){if(!drag)return;var p=pos(e);
  sel.style.left=(Math.min(sx,p[0])*100)+'%';sel.style.top=(Math.min(sy,p[1])*100)+'%';
  sel.style.width=(Math.abs(p[0]-sx)*100)+'%';sel.style.height=(Math.abs(p[1]-sy)*100)+'%';
  e.preventDefault();}
function up(e){if(!drag)return;drag=false;sel.style.display='none';
  var p=pos(e.changedTouches?{touches:e.changedTouches}:e);
  fetch('/zone?x1='+sx+'&y1='+sy+'&x2='+p[0]+'&y2='+p[1]);}
box.addEventListener('mousedown',down);window.addEventListener('mousemove',move);
window.addEventListener('mouseup',up);
box.addEventListener('touchstart',down);box.addEventListener('touchmove',move);
box.addEventListener('touchend',up);
document.addEventListener('keydown',function(e){
  var m={ArrowUp:'t',ArrowLeft:'f',ArrowRight:'g',ArrowDown:'b'};
  if(m[e.key]){k(m[e.key]);e.preventDefault();}
  else if('=-avhdkeirlc'.includes(e.key)) k(e.key);});
</script></body></html>"""

class StreamHandler(BaseHTTPRequestHandler):
    def log_message(self, *a):
        pass

    def do_GET(self):
        u = urlparse(self.path)
        if u.path == "/":
            body = PAGE.encode()
            self.send_response(200)
            self.send_header("Content-Type", "text/html")
            self.send_header("Content-Length", str(len(body)))
            self.end_headers()
            self.wfile.write(body)
        elif u.path == "/key":
            q = parse_qs(u.query).get("k", [""])[0]
            if q:
                web_keys.put(q[0])
            self.send_response(204)
            self.end_headers()
        elif u.path == "/zone":
            q = parse_qs(u.query)
            try:
                x1 = float(q["x1"][0]); y1 = float(q["y1"][0])
                x2 = float(q["x2"][0]); y2 = float(q["y2"][0])
                web_zone_q.put((
                    int(min(x1, x2) * DISPLAY_W), int(min(y1, y2) * DISPLAY_H),
                    int(max(x1, x2) * DISPLAY_W), int(max(y1, y2) * DISPLAY_H)))
            except Exception:
                pass
            self.send_response(204)
            self.end_headers()
        elif u.path == "/stream":
            self.send_response(200)
            self.send_header("Age", "0")
            self.send_header("Cache-Control", "no-cache, private")
            self.send_header("Content-Type",
                             "multipart/x-mixed-replace; boundary=FRAME")
            self.end_headers()
            try:
                while True:
                    with _jpeg_lock:
                        buf = _latest_jpeg
                    if buf is None:
                        time.sleep(0.05)
                        continue
                    self.wfile.write(b"--FRAME\r\n")
                    self.send_header("Content-Type", "image/jpeg")
                    self.send_header("Content-Length", str(len(buf)))
                    self.end_headers()
                    self.wfile.write(buf)
                    self.wfile.write(b"\r\n")
                    time.sleep(1.0 / max(1, STREAM_MAX_FPS))
            except Exception:
                pass
        else:
            self.send_response(404)
            self.end_headers()

def _local_ip():
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    try:
        s.connect(("8.8.8.8", 80))
        return s.getsockname()[0]
    except Exception:
        return "127.0.0.1"
    finally:
        s.close()

def start_stream_server():
    if not STREAM_ENABLED:
        return
    try:
        srv = ThreadingHTTPServer(("0.0.0.0", STREAM_PORT), StreamHandler)
        srv.daemon_threads = True
        Thread(target=srv.serve_forever, daemon=True).start()
        print(f"[STREAM] open  http://{_local_ip()}:{STREAM_PORT}  "
              "on any device on this network")
    except Exception as e:
        print(f"[STREAM] could not start on port {STREAM_PORT}: {e}")

def get_key_web():
    """A key pressed in the browser, or None."""
    try:
        return web_keys.get_nowait()
    except Empty:
        return None

def drain_web_zones():
    added = False
    while True:
        try:
            z = web_zone_q.get_nowait()
        except Empty:
            break
        if z[2] - z[0] > 20 and z[3] - z[1] > 20 and len(zones) < MAX_ZONES:
            zones.append(z)
            added = True
            print(f"Zone {len(zones)} set from browser")
    return added

def show(frame):
    """Send the composited frame to the window, the browser, or both."""
    if not HEADLESS:
        cv2.imshow("Face Recognition", frame)
    publish_frame(frame)

# ========================= SCREENSHOT ================================
def save_screenshot(frame):
    """Save exactly what is on screen (boxes, labels, zones, HUD) to
    the screenshot/ folder with a timestamped filename. Returns the
    path, or None on failure."""
    try:
        os.makedirs(SCREENSHOT_DIR, exist_ok=True)
        fname = time.strftime("screenshot_%Y%m%d_%H%M%S.png")
        path = os.path.join(SCREENSHOT_DIR, fname)
        # If two shots land in the same second, add a counter suffix.
        if os.path.exists(path):
            n = 2
            while os.path.exists(path):
                path = os.path.join(
                    SCREENSHOT_DIR,
                    time.strftime("screenshot_%Y%m%d_%H%M%S") + f"_{n}.png")
                n += 1
        cv2.imwrite(path, frame)
        print(f"[SHOT] saved {path}")
        return path
    except Exception as e:
        print(f"[SHOT] save failed: {e}")
        return None

# ========================= FPS COUNTER ===============================
def calculate_fps():
    global frame_count, start_time, fps
    frame_count += 1
    elapsed = time.time() - start_time
    if elapsed > 1:
        fps = frame_count / elapsed
        frame_count = 0
        start_time = time.time()
    return fps

# ============================ MAIN LOOP ==============================
start_stream_server()
print("[INFO] starting...")
print(f"[INFO] Sensor: BINNED {SENSOR_BINNED} <-> FULL request "
      f"{SENSOR_FULL} capped at {SENSOR_MAX_REQ} "
      f"(auto-switch at {ZOOM_FULLRES_ON}x zoom, native {SENSOR_NATIVE})")
print(f"[INFO] Far encodings: "
      f"{'loaded' if HAS_FAR_ENCODINGS else 'NOT FOUND - using close'}")
print("Controls:")
print("  LOOK AROUND  T/F/G/B = up/left/right/down | V re-centre")
print("               (arrow keys and 8/4/6/2 also work)")
print("               zoom in first: at 1.0x there is nowhere to pan")
print("  ZOOM         = in | - out | A auto zoom on/off")
print("  CALIB        D distance (stand at 1m) | K camera height")
print("  OTHER        H sensor | E encodings | I info | R reload")
print("               L screenshot | S zones | C clear | Q quit")

while True:
    if selection_mode:
        if frozen_main is None:
            frozen_main = picam2.capture_array("main").copy()
            frozen_display = cv2.resize(frozen_main, (DISPLAY_W, DISPLAY_H),
                                        interpolation=cv2.INTER_AREA)
        show(render_selection_frame())
        if HEADLESS:
            time.sleep(0.015); key = 255
        else:
            key = cv2.waitKey(15) & 0xFF
        web = get_key_web()
        if web is not None:
            key = ord(web)
        if key in (ord('s'), ord('S'), 27):
            selection_mode = False
            frozen_main = None
            frozen_display = None
            tracked_faces = []
            cache_reset = True
            print(f"Selection confirmed - {len(zones)} zone(s) active")
        elif key in (ord('c'), ord('C')):
            zones.clear()
        elif key == ord('q'):
            break
        continue

    maybe_switch_sensor()

    display_raw = picam2.capture_array("lores")
    display_frame = cv2.cvtColor(display_raw, LORES_COLOR)
    if display_frame.shape[1] != DISPLAY_W or display_frame.shape[0] != DISPLAY_H:
        display_frame = cv2.resize(display_frame, (DISPLAY_W, DISPLAY_H),
                                   interpolation=cv2.INTER_LINEAR)

    # Feed the worker a fresh MAIN frame the moment the previous pass
    # finishes. Detection-only passes are fast, so boxes stay current.
    # The crop in effect at capture rides along, so results come back
    # anchored to the right coordinates even after the crop moves.
    if not processing:
        main_frame = picam2.capture_array("main")
        with main_frame_lock:
            latest_main_frame = main_frame
            latest_frame_meta = (zoom_factor, last_applied_pan_x,
                                 last_applied_pan_y, time.time())
        processing = True

    if auto_zoom_enabled and not zones:
        with result_lock:
            rt = last_result_time
            locs = list(face_locations)
            meta = result_meta

        # Step the targets ONCE per worker result. The old loop
        # recomputed them every display frame from stale coordinates,
        # so the zoom ramped ~25x faster than intended and the pan
        # chased face positions measured under an old crop. Both made
        # the camera wander to random spots.
        if rt != consumed_result_time:
            consumed_result_time = rt
            steer = [b for b in locs if (b[2] - b[0]) >= AUTO_MIN_FACE_PX]
            auto_zoom_target = compute_auto_zoom(steer, auto_zoom_target)
            pan_result = compute_auto_pan(steer)
            if pan_result is not None:
                fx, fy = pan_result
                mz, mpx, mpy, _cap = meta
                # Convert to an absolute sensor-space target using the
                # crop in effect when THAT frame was captured.
                #
                # SIGN: ScalerCrop lives in fixed sensor coordinates, but
                # Transform flips the OUTPUT image. So with a flip, moving
                # the crop one way moves the view the other way, and the
                # feedback has to invert or the pan chases away from the
                # face and slams into the clamp.
                ex = -(fx - 0.5) if HFLIP else (fx - 0.5)
                ey = -(fy - 0.5) if VFLIP else (fy - 0.5)
                pan_target_x_abs = mpx + ex / max(mz, 1.0)
                pan_target_y_abs = mpy + ey / max(mz, 1.0)
                pan_target_x_abs = max(0.0, min(1.0, pan_target_x_abs))
                pan_target_y_abs = max(0.0, min(1.0, pan_target_y_abs))
            else:
                pan_target_x_abs = 0.5
                pan_target_y_abs = 0.5

        auto_zoom_current += (auto_zoom_target - auto_zoom_current) * 0.08
        pan_current_x += (pan_target_x_abs - pan_current_x) * 0.08
        pan_current_y += (pan_target_y_abs - pan_current_y) * 0.08
        # Clamp to what the crop can actually do, not 0..1, or auto-pan
        # winds up at the edges and takes a moment to respond coming back.
        pan_current_x = pan_clamp(pan_current_x)
        pan_current_y = pan_clamp(pan_current_y)

        zoom_changed = abs(auto_zoom_current - last_applied_zoom) > 0.05
        pan_changed = (abs(pan_current_x - last_applied_pan_x) > 0.01 or
                       abs(pan_current_y - last_applied_pan_y) > 0.01)

        if zoom_changed or pan_changed:
            zoom_factor = round(auto_zoom_current, 2)
            apply_zoom(zoom_factor, pan_current_x, pan_current_y)
            last_applied_zoom = auto_zoom_current
            last_applied_pan_x = pan_current_x
            last_applied_pan_y = pan_current_y

    if zones:
        display_frame = draw_zones(display_frame)

    display_frame = draw_results(display_frame)
    update_announcements()
    current_fps = calculate_fps()
    display_frame = draw_hud(display_frame, current_fps, zoom_factor,
                             auto_zoom_enabled)

    # This is the fully composited image, exactly what you see. Keep a
    # clean copy so a screenshot never includes the SAVED flash.
    clean_frame = display_frame.copy()

    if time.time() < screenshot_flash_until:
        cv2.rectangle(display_frame, (0, 0),
                      (DISPLAY_W - 1, DISPLAY_H - 1), (255, 255, 255), 6)
        cv2.putText(display_frame, "SAVED", (DISPLAY_W // 2 - 60, 90),
                    cv2.FONT_HERSHEY_SIMPLEX, 1.2, (255, 255, 255), 3,
                    cv2.LINE_AA)

    show(display_frame)

    # Zones drawn by dragging on the browser video.
    if drain_web_zones():
        tracked_faces = []
        cache_reset = True

    # Local keys from the window (arrow codes survive waitKeyEx), plus
    # any key pressed in the browser. Headless still needs a short wait
    # to yield the CPU even with no window.
    if HEADLESS:
        time.sleep(0.004)
        raw = -1
    else:
        raw = cv2.waitKeyEx(1)
    key = (raw & 0xFF) if raw != -1 else 255
    web = get_key_web()
    if web is not None:
        key = ord(web)          # a browser press overrides this frame

    # ---- manual pan: T/F/G/B, arrows, or numpad 8/4/6/2, V/5 centre ----
    if raw in PAN_KEYS_UP or key in (ord('8'), ord('t'), ord('T')):
        manual_pan(0.0, -PAN_STEP)
        continue
    if raw in PAN_KEYS_DOWN or key in (ord('2'), ord('b'), ord('B')):
        manual_pan(0.0, PAN_STEP)
        continue
    if raw in PAN_KEYS_LEFT or key in (ord('4'), ord('f'), ord('F')):
        manual_pan(-PAN_STEP, 0.0)
        continue
    if raw in PAN_KEYS_RIGHT or key in (ord('6'), ord('g'), ord('G')):
        manual_pan(PAN_STEP, 0.0)
        continue
    if key in (ord('5'), ord('v'), ord('V')):
        center_pan()
        continue

    if key in (ord('l'), ord('L')):
        if save_screenshot(clean_frame) is not None:
            screenshot_flash_until = time.time() + 0.5

    elif key == ord('='):
        zoom_to(min(zoom_factor + 0.5, 8.0))

    elif key == ord('-'):
        zoom_to(max(zoom_factor - 0.5, 1.0))

    elif key in (ord('a'), ord('A')):
        auto_zoom_enabled = not auto_zoom_enabled
        auto_zoom_current = zoom_factor
        if not auto_zoom_enabled:
            pan_current_x = 0.5
            pan_current_y = 0.5
            pan_target_x_abs = 0.5
            pan_target_y_abs = 0.5
        print(f"Auto zoom: {'ON' if auto_zoom_enabled else 'OFF'}")

    elif key in (ord('h'), ord('H')):
        cycle_sensor_pref()

    elif key in (ord('e'), ord('E')):
        cycle_encoding_mode()

    elif key in (ord('d'), ord('D')):
        calibrate_distance()

    elif key in (ord('i'), ord('I')):
        info_mode = "full" if info_mode == "basic" else "basic"
        print(f"Info mode: {info_mode}")

    elif key in (ord('k'), ord('K')):
        calibrate_cam_height()

    elif key in (ord('r'), ord('R')):
        # Reload: throw away all tracks and cached identities and let
        # the detector rebuild from the current frame. Use this if a box
        # is stuck in the wrong place or a face is not being picked up.
        tracked_faces = []
        cache_reset = True
        with result_lock:
            face_locations = []
            face_names = []
            face_confidences = []
            face_zones = []
        af_kick()
        announced_names.clear()
        present_names.clear()
        print("Reload: tracks and identity cache cleared, re-detecting")

    elif key in (ord('s'), ord('S')):
        if HEADLESS:
            print("Headless: drag on the browser video to draw a zone")
        else:
            selection_mode = True
            frozen_main = None
            frozen_display = None
            print("Selection mode - draw zones on the frozen high-res frame")

    elif key in (ord('c'), ord('C')):
        zones.clear()
        tracked_faces = []
        cache_reset = True
        print("All zones cleared")

    elif key == ord('q'):
        break

cv2.destroyAllWindows()
picam2.stop()
```

</details>

---

### Model Training (build_encodings.py)

Note: the recognition script uses InsightFace embeddings, which are different from the older face_recognition (dlib) embeddings. Enroll people with an InsightFace based script that writes `encodings_close.pickle` and `encodings_far.pickle` in the same 512 number format the recognition script expects. The capture scripts below still work for taking the photos; only the encoding step changed.

---

### Headshot Capture (headshots_capture-picam.py)

Takes training photos at 4K resolution with live preview, autofocus triggering before each shot, and a 1-second cooldown to prevent accidental double captures.

<details>
<summary><b>Click to view headshots_capture-picam.py</b></summary>

```python
import cv2
import os
from datetime import datetime
from picamera2 import Picamera2
from libcamera import controls, Transform
import time
import numpy as np

PERSON_NAME = "Name"
CAPTURE_SIZE = (3840, 2160)
DISPLAY_SIZE = (1024, 600)

def create_folder(name):
    person_folder = os.path.join("dataset", name)
    os.makedirs(person_folder, exist_ok=True)
    return person_folder

def apply_zoom(picam2, z):
    size = picam2.camera_properties['PixelArraySize']
    fw, fh = size
    cw = int(fw / z)
    ch = int(fh / z)
    cx = (fw - cw) // 2
    cy = (fh - ch) // 2
    picam2.set_controls({"ScalerCrop": (cx, cy, cw, ch)})

def update_zoom_full(picam2, z):
    size = picam2.camera_properties['PixelArraySize']
    fw, fh = size
    cw = int(fw / z)
    ch = int(fh / z)
    cx = (fw - cw) // 2
    cy = (fh - ch) // 2
    picam2.set_controls({
        "ScalerCrop": (cx, cy, cw, ch),
        "AfMode": controls.AfModeEnum.Auto,
        "AfTrigger": controls.AfTriggerEnum.Start,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })
    time.sleep(0.5)
    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Continuous,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })

def capture_photos(name):
    folder = create_folder(name)
    picam2 = Picamera2()
    config = picam2.create_preview_configuration(
        main={"size": CAPTURE_SIZE, "format": "RGB888"},
        lores={"size": (640, 480), "format": "YUV420"},
        transform=Transform(hflip=True, vflip=True)
    )
    picam2.configure(config)
    picam2.start()

    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Auto,
        "AfTrigger": controls.AfTriggerEnum.Start,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })
    time.sleep(2)
    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Continuous,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })

    zoom_factor = 1.0
    apply_zoom(picam2, zoom_factor)

    photo_count = 0
    last_photo_time = 0
    flash_frames = 0
    DISPLAY_W, DISPLAY_H = DISPLAY_SIZE

    print(f"""
CLOSE RANGE CAPTURE - {name}
Stand 0.5-2 meters from the camera
Vary angles between shots

Controls:
  SPACE     = take photo
  = / -     = zoom in / out
  Q         = quit
""")

    while True:
        preview = picam2.capture_array("lores")
        preview = cv2.cvtColor(preview, cv2.COLOR_YUV420p2RGB)
        preview = cv2.resize(preview, DISPLAY_SIZE, interpolation=cv2.INTER_LINEAR)
        h, w = preview.shape[:2]

        if flash_frames > 0:
            overlay = preview.copy()
            cv2.rectangle(overlay, (0,0), (w,h), (255,255,255), -1)
            cv2.addWeighted(overlay, 0.3, preview, 0.7, 0, preview)
            flash_frames -= 1

        bar = preview.copy()
        cv2.rectangle(bar, (0,0), (w,50), (0,0,0), -1)
        cv2.addWeighted(bar, 0.55, preview, 0.45, 0, preview)
        cv2.putText(preview, f"ZOOM  {zoom_factor:.1f}x", (10,32),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0,220,80), 2, cv2.LINE_AA)
        cv2.putText(preview, f"CLOSE DATASET - {name.upper()}", (180,32),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0,220,255), 1, cv2.LINE_AA)
        cv2.putText(preview, f"Photos: {photo_count}", (w-150,32),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (180,180,180), 1, cv2.LINE_AA)

        box_w = w // 3
        box_h = int(h * 0.6)
        bx1 = w//2 - box_w//2
        by1 = h//2 - box_h//2
        bx2 = w//2 + box_w//2
        by2 = h//2 + box_h//2
        corner = 20
        color = (0, 220, 255)
        for (x, y, dx, dy) in [(bx1,by1,1,1),(bx2,by1,-1,1),(bx1,by2,1,-1),(bx2,by2,-1,-1)]:
            cv2.line(preview, (x,y), (x+dx*corner,y), color, 2)
            cv2.line(preview, (x,y), (x,y+dy*corner), color, 2)
        cv2.putText(preview, "Align face here", (bx1, by1-8),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.45, color, 1, cv2.LINE_AA)

        cooldown = time.time() - last_photo_time
        if cooldown < 1.0:
            cv2.putText(preview, "WAIT...", (10, h//2),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0,165,255), 2, cv2.LINE_AA)
        else:
            cv2.putText(preview, "READY", (10, h//2),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0,220,80), 2, cv2.LINE_AA)

        if photo_count > 0 and photo_count % 10 == 0:
            cv2.putText(preview, f"Try a different angle - {photo_count} taken",
                        (10, h-45), cv2.FONT_HERSHEY_SIMPLEX, 0.42, (0,220,255), 1, cv2.LINE_AA)

        target = 50
        progress = min(photo_count / target, 1.0)
        bar_x1, bar_y1 = 10, h-50
        bar_x2, bar_y2 = w-10, h-42
        cv2.rectangle(preview, (bar_x1,bar_y1), (bar_x2,bar_y2), (40,40,40), -1)
        fill_x = int(bar_x1 + (bar_x2-bar_x1) * progress)
        bar_color = (0,220,80) if progress >= 1.0 else (0,180,255)
        cv2.rectangle(preview, (bar_x1,bar_y1), (fill_x,bar_y2), bar_color, -1)
        cv2.putText(preview, f"{photo_count}/{target}", (fill_x+4, bar_y2-1),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.35, (200,200,200), 1, cv2.LINE_AA)

        bar2 = preview.copy()
        cv2.rectangle(bar2, (0,h-36), (w,h), (0,0,0), -1)
        cv2.addWeighted(bar2, 0.55, preview, 0.45, 0, preview)
        cv2.putText(preview, "SPACE  capture     =  zoom in     -  zoom out     Q  quit",
                    (10,h-10), cv2.FONT_HERSHEY_SIMPLEX, 0.38, (150,150,150), 1, cv2.LINE_AA)

        cv2.imshow("Close Headshot Capture", preview)
        key = cv2.waitKey(1) & 0xFF

        if key == ord('='):
            zoom_factor = min(zoom_factor + 0.5, 8.0)
            update_zoom_full(picam2, zoom_factor)
            print(f"Zoom: {zoom_factor}x")

        elif key == ord('-'):
            zoom_factor = max(zoom_factor - 0.5, 1.0)
            update_zoom_full(picam2, zoom_factor)
            print(f"Zoom: {zoom_factor}x")

        elif key == ord(' '):
            if time.time() - last_photo_time < 1.0:
                continue
            photo_count += 1
            last_photo_time = time.time()
            flash_frames = 5

            picam2.set_controls({
                "AfMode": controls.AfModeEnum.Auto,
                "AfTrigger": controls.AfTriggerEnum.Start
            })
            time.sleep(0.5)
            picam2.set_controls({"AfMode": controls.AfModeEnum.Continuous})

            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            filepath = os.path.join(folder, f"{name}_z{zoom_factor:.1f}_{timestamp}.jpg")
            full_frame = picam2.capture_array("main")
            cv2.imwrite(filepath, full_frame, [cv2.IMWRITE_JPEG_QUALITY, 95])
            print(f"Saved {photo_count}/50: {filepath}")

            if photo_count % 10 == 0:
                print(f"{photo_count} photos - try a different angle now")

        elif key == ord('q'):
            break

    cv2.destroyAllWindows()
    picam2.stop()
    print(f"\nDone. {photo_count} photos saved for {name}.")
    if photo_count < 30:
        print(f"Warning: only {photo_count} photos - aim for 50")

if __name__ == "__main__":
    capture_photos(PERSON_NAME)
```

</details>


---

### Headshot Capture (headshots_capture-far.py)

Changes: starts at 2x zoom since shooting at distance, 1080p capture instead of 4K to match live feed resolution, distance rings overlay.

<details>
<summary><b>Click to view headshots_capture-far.py</b></summary>

```python
import cv2
import os
from datetime import datetime
from picamera2 import Picamera2
from libcamera import controls, Transform
import time
import numpy as np

PERSON_NAME = "Name"
CAPTURE_SIZE = (1920, 1080)
DISPLAY_SIZE = (1024, 600)

def create_folder(name):
    person_folder = os.path.join("dataset_far", name)
    os.makedirs(person_folder, exist_ok=True)
    return person_folder

def apply_zoom(picam2, z):
    size = picam2.camera_properties['PixelArraySize']
    fw, fh = size
    cw = int(fw / z)
    ch = int(fh / z)
    cx = (fw - cw) // 2
    cy = (fh - ch) // 2
    picam2.set_controls({"ScalerCrop": (cx, cy, cw, ch)})

def update_zoom_full(picam2, z):
    size = picam2.camera_properties['PixelArraySize']
    fw, fh = size
    cw = int(fw / z)
    ch = int(fh / z)
    cx = (fw - cw) // 2
    cy = (fh - ch) // 2
    picam2.set_controls({
        "ScalerCrop": (cx, cy, cw, ch),
        "AfMode": controls.AfModeEnum.Auto,
        "AfTrigger": controls.AfTriggerEnum.Start,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })
    time.sleep(0.5)
    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Continuous,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })

def capture_photos(name):
    folder = create_folder(name)
    picam2 = Picamera2()
    config = picam2.create_preview_configuration(
        main={"size": CAPTURE_SIZE, "format": "RGB888"},
        lores={"size": (640, 480), "format": "YUV420"},
        transform=Transform(hflip=True, vflip=True)
    )
    picam2.configure(config)
    picam2.start()

    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Auto,
        "AfTrigger": controls.AfTriggerEnum.Start,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })
    time.sleep(2)
    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Continuous,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })

    zoom_factor = 2.0
    apply_zoom(picam2, zoom_factor)

    photo_count = 0
    last_photo_time = 0
    flash_frames = 0
    DISPLAY_W, DISPLAY_H = DISPLAY_SIZE

    print(f"""
DISTANCE TRAINING CAPTURE - {name}
Stand 3-5 meters from the camera
Vary angles slightly between shots

Controls:
  SPACE     = take photo
  = / -     = zoom in / out
  Q         = quit
""")

    while True:
        preview = picam2.capture_array("lores")
        preview = cv2.cvtColor(preview, cv2.COLOR_YUV420p2RGB)
        preview = cv2.resize(preview, DISPLAY_SIZE, interpolation=cv2.INTER_LINEAR)
        h, w = preview.shape[:2]

        if flash_frames > 0:
            overlay = preview.copy()
            cv2.rectangle(overlay, (0,0), (w,h), (255,255,255), -1)
            cv2.addWeighted(overlay, 0.3, preview, 0.7, 0, preview)
            flash_frames -= 1

        bar = preview.copy()
        cv2.rectangle(bar, (0,0), (w,50), (0,0,0), -1)
        cv2.addWeighted(bar, 0.55, preview, 0.45, 0, preview)
        cv2.putText(preview, f"ZOOM  {zoom_factor:.1f}x", (10,32),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0,220,80), 2, cv2.LINE_AA)
        cv2.putText(preview, f"FAR DATASET - {name.upper()}", (180,32),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0,220,255), 1, cv2.LINE_AA)
        cv2.putText(preview, f"Photos: {photo_count}", (w-150,32),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (180,180,180), 1, cv2.LINE_AA)

        box_w = w // 4
        box_h = h // 3
        bx1 = w//2 - box_w//2
        by1 = h//2 - box_h//2
        bx2 = w//2 + box_w//2
        by2 = h//2 + box_h//2
        corner = 20
        color = (0, 220, 255)
        for (x, y, dx, dy) in [(bx1,by1,1,1),(bx2,by1,-1,1),(bx1,by2,1,-1),(bx2,by2,-1,-1)]:
            cv2.line(preview, (x,y), (x+dx*corner,y), color, 2)
            cv2.line(preview, (x,y), (x,y+dy*corner), color, 2)
        cv2.putText(preview, "Align face here", (bx1, by1-8),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.45, color, 1, cv2.LINE_AA)

        cx_ring, cy_ring = w//2, h//2
        for radius, label, ring_color in [
            (60, "3m", (0,255,150)),
            (100, "4m", (0,200,255)),
            (140, "5m", (0,160,255))
        ]:
            cv2.circle(preview, (cx_ring,cy_ring), radius, ring_color, 1)
            cv2.putText(preview, label, (cx_ring+radius+3, cy_ring),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.35, ring_color, 1, cv2.LINE_AA)

        cooldown = time.time() - last_photo_time
        if cooldown < 1.0:
            cv2.putText(preview, "WAIT...", (10, h//2),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0,165,255), 2, cv2.LINE_AA)
        else:
            cv2.putText(preview, "READY", (10, h//2),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0,220,80), 2, cv2.LINE_AA)

        if photo_count > 0 and photo_count % 5 == 0:
            cv2.putText(preview, f"Try a different angle - {photo_count} taken",
                        (10, h-45), cv2.FONT_HERSHEY_SIMPLEX, 0.42, (0,220,255), 1, cv2.LINE_AA)

        target = 15
        progress = min(photo_count / target, 1.0)
        bar_x1, bar_y1 = 10, h-50
        bar_x2, bar_y2 = w-10, h-42
        cv2.rectangle(preview, (bar_x1,bar_y1), (bar_x2,bar_y2), (40,40,40), -1)
        fill_x = int(bar_x1 + (bar_x2-bar_x1) * progress)
        bar_color = (0,220,80) if progress >= 1.0 else (0,180,255)
        cv2.rectangle(preview, (bar_x1,bar_y1), (fill_x,bar_y2), bar_color, -1)
        cv2.putText(preview, f"{photo_count}/{target}", (fill_x+4, bar_y2-1),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.35, (200,200,200), 1, cv2.LINE_AA)

        bar2 = preview.copy()
        cv2.rectangle(bar2, (0,h-36), (w,h), (0,0,0), -1)
        cv2.addWeighted(bar2, 0.55, preview, 0.45, 0, preview)
        cv2.putText(preview, "SPACE  capture     =  zoom in     -  zoom out     Q  quit",
                    (10,h-10), cv2.FONT_HERSHEY_SIMPLEX, 0.38, (150,150,150), 1, cv2.LINE_AA)

        cv2.imshow("Distance Headshot Capture", preview)
        key = cv2.waitKey(1) & 0xFF

        if key == ord('='):
            zoom_factor = min(zoom_factor + 0.5, 8.0)
            update_zoom_full(picam2, zoom_factor)
            print(f"Zoom: {zoom_factor}x")

        elif key == ord('-'):
            zoom_factor = max(zoom_factor - 0.5, 1.0)
            update_zoom_full(picam2, zoom_factor)
            print(f"Zoom: {zoom_factor}x")

        elif key == ord(' '):
            if time.time() - last_photo_time < 1.0:
                continue
            photo_count += 1
            last_photo_time = time.time()
            flash_frames = 5

            picam2.set_controls({
                "AfMode": controls.AfModeEnum.Auto,
                "AfTrigger": controls.AfTriggerEnum.Start
            })
            time.sleep(0.5)
            picam2.set_controls({"AfMode": controls.AfModeEnum.Continuous})

            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            filepath = os.path.join(folder, f"{name}_far_z{zoom_factor:.1f}_{timestamp}.jpg")
            full_frame = picam2.capture_array("main")
            cv2.imwrite(filepath, full_frame, [cv2.IMWRITE_JPEG_QUALITY, 95])
            print(f"Saved {photo_count}: {filepath} (zoom {zoom_factor}x)")

            if photo_count % 5 == 0:
                print(f"{photo_count} photos - try a different angle or zoom level now")

        elif key == ord('q'):
            break

    cv2.destroyAllWindows()
    picam2.stop()
    print(f"\nDone. {photo_count} distance photos saved for {name}.")
    if photo_count < 15:
        print(f"Tip: aim for 15+ photos at varied angles and zoom levels")

if __name__ == "__main__":
    capture_photos(PERSON_NAME)
```

</details>


---

## Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| 10.1" Security Monitor | Displays the live recognition feed | $78 | <a href="https://www.amazon.com/Haiway-Security-Surveillance-Controller-Resolution/dp/B07WKG9J35?th=1">Link</a> |
| Raspberry Pi 4 Starter Kit | Pi 4, micro SD, power supply, and case | $150 | <a href="https://www.amazon.com/CanaKit-Raspberry-4GB-Starter-Kit/dp/B07V5JTMV9">Link</a> |
| Arducam 64MP Hawkeye | 64MP autofocus camera for long-range detection | $50 | <a href="https://www.amazon.com/Raspberry-Pi-Camera-Module/dp/B0BRY6MVXL">Link</a> |
| Keyboard and Mouse | Controls the Pi | $15 | <a href="https://www.amazon.com/Logitech-Keyboard-Windows-Optical-Full-Size/dp/B003NREDC8">Link</a> |
| Custom 3d printed Case | Houses the Pi | ~ | <a href="https://cad.onshape.com/documents/c6fa1683221d288d393bb5ee/w/c7001c4204dbef570c5fb88e/e/94082adbb27febdda5911354?renderMode=0&uiState=6a5e4378b13ce118680cd69e">Link</a> |

---

## Other Resources/Examples

- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)
