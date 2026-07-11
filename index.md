
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

## Schematics

![Schematics](Schematic.png)

---

## Code

### Face Recognition (face_rec-picam.py)
The main script. Runs continuous facial recognition on the Pi Camera feed with dynamic zoom, autofocus control, and live confidence scoring.

```python
import face_recognition
import cv2
import numpy as np
from picamera2 import Picamera2
from libcamera import controls, Transform
import time
import pickle
from threading import Thread, Lock

print("[INFO] loading encodings...")
with open("encodings.pickle", "rb") as f:
    data = pickle.loads(f.read())
known_face_encodings = data["encodings"]
known_face_names = data["names"]
print(f"[INFO] loaded {len(set(known_face_names))} people, {len(known_face_names)} total encodings")

# ── CAMERA INIT ──
picam2 = Picamera2()
config = picam2.create_preview_configuration(
    main={"size": (1280, 720), "format": "RGB888"},   # dropped from 1080p for speed
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

# ── STATE ──
zoom_factor = 1.5
cv_scaler = 3
frame_skip = 5

face_locations = []
face_names = []
face_distances_list = []
result_lock = Lock()

auto_zoom_enabled = True
auto_zoom_target = 1.5
auto_zoom_current = 1.5
DISPLAY_W, DISPLAY_H = 1024, 600

frame_count = 0
start_time = time.time()
fps = 0
frame_skip_count = 0

# ── ZOOM ──
def apply_zoom(z):
    size = picam2.camera_properties['PixelArraySize']
    fw, fh = size
    cw = int(fw / z)
    ch = int(fh / z)
    cx = (fw - cw) // 2
    cy = (fh - ch) // 2
    picam2.set_controls({"ScalerCrop": (cx, cy, cw, ch)})

def update_zoom_full(z):
    """Full zoom update with autofocus retrigger — only for manual zoom"""
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
    time.sleep(0.4)
    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Continuous,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })

def get_cv_scaler(z):
    if z >= 5.0: return 2
    elif z >= 3.0: return 3
    else: return 4

def get_frame_skip(z):
    if z >= 5.0: return 8
    elif z >= 3.0: return 6
    else: return 4

apply_zoom(zoom_factor)

# ── BACKGROUND FACE RECOGNITION THREAD ──
processing = False
latest_main_frame = None
main_frame_lock = Lock()

def recognition_worker():
    global face_locations, face_names, face_distances_list, processing
    while True:
        if not processing:
            time.sleep(0.01)
            continue

        with main_frame_lock:
            frame = latest_main_frame.copy() if latest_main_frame is not None else None

        if frame is None:
            processing = False
            continue

        scaler = cv_scaler
        resized = cv2.resize(frame, (0, 0), fx=(1/scaler), fy=(1/scaler))
        rgb = cv2.cvtColor(resized, cv2.COLOR_BGR2RGB)

        locs = face_recognition.face_locations(rgb, model="hog", number_of_times_to_upsample=1)
        encs = face_recognition.face_encodings(rgb, locs, model="small")  # small for speed

        names = []
        dists = []
        for enc in encs:
            matches = face_recognition.compare_faces(known_face_encodings, enc, tolerance=0.55)
            name = "Unknown"
            confidence = 0
            face_dists = face_recognition.face_distance(known_face_encodings, enc)
            idx = np.argmin(face_dists)
            if matches[idx]:
                name = known_face_names[idx]
                confidence = round((1 - face_dists[idx]) * 100, 1)
            names.append(name)
            dists.append(confidence)

        with result_lock:
            face_locations = locs
            face_names = names
            face_distances_list = dists

        processing = False

worker_thread = Thread(target=recognition_worker, daemon=True)
worker_thread.start()

# ── AUTO ZOOM TOWARD FACE ──
def compute_auto_zoom(locs, frame_w, frame_h, current_z):
    if not locs:
        # No face — slowly return to base zoom
        return max(1.5, current_z - 0.05)

    # Use largest face
    largest = max(locs, key=lambda b: (b[2]-b[0]) * (b[1]-b[3]))
    top, right, bottom, left = largest
    face_h = bottom - top
    face_w = right - left
    face_area = face_h * face_w
    frame_area = frame_h * frame_w

    ratio = face_area / frame_area

    # If face is too small, zoom in. If too large, zoom out.
    if ratio < 0.03:      # face very small — zoom in
        return min(current_z + 0.15, 6.0)
    elif ratio < 0.06:    # face small — zoom in gently
        return min(current_z + 0.05, 6.0)
    elif ratio > 0.25:    # face too large — zoom out
        return max(current_z - 0.1, 1.5)
    else:
        return current_z  # face is good size — hold zoom

# ── DRAW RESULTS ──
def draw_results(frame, scaler, scale):
    with result_lock:
        locs = list(face_locations)
        names = list(face_names)
        dists = list(face_distances_list)

    for (top, right, bottom, left), name, confidence in zip(locs, names, dists):
        top = int(top * scaler * scale)
        right = int(right * scaler * scale)
        bottom = int(bottom * scaler * scale)
        left = int(left * scaler * scale)

        if name == "Unknown":
            color = (60, 60, 255)
            label = "UNKNOWN"
        elif confidence >= 75:
            color = (0, 220, 80)
            label = f"{name.upper()}  {confidence}%"
        else:
            color = (0, 160, 255)
            label = f"{name.upper()}  {confidence}%"

        # Box
        cv2.rectangle(frame, (left, top), (right, bottom), color, 2)

        # Corner accents
        corner = 12
        thick = 2
        for (x, y, dx, dy) in [(left,top,1,1),(right,top,-1,1),(left,bottom,1,-1),(right,bottom,-1,-1)]:
            cv2.line(frame, (x, y), (x + dx*corner, y), color, thick+1)
            cv2.line(frame, (x, y), (x, y + dy*corner), color, thick+1)

        # Label background
        label_y = top - 10 if top > 30 else bottom + 25
        (lw, lh), _ = cv2.getTextSize(label, cv2.FONT_HERSHEY_SIMPLEX, 0.55, 1)
        cv2.rectangle(frame, (left, label_y - lh - 6), (left + lw + 8, label_y + 2), color, -1)
        cv2.putText(frame, label, (left + 4, label_y - 2), cv2.FONT_HERSHEY_SIMPLEX, 0.55, (0,0,0), 1, cv2.LINE_AA)

    return frame

# ── HUD ──
def draw_hud(frame, current_fps, zoom, scaler, skip, auto_zoom):
    h, w = frame.shape[:2]

    # Semi-transparent top bar
    overlay = frame.copy()
    cv2.rectangle(overlay, (0, 0), (w, 50), (0,0,0), -1)
    cv2.addWeighted(overlay, 0.5, frame, 0.5, 0, frame)

    # FPS
    fps_color = (0,220,80) if current_fps >= 8 else (0,160,255) if current_fps >= 4 else (60,60,255)
    cv2.putText(frame, f"FPS {current_fps:.0f}", (w-90, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, fps_color, 2, cv2.LINE_AA)

    # Zoom
    zoom_label = f"ZOOM  {zoom:.1f}x"
    cv2.putText(frame, zoom_label, (10, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0,220,80), 2, cv2.LINE_AA)

    # Auto zoom indicator
    az_color = (0,220,80) if auto_zoom else (100,100,100)
    az_label = "AUTO" if auto_zoom else "MANUAL"
    cv2.putText(frame, az_label, (180, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.55, az_color, 1, cv2.LINE_AA)

    # Face count
    with result_lock:
        known = sum(1 for n in face_names if n != "Unknown")
        unknown = sum(1 for n in face_names if n == "Unknown")

    count_label = f"DETECTED  {known+unknown}   KNOWN  {known}"
    cv2.putText(frame, count_label, (280, 32),
                cv2.FONT_HERSHEY_SIMPLEX, 0.5, (180,180,180), 1, cv2.LINE_AA)

    # Bottom control bar
    overlay2 = frame.copy()
    cv2.rectangle(overlay2, (0, h-36), (w, h), (0,0,0), -1)
    cv2.addWeighted(overlay2, 0.5, frame, 0.5, 0, frame)

    controls_text = "=  zoom in     -  zoom out     A  auto zoom     Q  quit"
    cv2.putText(frame, controls_text, (10, h-10),
                cv2.FONT_HERSHEY_SIMPLEX, 0.45, (150,150,150), 1, cv2.LINE_AA)

    return frame

# ── MAIN LOOP ──
print("[INFO] starting — = zoom in | - zoom out | A toggle auto zoom | Q quit")

apply_zoom(zoom_factor)

while True:
    # Display from lores — always smooth
    display_raw = picam2.capture_array("lores")
    display_raw = cv2.cvtColor(display_raw, cv2.COLOR_YUV420p2RGB)
    display_frame = cv2.resize(display_raw, (DISPLAY_W, DISPLAY_H), interpolation=cv2.INTER_LINEAR)

    # Trigger recognition thread when free
    frame_skip_count += 1
    if frame_skip_count % frame_skip == 0 and not processing:
        main_frame = picam2.capture_array("main")
        with main_frame_lock:
            latest_main_frame = main_frame
        processing = True
        frame_skip_count = 0

    # Auto zoom logic — smooth interpolation toward target
    if auto_zoom_enabled:
        with result_lock:
            locs = list(face_locations)
        scaler_h = DISPLAY_H / 720
        auto_zoom_target = compute_auto_zoom(locs, DISPLAY_W, DISPLAY_H, auto_zoom_current)
        # Smooth interpolation — avoids lag spikes from instant zoom changes
        auto_zoom_current += (auto_zoom_target - auto_zoom_current) * 0.08
        if abs(auto_zoom_current - zoom_factor) > 0.05:
            zoom_factor = round(auto_zoom_current, 2)
            cv_scaler = get_cv_scaler(zoom_factor)
            frame_skip = get_frame_skip(zoom_factor)
            apply_zoom(zoom_factor)  # no autofocus retrigger for smooth auto zoom

    display_scale = DISPLAY_W / 1280
    display_frame = draw_results(display_frame, cv_scaler, display_scale)

    current_fps = calculate_fps() if (frame_count := frame_count + 1) else 0
    elapsed = time.time() - start_time
    if elapsed > 1:
        fps = frame_count / elapsed
        frame_count = 0
        start_time = time.time()

    display_frame = draw_hud(display_frame, fps, zoom_factor, cv_scaler, frame_skip, auto_zoom_enabled)
    cv2.imshow("Face Recognition", display_frame)

    key = cv2.waitKey(1) & 0xFF

    if key == ord('='):
        auto_zoom_enabled = False
        auto_zoom_current = zoom_factor
        zoom_factor = min(zoom_factor + 0.5, 8.0)
        auto_zoom_current = zoom_factor
        cv_scaler = get_cv_scaler(zoom_factor)
        frame_skip = get_frame_skip(zoom_factor)
        update_zoom_full(zoom_factor)  # full zoom with AF retrigger only on manual
        print(f"Manual zoom: {zoom_factor}x")

    elif key == ord('-'):
        auto_zoom_enabled = False
        auto_zoom_current = zoom_factor
        zoom_factor = max(zoom_factor - 0.5, 1.0)
        auto_zoom_current = zoom_factor
        cv_scaler = get_cv_scaler(zoom_factor)
        frame_skip = get_frame_skip(zoom_factor)
        update_zoom_full(zoom_factor)
        print(f"Manual zoom: {zoom_factor}x")

    elif key == ord('a') or key == ord('A'):
        auto_zoom_enabled = not auto_zoom_enabled
        auto_zoom_current = zoom_factor
        print(f"Auto zoom: {'ON' if auto_zoom_enabled else 'OFF'}")

    elif key == ord('q'):
        break

cv2.destroyAllWindows()
picam2.stop()
```

