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

-Trained the model on images in the dataset using Python, allowing the live feed to show people’s names

-Surprised by how much data the model needs, only 200 pictures of 4 people in the dataset wasn’t enough to allow the model to recognize people past 3 feet

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

The fix: The camera has a 64 megapixel sensor. There’s more detail in the sensor than the screen. The image can be stretched during digitizing, because more pixels=more detail. However, making an image bigger loses no detail, so I can cut into the sensor to capture a face and its hundreds of pixels. This digital zoom works with the =and - keys, on the keyboard. I made it also automatic.

## Blurry and corrupted training photos

The problem: When the camera was set up, only when someone was in front of the camera did it capture sharp photos. Some of my saved photos became corrupted without any warning. This meant deleting many of my photos.

The fix: Adding autofocus to the camera means that the camera will focus when every photo is taken. It will also cool down to give some time to ensure that the autofocus will not be taken twice in a row. Saving the photos at high resolution with a quality set at the camera ensured that no more photos would be corrupted.

## Not enough training data

The problem: My data set had only 200 photos of 4 people. The model worked well for people at close range, but failed at 3 feet away.

The fix: I take more photos of myself from many angles and distances. I used image augmentation to take each of my photos and make a set of 6 photos of myself from flipped, brighter, darker, and rotated images. This teaches my model to see more lighting angles, without taking more of my photos.

## Switching the recognition engine

The first recognition engine used was the face_recognition library together with MediaPipe. It worked well for people up-close, but would fail at a distance or high accuracy.

I changed to InsightFace, which is a stronger and more modern recognition engine together with the buffalo_l model. This model converts a face to a list of 512 numbers called an embedding. These numbers act like a fingerprint. By comparing the embedding to their saved embeddings, my program can recognize which face is which.

## The speed problem

The problem: The new insightface model is slow on the Raspberry Pi. It takes close to a second to recognize a person. However, the model often freezes when drawing a box around the recognized face. It freezes again after the name is found.

The fix: Splitting the model into two jobs can run at the same time. The first job is the slow job to recognize the face. The second job is the fast job to run each frame of video and follow the face using a small image of face called a template. The template is used to follow the face and box it in every frame.

## Boxes stuck on the wall

The problem: The box with my name was stuck on the wall behind me. The model was tracking my face, but it was moving to the wall and stuck to the wall.

The fix: Adding three guards to the model will make it recognize wrong tracking. The tracking model checks to see if the face has detail or not. A wall has no detail. If a box is not confirmed by the detector, it is automatically removed. The detector also raises the bar for confidence to avoid marking shadows onto the wall as a face.

## The box showing where I used to be

The problem: Since the model is slow, the box illustrating my face has a lag of a second. If I move my head, the box follows me, but stuck to where I was a second earlier.

The insight: The detector model is slow, so it should not be in charge of the position of the box. The fast tracking model owns the position of the box, but the slow detector model only supplies the name of the detected face and corrects the size of the box.

## The box only covering part of my face, or sitting off-center

The problem: The box is too small or moves to the side of my face.

The fix: The box now expands to cover my face or my head. If I move away or near the camera, the box adjusts to cover me. The box is made to automatically re-center my face. The box is also widened to show my whole head rather than only half my face. The box can now grow or shrink to fit my face.

## Losing the box while moving

The problem: If I move quicker than normal, the box is lost.

The fix: The tracking model is made more forgiving of motion. A motion predictor forecasts where my face will be on the next frame of video. This helps the tracking model to stay on my face even if I move quicker than normal.

## Running out of memory

The problem: When running at full resolution, the Raspberry Pi limits running out of memory.

The fix: Limiting the resolution at which the camera works. Fewer buffers of frames of video to hold. Raw memory to reserve for the python program to run. A message is set up in the program so that if the camera fails to start in high resolution mode, program falls back to a below resolution mode instead of crashing.

## Small bugs along the way

The video feed showed everyone with blue faces. It was a quirk of the color conversion. By swapping blue and red, it was fixed.

## Quality of life features

