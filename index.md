---
layout: default
title: Facial Recognition System
---
# Facial Recognition System

This project uses a Raspberry Pi to build a real-time facial recognition system capable of identifying people at distance using a 64MP camera, digital zoom, and machine learning. The system is trained on a custom image dataset to recognize specific individuals through a connected camera with live confidence scoring.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Aarush H | Evergreen Valley High School | Computer Engineering | Incoming Senior |

![Headstone Image](Aarush%20H.jpeg)

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

- Trained the model on images stored in the dataset using Python — the live feed now displays people's names in real time
- Surprised by how much data the model actually needs — 200 pictures across 4 people wasn't enough, the model still struggled beyond 3 feet
- The live feed was extremely zoomed in on first run and hard to control — fixed it by adjusting resolution to fit the monitor, then tackled accuracy by adding more people to the dataset and tightening the model's tolerance
- Next goal: detect a person from across the classroom, approximately 10 meters

---

## First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/wKA9XhxVHsE?si=93JbfTmWjfTUcsn5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

- Set up OpenCV and Python scripts that capture photos on spacebar press and organize them into named folders so the model can associate names with faces
- The camera quality was worse than expected — only looked sharp when the subject was directly in front of it at close range
- Some captured images became corrupted silently — had to mass-delete batches because there was no way to tell which ones had gone bad without checking each individually
- Next goal: detect myself and at least one other person reliably

---

## Starter Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/wKA9XhxVHsE?si=93JbfTmWjfTUcsn5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Built and soldered a retro arcade device — connected a battery to metal contacts on the back, soldered the side charging port and control buttons (movement and reset), then assembled the acrylic case with screws, improvising where hardware was missing.

---
<!--
## Schematics

This is a software-heavy project, so traditional circuit diagrams are minimal. Here's what's worth documenting visually:

**System Architecture Diagram**
A block diagram showing how data flows through the system: Camera → Picamera2 → OpenCV frame pipeline → face_recognition (HOG detection → encoding → comparison) → display output. Shows the dual-stream setup (lores for display, main for recognition) and where zoom/ScalerCrop fits in.

**Camera Connection Diagram**
The Arducam 64MP Hawkeye connects via CSI ribbon cable to the Raspberry Pi 4's camera port. Worth showing the physical connection and the ribbon cable orientation since it's easy to insert backwards.

**Component Layout**
A simple labeled photo or diagram showing: Raspberry Pi 4, Arducam 64MP Hawkeye on CSI port, monitor via HDMI, keyboard/mouse via USB, power supply. No breadboard or wiring complexity — this project's hardware is all plug-and-play, the complexity lives in the software.

---
-->
## Code

### Face Recognition (face_rec-picam.py)
The main script. Runs continuous facial recognition on the Pi Camera feed with dynamic zoom, autofocus control, and live confidence scoring.