---

### Model Training (model_training.py)
Processes every photo in the dataset, augments each image 6x, and builds the encodings.pickle file the recognition script loads.

```python
import gc
import os
import face_recognition
import pickle
import cv2
import numpy as np
from datetime import datetime

def list_images(path):
    extensions = [".jpg", ".jpeg", ".png", ".bmp", ".pgm"]
    for root, dirs, files in os.walk(path):
        for file in files:
            if os.path.splitext(file)[1].lower() in extensions:
                yield os.path.join(root, file)

def augment_image(image):
    augmented = [image]
   
    # Horizontal flip
    augmented.append(cv2.flip(image, 1))
   
    # Slightly brighter
    bright = cv2.convertScaleAbs(image, alpha=1.2, beta=20)
    augmented.append(bright)
   
    # Slightly darker
    dark = cv2.convertScaleAbs(image, alpha=0.8, beta=-20)
    augmented.append(dark)
   
    # Slight rotation left
    h, w = image.shape[:2]
    M = cv2.getRotationMatrix2D((w/2, h/2), 10, 1.0)
    rotated_left = cv2.warpAffine(image, M, (w, h))
    augmented.append(rotated_left)
   
    # Slight rotation right
    M = cv2.getRotationMatrix2D((w/2, h/2), -10, 1.0)
    rotated_right = cv2.warpAffine(image, M, (w, h))
    augmented.append(rotated_right)
   
    return augmented

print("=" * 50)
print("FACE RECOGNITION MODEL TRAINER")
print("=" * 50)
print(f"Started: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
print()

# Get all image paths
imagePaths = list(list_images("dataset"))

if len(imagePaths) == 0:
    print("[ERROR] No images found in dataset folder.")
    print("Make sure your dataset folder has subfolders named after each person.")
    exit()

# Count people and photos
people = {}
for imagePath in imagePaths:
    name = imagePath.split(os.path.sep)[-2]
    if name not in people:
        people[name] = 0
    people[name] += 1

print(f"[INFO] Found {len(people)} people and {len(imagePaths)} total images:")
for person, count in sorted(people.items()):
    status = "✓" if count >= 30 else "⚠ low"
    print(f"  {status} {person}: {count} photos")
print()

if any(count < 30 for count in people.values()):
    print("[WARNING] Some people have fewer than 30 photos — accuracy may be lower")
    print()

knownEncodings = []
knownNames = []
failed = []
total = len(imagePaths)

for (i, imagePath) in enumerate(imagePaths):
    name = imagePath.split(os.path.sep)[-2]
    print(f"[INFO] Processing {i + 1}/{total} — {name} — {os.path.basename(imagePath)}")
   
    # Read image
    image = cv2.imread(imagePath)
   
    if image is None:
        print(f"  [SKIP] Could not read image: {imagePath}")
        failed.append(imagePath)
        continue
   
    # Convert to RGB
    rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
   
    # Find faces using HOG
    boxes = face_recognition.face_locations(rgb, model="hog")


   
    if len(boxes) == 0:
        print(f"  [SKIP] No face found in {os.path.basename(imagePath)}")
        failed.append(imagePath)

        try:
            os.remove(imagePath)
            print(f"  [DELETE] Removed {os.path.basename(imagePath)}")
        except Exception as e:
            print(f"  [ERROR] Could not delete file: {e}")

        continue

   
    if len(boxes) > 1:
        print(f"  [WARN] Multiple faces found, using largest face only")
        # Use largest face (closest to camera)
        boxes = [max(boxes, key=lambda b: (b[2]-b[0]) * (b[1]-b[3]))]
   
    # Get encodings using large model for better accuracy
    encodings = face_recognition.face_encodings(rgb, boxes, model="large", num_jitters=1)
   
    # Augment image to improve training
    augmented_images = augment_image(rgb)
   
    for aug_image in augmented_images:
        aug_boxes = face_recognition.face_locations(aug_image, model="hog")
        if len(aug_boxes) > 0:
            aug_encodings = face_recognition.face_encodings(aug_image, aug_boxes, model="large", num_jitters=1)
            for encoding in aug_encodings:
                knownEncodings.append(encoding)
                knownNames.append(name)
   
    for encoding in encodings:
        knownEncodings.append(encoding)
        knownNames.append(name)
       
    gc.collect()
print()
print("=" * 50)
print("[INFO] Saving encodings...")
data = {"encodings": knownEncodings, "names": knownNames}
with open("encodings.pickle", "wb") as f:
    f.write(pickle.dumps(data))

print()
print("=" * 50)
print("TRAINING COMPLETE")
print("=" * 50)
print(f"Finished: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
print(f"Total encodings saved: {len(knownEncodings)}")
print(f"People trained: {len(people)}")
if failed:
    print(f"Failed/skipped images: {len(failed)}")
    for f in failed:
        print(f"  - {f}")
print("=" * 50)
```