Once the basic model was working, I added a scan zone to focus the recognition on a portion of the screen. Auto zoom to focus on the face of the nearest person. A reload key to clear the screen. A screenshot key to save the screen shot. Also working in development is an offline voice module to announce a recognized person’s name when sufficient confidence above 60 percent.

## What I learned

In making this project, I learned that most of my time was spent closing the gap between a model that works in theory and the model that works in real time with cheap hardware. While making the recognition model, it was mostly challenging to make the box of the detected face smooth, accurate, and centered around my face at the same time as the slower model.
 
---  

## Schematics
<img src="Aarush%20H.jpeg" alt="Headstone Image" width="300" height="400">
<img src="Aarush%20H.jpeg" alt="Headstone Image" width="300" height="400">
<img src="Aarush%20H.jpeg" alt="Headstone Image" width="300" height="400">
<img src="Aarush%20H.jpeg" alt="Headstone Image" width="300" height="400">


---

## Code

### Face Recognition (face_rec-picam.py)
The main script. Runs continuous face recognition on the Pi Camera feed with InsightFace, a fast template tracker for smooth boxes, digital zoom, scan zones, live confidence scoring, a reload key, a screenshot key, and offline voice announcements. Controls: `=` zoom in, `-` zoom out, `A` auto zoom, `H` sensor mode, `E` encoding set, `R` reload, `L` screenshot, `S` zones, `C` clear zones, `Q` quit.