```python
import face_recognition
import cv2
import numpy as np
from picamera2 import Picamera2
from libcamera import controls
import time
import pickle

print("[INFO] loading encodings...")
with open("encodings.pickle", "rb") as f:
    data = pickle.loads(f.read())
known_face_encodings = data["encodings"]
known_face_names = data["names"]

print(f"[INFO] loaded {len(set(known_face_names))} people, {len(known_face_names)} total encodings")

def get_cv_scaler(zoom_factor):
    if zoom_factor >= 6.0:
        return 1
    elif zoom_factor >= 4.0:
        return 2
    elif zoom_factor >= 2.0:
        return 3
    else:
        return 4

def get_frame_skip(zoom_factor):
    if zoom_factor >= 4.0:
        return 12
    elif zoom_factor >= 2.0:
        return 8
    else:
        return 6

def update_zoom(picam2, zoom_factor):
    size = picam2.camera_properties['PixelArraySize']
    full_width, full_height = size
    crop_width = int(full_width / zoom_factor)
    crop_height = int(full_height / zoom_factor)
    crop_x = (full_width - crop_width) // 2
    crop_y = (full_height - crop_height) // 2
    picam2.set_controls({
        "ScalerCrop": (crop_x, crop_y, crop_width, crop_height),
        "AfMode": controls.AfModeEnum.Auto,
        "AfTrigger": controls.AfTriggerEnum.Start,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })
    time.sleep(0.8)
    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Continuous,
        "AfSpeed": controls.AfSpeedEnum.Fast,
        "AfRange": controls.AfRangeEnum.Full
    })
    return zoom_factor

picam2 = Picamera2()
config = picam2.create_preview_configuration(
    main={"size": (1920, 1080), "format": "RGB888"},
    lores={"size": (640, 480), "format": "YUV420"}
)
picam2.configure(config)
picam2.start()

picam2.set_controls({
    "AfMode": controls.AfModeEnum.Auto,
    "AfTrigger": controls.AfTriggerEnum.Start,
    "AfSpeed": controls.AfSpeedEnum.Fast,
    "AfRange": controls.AfRangeEnum.Full
})

time.sleep(3)

picam2.set_controls({
    "AfMode": controls.AfModeEnum.Continuous,
    "AfSpeed": controls.AfSpeedEnum.Fast,
    "AfRange": controls.AfRangeEnum.Full
})

zoom_factor = 2.0
update_zoom(picam2, zoom_factor)
cv_scaler = get_cv_scaler(zoom_factor)
frame_skip = get_frame_skip(zoom_factor)

time.sleep(1)

face_locations = []
face_encodings_list = []
face_names = []
face_distances_list = []
frame_count = 0
start_time = time.time()
fps = 0
frame_skip_count = 0
last_known_display = None

def process_frame(frame):
    global face_locations, face_encodings_list, face_names, face_distances_list
    resized_frame = cv2.resize(frame, (0, 0), fx=(1/cv_scaler), fy=(1/cv_scaler))
    rgb_resized_frame = cv2.cvtColor(resized_frame, cv2.COLOR_BGR2RGB)
    face_locations = face_recognition.face_locations(rgb_resized_frame, model="hog", number_of_times_to_upsample=1)
    if len(face_locations) == 0 and zoom_factor >= 3.0:
        face_locations = face_recognition.face_locations(rgb_resized_frame, model="hog", number_of_times_to_upsample=2)
    face_encodings_list = face_recognition.face_encodings(rgb_resized_frame, face_locations, model="large")
    face_names = []
    face_distances_list = []
    for face_encoding in face_encodings_list:
        matches = face_recognition.compare_faces(known_face_encodings, face_encoding, tolerance=0.5)
        name = "Unknown"
        confidence = 0
        face_distances = face_recognition.face_distance(known_face_encodings, face_encoding)
        best_match_index = np.argmin(face_distances)
        if matches[best_match_index]:
            name = known_face_names[best_match_index]
            confidence = round((1 - face_distances[best_match_index]) * 100, 1)
        face_names.append(name)
        face_distances_list.append(confidence)

def draw_results(frame, display_scale):
    for (top, right, bottom, left), name, confidence in zip(face_locations, face_names, face_distances_list):
        top = int(top * cv_scaler * display_scale)
        right = int(right * cv_scaler * display_scale)
        bottom = int(bottom * cv_scaler * display_scale)
        left = int(left * cv_scaler * display_scale)
        if name == "Unknown":
            color = (0, 0, 255)
        elif confidence >= 70:
            color = (0, 255, 0)
        else:
            color = (0, 165, 255)
        cv2.rectangle(frame, (left, top), (right, bottom), color, 2)
        cv2.rectangle(frame, (left - 3, top - 45), (right + 3, top), color, cv2.FILLED)
        font = cv2.FONT_HERSHEY_DUPLEX
        cv2.putText(frame, name, (left + 6, top - 26), font, 0.7, (255, 255, 255), 1)
        if name != "Unknown":
            cv2.putText(frame, f"{confidence}%", (left + 6, top - 8), font, 0.5, (255, 255, 255), 1)
    return frame

def calculate_fps():
    global frame_count, start_time, fps
    frame_count += 1
    elapsed_time = time.time() - start_time
    if elapsed_time > 1:
        fps = frame_count / elapsed_time
        frame_count = 0
        start_time = time.time()
    return fps

def draw_hud(frame, current_fps, zoom_factor, cv_scaler, frame_skip):
    h, w = frame.shape[:2]
    fps_color = (0, 255, 0) if current_fps >= 5 else (0, 165, 255) if current_fps >= 3 else (0, 0, 255)
    cv2.putText(frame, f"FPS: {current_fps:.1f}", (w - 150, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.8, fps_color, 2)
    cv2.putText(frame, f"Zoom: {zoom_factor}x", (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 0), 2)
    cv2.putText(frame, f"Detail: {cv_scaler} | Skip: {frame_skip}", (10, 60), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)
    known_count = sum(1 for n in face_names if n != "Unknown")
    unknown_count = sum(1 for n in face_names if n == "Unknown")
    cv2.putText(frame, f"Known: {known_count} Unknown: {unknown_count}", (10, 90), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)
    cv2.putText(frame, "= zoom in | - zoom out | q quit", (10, h - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1)
    return frame

print("[INFO] starting face recognition...")
print("Controls: = zoom in | - zoom out | q quit")

while True:
    display_raw = picam2.capture_array("lores")
    display_raw = cv2.cvtColor(display_raw, cv2.COLOR_YUV420p2RGB)
    display_frame = cv2.resize(display_raw, (1280, 720), interpolation=cv2.INTER_AREA)
    frame_skip_count += 1
    if frame_skip_count % frame_skip == 0:
        main_frame = picam2.capture_array("main")
        process_frame(main_frame)
        frame_skip_count = 0
    display_scale = 1280 / 1920
    display_frame = draw_results(display_frame, display_scale)
    current_fps = calculate_fps()
    display_frame = draw_hud(display_frame, current_fps, zoom_factor, cv_scaler, frame_skip)
    cv2.imshow("Face Recognition", display_frame)
    key = cv2.waitKey(1) & 0xFF
    if key == ord('='):
        zoom_factor = min(zoom_factor + 0.5, 8.0)
        update_zoom(picam2, zoom_factor)
        cv_scaler = get_cv_scaler(zoom_factor)
        frame_skip = get_frame_skip(zoom_factor)
        print(f"Zoom: {zoom_factor}x | Detail: {cv_scaler} | Skip: {frame_skip}")
    elif key == ord('-'):
        zoom_factor = max(zoom_factor - 0.5, 1.0)
        update_zoom(picam2, zoom_factor)
        cv_scaler = get_cv_scaler(zoom_factor)
        frame_skip = get_frame_skip(zoom_factor)
        print(f"Zoom: {zoom_factor}x | Detail: {cv_scaler} | Skip: {frame_skip}")
    elif key == ord('q'):
        break

cv2.destroyAllWindows()
picam2.stop()
```