---

### Headshot Capture (headshots_capture-picam.py)
Takes training photos at 4K resolution with live preview, autofocus triggering before each shot, and a 1-second cooldown to prevent accidental double captures.

```python
import cv2
import os
from datetime import datetime
from picamera2 import Picamera2
from libcamera import controls
import time

# Change this to the person's name
PERSON_NAME = "Ali"

# Camera capture resolution (saved images)
CAPTURE_SIZE = (3840, 2160)  # 4K

# Monitor preview resolution
DISPLAY_SIZE = (1024, 600)

def create_folder(name):
    dataset_folder = "dataset"
    if not os.path.exists(dataset_folder):
        os.makedirs(dataset_folder)
    person_folder = os.path.join(dataset_folder, name)
    if not os.path.exists(person_folder):
        os.makedirs(person_folder)
    return person_folder

def capture_photos(name):
    folder = create_folder(name)

    picam2 = Picamera2()

    config = picam2.create_preview_configuration(
        main={"size": CAPTURE_SIZE, "format": "RGB888"},
        lores={"size": (640, 480), "format": "YUV420"}
    )
    picam2.configure(config)

    picam2.start()

    # Enable continuous autofocus
    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Continuous,
        "AfSpeed": controls.AfSpeedEnum.Fast
    })

    time.sleep(2)  # Give autofocus time to lock on startup

    photo_count = 0
    last_photo_time = 0
    flash_frames = 0

    print(f"""
Capturing photos for: {name}
SPACE = take photo
Q     = quit
Tip: vary angles, lighting and distance for better accuracy
""")

    while True:
        # Grab low res preview stream directly
        preview = picam2.capture_array("lores")

        # Convert YUV420 to BGR for OpenCV display
        preview = cv2.cvtColor(preview, cv2.COLOR_YUV420p2BGR)
        preview = cv2.cvtColor(preview, cv2.COLOR_RGB2BGR)

        # Resize to exact display size
        preview = cv2.resize(preview, DISPLAY_SIZE, interpolation=cv2.INTER_AREA)

        h, w = preview.shape[:2]

        # Flash effect when photo is taken
        if flash_frames > 0:
            overlay = preview.copy()
            cv2.rectangle(overlay, (0, 0), (w, h), (255, 255, 255), -1)
            cv2.addWeighted(overlay, 0.3, preview, 0.7, 0, preview)
            flash_frames -= 1

        # Face guide box in center
        cv2.rectangle(preview, (w//3, h//4), (2*w//3, 3*h//4), (0, 255, 0), 2)
        cv2.putText(preview, "Align face in box", (w//3, h//4 - 10),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)

        # Photo count
        cv2.putText(preview, f"Photos: {photo_count}",
                    (20, 40), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)

        # Ready indicator
        cooldown = time.time() - last_photo_time
        if cooldown < 1.0:
            cv2.putText(preview, "Wait...",
                        (20, 80), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 165, 255), 2)
        else:
            cv2.putText(preview, "Ready",
                        (20, 80), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)

        # Target count reminder
        cv2.putText(preview, f"Target: 50 photos",
                    (20, h - 20), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 1)

        cv2.imshow("Headshot Capture", preview)
        key = cv2.waitKey(1) & 0xFF

        if key == ord(' '):
            if time.time() - last_photo_time < 1.0:
                continue

            photo_count += 1
            last_photo_time = time.time()
            flash_frames = 5

            # Trigger autofocus and wait for lock before capturing
            picam2.set_controls({"AfMode": controls.AfModeEnum.Auto,
                                  "AfTrigger": controls.AfTriggerEnum.Start})
            time.sleep(0.5)  # Wait for autofocus to lock

            # Switch back to continuous after capture
            picam2.set_controls({"AfMode": controls.AfModeEnum.Continuous})

            # Capture full resolution image
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            filename = f"{name}_{timestamp}.jpg"
            filepath = os.path.join(folder, filename)

            full_frame = picam2.capture_array("main")
            print("Frame format:", full_frame.dtype, full_frame.shape)
           
            cv2.imwrite(filepath, full_frame, [cv2.IMWRITE_JPEG_QUALITY, 95])
            print(f"Saved {photo_count}/50: {filepath}")

            if photo_count % 10 == 0:
                print(f"{photo_count} photos taken — try a different angle or lighting now")

        elif key == ord('q'):
            break

    cv2.destroyAllWindows()
    picam2.stop()
    print(f"Finished. {photo_count} photos saved for {name}.")
    if photo_count < 30:
        print(f"Warning: only {photo_count} photos — aim for 50 for best accuracy")

if __name__ == "__main__":
    capture_photos(PERSON_NAME)
```

---

## Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| 10.1" Security Monitor | Displays the live recognition feed | $78 | <a href="https://www.amazon.com/Haiway-Security-Surveillance-Controller-Resolution/dp/B07WKG9J35?th=1">Link</a> |
| Raspberry Pi 4 Starter Kit | Pi 4, micro SD, power supply, and case | $150 | <a href="https://www.amazon.com/CanaKit-Raspberry-4GB-Starter-Kit/dp/B07V5JTMV9">Link</a> |
| Arducam 64MP Hawkeye | 64MP autofocus camera for long-range detection | $50 | <a href="https://www.amazon.com/Raspberry-Pi-Camera-Module/dp/B0BRY6MVXL">Link</a> |
| Keyboard and Mouse | Controls the Pi | $15 | <a href="https://www.amazon.com/Logitech-Keyboard-Windows-Optical-Full-Size/dp/B003NREDC8">Link</a> |

---

## Other Resources/Examples

- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)