```python
import cv2
import numpy as np
from picamera2 import Picamera2
from libcamera import controls, Transform
import time
import pickle
import os
import subprocess
from queue import Queue, Empty
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
LORES_W, LORES_H = 1024, 576    # display source stream
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
MATCH_THRESHOLD = 0.34          # min correlation to MOVE the box. Kept
                                # low (like the early versions) so the
                                # box follows through motion blur. On a
                                # miss the box holds; misses cull ghosts
PREDICT_GAIN = 0.7              # how much last-frame velocity shifts
                                # the search window ahead of the face
MIN_TEMPLATE_STD = 12.0         # tracked patch must have texture, so
                                # templates never latch onto flat walls
RESEED_SEARCH_MARGIN = 300      # a LAGGED detection still associates
                                # with your moved box instead of
                                # spawning a phantom track behind you
SPAWN_MIN_DIST = 150            # a new track spawns only when a
                                # detection is this far from every
                                # existing track (kills phantom dupes)
DRIFT_CORRECT_DIST = 140        # if a detection lands within this of
                                # the tracked box, re-center the box on
                                # the detected face (kills the slow
                                # template drift that pulls the box off
                                # your face). Farther than this, the
                                # detection is treated as stale and does
                                # NOT move the box (no yank while moving)
DRIFT_CORRECT_BLEND = 0.6       # firm re-center toward the true face
                                # center each detection pass
SIZE_BLEND = 0.35               # how fast the box grows/shrinks toward
                                # the detected face size (0..1 per pass)
TRACK_MAX_MISSES = 6            # detector passes with no matching
                                # detection before a track is removed
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
AUTO_MIN_FACE_PX = 30

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
app = FaceAnalysis(
    name=MODEL_PACK,
    allowed_modules=["detection", "recognition"],
    providers=["CPUExecutionProvider"]
)
app.prepare(ctx_id=-1, det_size=DET_SIZE, det_thresh=DET_CONFIDENCE)
rec_model = app.models.get("recognition")
if rec_model is None:
    raise RuntimeError("Recognition model missing from pack "
                       f"'{MODEL_PACK}'")
print("[INFO] InsightFace ready")

def detect_only(rgb):
    """Fast pass: SCRFD detection only. Returns (bboxes Nx5, kpss)."""
    bboxes, kpss = app.det_model.detect(rgb, max_num=0, metric="default")
    return bboxes, kpss

def compute_embedding(rgb, bbox, kps, det_score):
    """Heavy pass: ArcFace 512-D embedding for one detected face.
    NOTE: color handling matches v1 and your enrollment pipeline.
    Do not change the BGR->RGB conversion unless you re-enroll."""
    face = Face(bbox=np.asarray(bbox, dtype=np.float32),
                kps=np.asarray(kps, dtype=np.float32),
                det_score=float(det_score))
    rec_model.get(rgb, face)
    return face.embedding

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

def make_config(sensor_size, buffers):
    """The 'sensor' hint selects the readout mode WITHOUT allocating
    raw buffers to the app. On a Camera Module 3 in full mode this
    saves roughly 60 MB of contiguous memory versus requesting a raw
    stream. Falls back to a raw request on older picamera2."""
    common = dict(
        main={"size": (MAIN_W, MAIN_H), "format": "RGB888"},
        lores={"size": (LORES_W, LORES_H), "format": "YUV420"},
        transform=Transform(hflip=True, vflip=True),
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
latest_frame_meta = (zoom_factor, 0.5, 0.5)   # crop (zoom, panx, pany)
                                              # in effect at capture
result_meta = (zoom_factor, 0.5, 0.5)         # crop the latest worker
                                              # results were measured in
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

cv2.namedWindow("Face Recognition")
cv2.setMouseCallback("Face Recognition", mouse_callback)

# ===================== IDENTITY CACHE (worker) =======================
cache_reset = False             # set by the main thread after camera
                                # moves or zone edits; worker clears
                                # the cache at the next pass

def cache_match(cx, cy, fh):
    """A detection only inherits a cached identity when close AND of
    similar size. Without the size check, a false detection on a wall
    near a person could pick up their name without any embedding."""
    best = None
    best_d = max(120.0, fh * 1.5)
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

    def publish(locs, names, confs, fzones, meta):
        global face_locations, face_names, face_confidences, face_zones
        global result_meta, last_result_time
        with result_lock:
            face_locations = list(locs)
            face_names = list(names)
            face_confidences = list(confs)
            face_zones = list(fzones)
            result_meta = meta
            last_result_time = time.time()

    while True:
        if not processing:
            time.sleep(0.005)
            continue

        if cache_reset:
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

            entry = cache_match(cx, cy, fh)
            if entry is not None:
                entry["cx"], entry["cy"], entry["h"] = cx, cy, fh
                entry["seen"] = now
                name, conf = entry["name"], entry["conf"]
            else:
                name, conf = PENDING, 0.0    # not yet embedded

            idx = len(locations)
            locations.append((int(my1), int(mx2), int(my2), int(mx1)))
            names.append(name)
            confs.append(conf)
            zones_found.append(zone_idx)

            fresh = entry is not None and (now - entry["t"]) <= REVERIFY_SEC
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

        publish(locations, names, confs, zones_found, frame_meta)

        # ---- Phase 2: embeddings, biggest faces first ----
        jobs.sort(key=lambda j: j["fh"], reverse=True)
        embeds_done = 0
        for job in jobs:
            if embeds_done >= MAX_EMBEDS_PER_PASS:
                break
            emb = compute_embedding(job["rgb"], job["bbox"],
                                    job["kps"], job["score"])
            if emb is None:
                continue
            name, conf = match_face(emb, active_encodings,
                                    active_names_arr, job["threshold"])
            embeds_done += 1
            entry = job["entry"]
            if entry is None:
                entry = {"cx": job["cx"], "cy": job["cy"], "h": job["fh"],
                         "name": name, "conf": conf, "t": now, "seen": now}
                identity_cache.append(entry)
            else:
                entry["name"] = name
                entry["conf"] = conf
                entry["t"] = now
            names[job["idx"]] = name
            confs[job["idx"]] = conf

        if embeds_done > 0:
            publish(locations, names, confs, zones_found, frame_meta)

        cache_prune(now)
        processing = False

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

    identities = []
    for (top, right, bottom, left), name, conf, zone_idx in zip(
            locs, names, confs, fzones):
        # Keep the TIGHT SCRFD box for tracking. The template is
        # grabbed from this region, so it stays on face texture and
        # never picks up the wall. Expansion happens only at draw time.
        top_d = int(top * SY)
        bottom_d = int(bottom * SY)
        left_d = int(left * SX)
        right_d = int(right * SX)
        identities.append({
            "box": (top_d, right_d, bottom_d, left_d),
            "cx": (left_d + right_d) / 2,
            "cy": (top_d + bottom_d) / 2,
            "name": name,
            "confidence": conf,
            "zone_idx": zone_idx,
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

def update_tracks(gray_frame):
    global tracked_faces
    h, w = gray_frame.shape[:2]
    for t in tracked_faces:
        top, right, bottom, left = t["box"]
        # Predict where the face is heading and center the search there,
        # so a fast head move does not outrun the search window.
        vx = t.get("vx", 0.0) * PREDICT_GAIN
        vy = t.get("vy", 0.0) * PREDICT_GAIN
        pleft = left + vx
        ptop = top + vy
        pright = right + vx
        pbottom = bottom + vy
        m = TRACK_SEARCH_MARGIN
        sx1 = int(max(0, pleft - m)); sy1 = int(max(0, ptop - m))
        sx2 = int(min(w, pright + m)); sy2 = int(min(h, pbottom + m))
        search = gray_frame[sy1:sy2, sx1:sx2]
        tmpl = t["template"]
        th, tw = tmpl.shape[:2]
        if th == 0 or tw == 0 or search.shape[0] < th or search.shape[1] < tw:
            continue                        # hold box, wait for reseed
        res = cv2.matchTemplate(search, tmpl, cv2.TM_CCOEFF_NORMED)
        _, max_val, _, max_loc = cv2.minMaxLoc(res)
        if max_val < MATCH_THRESHOLD:
            t["vx"] = t.get("vx", 0.0) * 0.5   # decay guess, hold box
            t["vy"] = t.get("vy", 0.0) * 0.5
            continue
        new_left = sx1 + max_loc[0]
        new_top = sy1 + max_loc[1]
        new_right = new_left + tw
        new_bottom = new_top + th
        # Velocity estimate for next frame's prediction (smoothed).
        ocx, ocy = (left + right) / 2.0, (top + bottom) / 2.0
        ncx, ncy = (new_left + new_right) / 2.0, (new_top + new_bottom) / 2.0
        t["vx"] = 0.5 * t.get("vx", 0.0) + 0.5 * (ncx - ocx)
        t["vy"] = 0.5 * t.get("vy", 0.0) + 0.5 * (ncy - ocy)
        t["box"] = (new_top, new_right, new_bottom, new_left)
        # Blend the template slowly toward the current patch instead of
        # replacing it. This keeps motion-blur tolerance but stops the
        # box slowly walking onto a high-contrast feature (eyes, glasses)
        # and sitting off-center. The detector re-centers it in reseed.
        new_tmpl = gray_frame[new_top:new_bottom, new_left:new_right]
        old_tmpl = t["template"]
        if (new_tmpl.size > 0 and new_tmpl.std() >= MIN_TEMPLATE_STD
                and new_tmpl.shape == old_tmpl.shape):
            t["template"] = cv2.addWeighted(old_tmpl, 0.7, new_tmpl, 0.3, 0)

def reseed_tracks(gray_frame, identities):
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
            # Drift correction: only when the detection is CLOSE to the
            # tracked box (you are roughly still) do we nudge the center
            # toward it. When it is far, you have moved and the
            # detection is stale, so we keep the tracker's live center
            # and never yank the box backward.
            if best_dist <= DRIFT_CORRECT_DIST:
                tcx += (cx - tcx) * DRIFT_CORRECT_BLEND
                tcy += (cy - tcy) * DRIFT_CORRECT_BLEND
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
            t["zone_idx"] = ident["zone_idx"]
            t["misses"] = 0
            used_tracks.add(matched)
        else:
            # Only spawn if this detection is not just a lagged copy of
            # an existing track sitting nearby.
            near = False
            for i in range(len(tracked_faces)):
                top, right, bottom, left = tracked_faces[i]["box"]
                tcx, tcy = (left + right) / 2, (top + bottom) / 2
                if ((tcx - cx) ** 2 + (tcy - cy) ** 2) ** 0.5 < SPAWN_MIN_DIST:
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
def draw_results(frame):
    global last_reseed_time, tracked_faces

    gray_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    with result_lock:
        age = time.time() - last_result_time
        current_result_time = last_result_time

    if age > IDENTITY_STALE_SEC:
        tracked_faces = []

    update_tracks(gray_frame)

    if current_result_time != last_reseed_time:
        identities = get_identified_faces()
        reseed_tracks(gray_frame, identities)
        last_reseed_time = current_result_time

    _, _, using_far = get_active_encodings()

    for t in tracked_faces:
        # Track a tight face box, draw an expanded one framing the head.
        top, right, bottom, left = expand_box(t["box"])
        name = track_display_name(t)
        confidence, zone_idx = t["confidence"], t["zone_idx"]

        pending = name is None       # identity not resolved yet
        if zone_idx >= 0:
            color = ZONE_COLORS[zone_idx]
        elif pending:
            color = (150, 150, 150)
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

        # No label while the identity resolves. The box just sits on
        # the face quietly instead of flashing a SCANNING tag.
        if pending:
            continue

        if name == "Unknown":
            label = "UNKNOWN"
        else:
            far_tag = " [FAR]" if using_far else ""
            zone_tag = f" [{ZONE_NAMES[zone_idx]}]" if zone_idx >= 0 else ""
            label = f"{name.upper()}  {confidence}%{far_tag}{zone_tag}"

        label_y = top - 10 if top > 30 else bottom + 25
        (lw, lh), _ = cv2.getTextSize(label, cv2.FONT_HERSHEY_SIMPLEX, 0.5, 1)
        cv2.rectangle(frame, (left, label_y - lh - 6),
                      (left + lw + 8, label_y + 2), color, -1)
        cv2.putText(frame, label, (left + 4, label_y - 2),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 0), 1, cv2.LINE_AA)

    return frame

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

    overlay2 = frame.copy()
    cv2.rectangle(overlay2, (0, h - 36), (w, h), (0, 0, 0), -1)
    cv2.addWeighted(overlay2, 0.55, frame, 0.45, 0, frame)
    cv2.putText(frame,
                "=  zoom in    -  zoom out    A  auto zoom    "
                "H  sensor    E  enc    R  reload    L  shot    S  zones    "
                "C  clear    Q  quit",
                (10, h - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.36,
                (150, 150, 150), 1, cv2.LINE_AA)
    return frame

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
print("[INFO] starting...")
print(f"[INFO] Sensor: BINNED {SENSOR_BINNED} <-> FULL request "
      f"{SENSOR_FULL} capped at {SENSOR_MAX_REQ} "
      f"(auto-switch at {ZOOM_FULLRES_ON}x zoom, native {SENSOR_NATIVE})")
print(f"[INFO] Far encodings: "
      f"{'loaded' if HAS_FAR_ENCODINGS else 'NOT FOUND - using close'}")
print("Controls: = zoom in | - zoom out | A auto zoom | H sensor mode | "
      "E encodings | R reload | L screenshot | S zones | C clear | Q quit")

while True:
    if selection_mode:
        if frozen_main is None:
            frozen_main = picam2.capture_array("main").copy()
            frozen_display = cv2.resize(frozen_main, (DISPLAY_W, DISPLAY_H),
                                        interpolation=cv2.INTER_AREA)
        cv2.imshow("Face Recognition", render_selection_frame())
        key = cv2.waitKey(15) & 0xFF
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
                                 last_applied_pan_y)
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
                mz, mpx, mpy = meta
                # Convert to an absolute sensor-space target using the
                # crop in effect when THAT frame was captured.
                pan_target_x_abs = mpx + (fx - 0.5) / max(mz, 1.0)
                pan_target_y_abs = mpy + (fy - 0.5) / max(mz, 1.0)
                pan_target_x_abs = max(0.0, min(1.0, pan_target_x_abs))
                pan_target_y_abs = max(0.0, min(1.0, pan_target_y_abs))
            else:
                pan_target_x_abs = 0.5
                pan_target_y_abs = 0.5

        auto_zoom_current += (auto_zoom_target - auto_zoom_current) * 0.08
        pan_current_x += (pan_target_x_abs - pan_current_x) * 0.08
        pan_current_y += (pan_target_y_abs - pan_current_y) * 0.08
        pan_current_x = max(0.0, min(1.0, pan_current_x))
        pan_current_y = max(0.0, min(1.0, pan_current_y))

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

    cv2.imshow("Face Recognition", display_frame)

    key = cv2.waitKey(1) & 0xFF

    if key in (ord('l'), ord('L')):
        if save_screenshot(clean_frame) is not None:
            screenshot_flash_until = time.time() + 0.5

    elif key == ord('='):
        auto_zoom_enabled = False
        zoom_factor = min(zoom_factor + 0.5, 8.0)
        auto_zoom_current = zoom_factor
        pan_current_x = 0.5
        pan_current_y = 0.5
        last_applied_zoom = zoom_factor
        last_applied_pan_x = 0.5
        last_applied_pan_y = 0.5
        pan_target_x_abs = 0.5
        pan_target_y_abs = 0.5
        update_zoom_full(zoom_factor, 0.5, 0.5)
        tracked_faces = []
        cache_reset = True
        maybe_switch_sensor()

    elif key == ord('-'):
        auto_zoom_enabled = False
        zoom_factor = max(zoom_factor - 0.5, 1.0)
        auto_zoom_current = zoom_factor
        pan_current_x = 0.5
        pan_current_y = 0.5
        last_applied_zoom = zoom_factor
        last_applied_pan_x = 0.5
        last_applied_pan_y = 0.5
        pan_target_x_abs = 0.5
        pan_target_y_abs = 0.5
        update_zoom_full(zoom_factor, 0.5, 0.5)
        tracked_faces = []
        cache_reset = True
        maybe_switch_sensor()

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

---

### Model Training (build_encodings.py)

Note: the recognition script now uses InsightFace embeddings, which are different from the older face_recognition (dlib) embeddings. Enroll people with an InsightFace based script that writes `encodings_close.pickle` and `encodings_far.pickle` in the same 512 number format the recognition script expects. The capture scripts below still work for taking the photos; only the encoding step changed.

---

### Headshot Capture (headshots_capture-picam.py)
Takes training photos at 4K resolution with live preview, autofocus triggering before each shot, and a 1-second cooldown to prevent accidental double captures.

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

---
### Headshot Capture (headshots_capture-far.py)
Changes: starts at 2x zoom since shooting at distance, 1080p capture instead of 4K to match live feed resolution, distance rings overlay.

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

## Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| 10.1" Security Monitor | Displays the live recognition feed | $78 | <a href="https://www.amazon.com/Haiway-Security-Surveillance-Controller-Resolution/dp/B07WKG9J35?th=1">Link</a> |
| Raspberry Pi 4 Starter Kit | Pi 4, micro SD, power supply, and case | $150 | <a href="https://www.amazon.com/CanaKit-Raspberry-4GB-Starter-Kit/dp/B07V5JTMV9">Link</a> |
| Arducam 64MP Hawkeye | 64MP autofocus camera for long-range detection | $50 | <a href="https://www.amazon.com/Raspberry-Pi-Camera-Module/dp/B0BRY6MVXL">Link</a> |
| Keyboard and Mouse | Controls the Pi | $15 | <a href="https://www.amazon.com/Logitech-Keyboard-Windows-Optical-Full-Size/dp/B003NREDC8">Link</a> |
| Custom 3d printed Case | Houses the Pi | ~ | <a  href="https://cad.onshape.com/documents/c6fa1683221d288d393bb5ee/w/c7001c4204dbef570c5fb88e/e/94082adbb27febdda5911354?renderMode=0&uiState=6a5e4378b13ce118680cd69e">Link</a> |

---

## Other Resources/Examples

- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)