---

### Model Training (model_training.py)
Processes every photo in the dataset, augments each image 6x, and builds the encodings.pickle file the recognition script loads.

```python
import os
import face_recognition
import pickle
import cv2
import numpy as np
from datetime import datetime

def list_images(path):
    extensions = [".jpg", ".jpeg", ".png", ".bmp"]
    for root, dirs, files in os.walk(path):
        for file in files:
            if os.path.splitext(file)[1].lower() in extensions:
                yield os.path.join(root, file)

def augment_image(image):
    augmented = [image]
    augmented.append(cv2.flip(image, 1))
    bright = cv2.convertScaleAbs(image, alpha=1.2, beta=20)
    augmented.append(bright)
    dark = cv2.convertScaleAbs(image, alpha=0.8, beta=-20)
    augmented.append(dark)
    h, w = image.shape[:2]
    M = cv2.getRotationMatrix2D((w/2, h/2), 10, 1.0)
    rotated_left = cv2.warpAffine(image, M, (w, h))
    augmented.append(rotated_left)
    M = cv2.getRotationMatrix2D((w/2, h/2), -10, 1.0)
    rotated_right = cv2.warpAffine(image, M, (w, h))
    augmented.append(rotated_right)
    return augmented

print("=" * 50)
print("FACE RECOGNITION MODEL TRAINER")
print("=" * 50)
print(f"Started: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
print()

imagePaths = list(list_images("dataset"))

if len(imagePaths) == 0:
    print("[ERROR] No images found in dataset folder.")
    exit()

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

knownEncodings = []
knownNames = []
failed = []
total = len(imagePaths)

for (i, imagePath) in enumerate(imagePaths):
    name = imagePath.split(os.path.sep)[-2]
    print(f"[INFO] Processing {i + 1}/{total} — {name} — {os.path.basename(imagePath)}")
    image = cv2.imread(imagePath)
    if image is None:
        print(f"  [SKIP] Could not read image: {imagePath}")
        failed.append(imagePath)
        continue
    rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    boxes = face_recognition.face_locations(rgb, model="hog")
    if len(boxes) == 0:
        print(f"  [RETRY] HOG found no face, trying CNN...")
        boxes = face_recognition.face_locations(rgb, model="cnn")
    if len(boxes) == 0:
        print(f"  [SKIP] No face found in {os.path.basename(imagePath)}")
        failed.append(imagePath)
        continue
    if len(boxes) > 1:
        boxes = [max(boxes, key=lambda b: (b[2]-b[0]) * (b[1]-b[3]))]
    encodings = face_recognition.face_encodings(rgb, boxes, model="large", num_jitters=3)
    augmented_images = augment_image(rgb)
    for aug_image in augmented_images:
        aug_boxes = face_recognition.face_locations(aug_image, model="hog")
        if len(aug_boxes) > 0:
            aug_encodings = face_recognition.face_encodings(aug_image, aug_boxes, model="large", num_jitters=3)
            for encoding in aug_encodings:
                knownEncodings.append(encoding)
                knownNames.append(name)
    for encoding in encodings:
        knownEncodings.append(encoding)
        knownNames.append(name)

print()
print("=" * 50)
print("[INFO] Saving encodings...")
data = {"encodings": knownEncodings, "names": knownNames}
with open("encodings.pickle", "wb") as f:
    f.write(pickle.dumps(data))

print()
print("TRAINING COMPLETE")
print(f"Finished: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
print(f"Total encodings saved: {len(knownEncodings)}")
print(f"People trained: {len(people)}")
if failed:
    print(f"Failed/skipped: {len(failed)}")
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

PERSON_NAME = "Name"
CAPTURE_SIZE = (3840, 2160)
DISPLAY_SIZE = (1280, 720)

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
    picam2.set_controls({
        "AfMode": controls.AfModeEnum.Continuous,
        "AfSpeed": controls.AfSpeedEnum.Fast
    })
    time.sleep(2)
    photo_count = 0
    last_photo_time = 0
    flash_frames = 0
    print(f"Capturing photos for: {name}\nSPACE = take photo | Q = quit")

    while True:
        preview = picam2.capture_array("lores")
        preview = cv2.cvtColor(preview, cv2.COLOR_YUV420p2BGR)
        preview = cv2.resize(preview, DISPLAY_SIZE, interpolation=cv2.INTER_AREA)
        h, w = preview.shape[:2]
        if flash_frames > 0:
            overlay = preview.copy()
            cv2.rectangle(overlay, (0, 0), (w, h), (255, 255, 255), -1)
            cv2.addWeighted(overlay, 0.3, preview, 0.7, 0, preview)
            flash_frames -= 1
        cv2.rectangle(preview, (w//3, h//4), (2*w//3, 3*h//4), (0, 255, 0), 2)
        cv2.putText(preview, "Align face in box", (w//3, h//4 - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)
        cv2.putText(preview, f"Photos: {photo_count}", (20, 40), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
        cooldown = time.time() - last_photo_time
        status = "Wait..." if cooldown < 1.0 else "Ready"
        color = (0, 165, 255) if cooldown < 1.0 else (0, 255, 0)
        cv2.putText(preview, status, (20, 80), cv2.FONT_HERSHEY_SIMPLEX, 0.7, color, 2)
        cv2.putText(preview, "Target: 50 photos", (20, h - 20), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 1)
        cv2.imshow("Headshot Capture", preview)
        key = cv2.waitKey(1) & 0xFF
        if key == ord(' '):
            if time.time() - last_photo_time < 1.0:
                continue
            photo_count += 1
            last_photo_time = time.time()
            flash_frames = 5
            picam2.set_controls({"AfMode": controls.AfModeEnum.Auto, "AfTrigger": controls.AfTriggerEnum.Start})
            time.sleep(0.5)
            picam2.set_controls({"AfMode": controls.AfModeEnum.Continuous})
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            filepath = os.path.join(folder, f"{name}_{timestamp}.jpg")
            full_frame = picam2.capture_array("main")
            cv2.imwrite(filepath, full_frame, [cv2.IMWRITE_JPEG_QUALITY, 95])
            print(f"Saved {photo_count}/50: {filepath}")
            if photo_count % 10 == 0:
                print(f"{photo_count} photos — try a different angle now")
        elif key == ord('q'):
            break

    cv2.destroyAllWindows()
    picam2.stop()
    print(f"Done. {photo_count} photos saved.")
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
