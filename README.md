# README.md

# YOLO Push‑Up Counter (yolo-main.py)

A lightweight push‑up counter that uses **YOLO11-nano** pose estimation to extract person keypoints, embeds them into a compact vector and classifies frame posture with a **K‑Nearest Neighbors (KNN)** model. A small state machine with geometric checks and debouncing turns frame-level predictions into robust push‑up counts.

> This repository contains the inference script `yolo-main.py`, helper artifacts (model embeddings and KNN model) and a sample video. The code is designed to be run on a CPU or edge device (OpenVINO models supported by the Ultralytics wrapper).

## Key features

* Pose extraction via YOLO11-nano (pose model).
* Compact keypoint embedding (centering & scale normalization).
* KNN classifier trained on \~200 labeled images (two classes: `pushups_down`, `pushups_up`).
* Robust push‑up counter state machine with:

  * minimum visible parts checks,
  * torso horizontal / size checks,
  * elbow‑angle verification,
  * temporal debouncing and a short cooldown between reps.
* Two run modes:

  * Local webcam/video processing (GUI window)
  * Streaming server via FastAPI (`/` endpoint returns MJPEG stream)

## What you will find in this repo

```
/ (repo root)
├─ yolo-main.py                # main script 
├─ README.md                   # this file
├─ requirements.txt            # pip requirements
├─ models/
│  ├─ pose_embeddings_yolo11.pkl  #  precomputed embeddings (binary)
│  └─ knn_model_yolo11na.pkl      #  trained KNN model (binary)
├─ samples/
│  └─ sample_pushups.mp4        #  sample video demonstrating the model
├─ .gitattributes              # (recommended if using Git LFS)
└─.gitignore
```

## Quickstart

1. Clone the repository:

```bash
git clone https://github.com/AHHJZ/Push-Up-Down-counter.git
cd Push-Up-Down-counter
```

2. (Recommended) create and activate a virtual environment:

```bash
python -m venv venv
# macOS / Linux
source venv/bin/activate
# Windows (PowerShell)
venv\Scripts\Activate.ps1
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Put the two model files in `models/` (or download them from the Releases page):

* `models/pose_embeddings_yolo11.pkl`
* `models/knn_model_yolo11na.pkl`

> **If you include the binary files in the repo:** prefer using **Git LFS** for `.pkl` and large video files. If you prefer not to store them in the repo, upload them to a GitHub Release and link/download them in the README.

5. Configure input source:

* By default the script uses `RTSP_URL = 0` which opens the default webcam.
* Replace `0` with a local video path (e.g. `samples/sample_pushups.mp4`) or your RTSP URL (e.g. `rtsp://...`) before running, or edit the top-level `RTSP_URL` constant in `yolo-main.py`.

6. Run the script (webcam GUI mode):

```bash
python yolo-main.py
```

7. (Alternative) run the FastAPI MJPEG streaming server:

```bash
uvicorn yolo-main:app --host 0.0.0.0 --port 8000
# then open http://localhost:8000/ in your browser
```

> Note: `uvicorn` spawns the ASGI server and imports the module, so the `if __name__ == "__main__"` block will not execute. Use the `python yolo-main.py` path for local GUI mode.

## How the model was built (short)

* Pose estimation: **YOLO11-nano** (Ultralytics wrapper) to extract keypoints per person.
* Embedding: normalized XY coordinates (centroid + scale normalization), flattened and padded to a fixed length (default 17 keypoints => 34 values).
* Classifier: **KNeighborsClassifier** trained on \~200 labeled images across two classes (`pushups_down`, `pushups_up`).
* The push‑up counter uses the KNN frame-level label plus geometric checks (torso horizontal + elbow angle) and a short cooldown to avoid double counting.

## Troubleshooting & tips

* If the camera can't open, verify `RTSP_URL` and camera permissions.
* If the YOLO model path is wrong, set `YOLO_MODEL_PATH` to the correct directory or model file. Ultralytics accepts local model paths and weights.
* If KNN or embedding files are missing, the script will fail when trying to open them — place them in `models/` or adjust paths.
* For better accuracy: increase the number of labeled images or use a small neural classifier (if you later want to swap KNN with a small MLP).

## License

Choose a license (MIT recommended). Include a `LICENSE` file.

---

# requirements.txt (suggested)

```
# Minimal recommended Python version: 3.8+
ultralytics>=8.0
opencv-python>=4.5
numpy>=1.21
scikit-learn>=1.0
fastapi>=0.85
uvicorn[standard]>=0.18
```

> Note: exact versions depend on your environment. If you rely on a system OpenCV (e.g. OpenVINO), adjust accordingly.

---

# .gitattributes (recommended if you use Git LFS)

```
# Use Git LFS for binary models & sample video
*.pkl filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
```

# .gitignore (suggested)

```
venv/
__pycache__/
*.pyc
.DS_Store
.env
```

---

# Where to put the weight files and the sample video

**Option A — add to repo with Git LFS (recommended for reproducibility):**

1. Install Git LFS locally:

```bash
# install git-lfs (system/package manager) then:
git lfs install
```

2. Track file types and add files:

```bash
git lfs track "*.pkl"
git lfs track "*.mp4"
# make sure .gitattributes is added
git add .gitattributes
# create a models/ folder and place the files there
mkdir -p models samples
cp /path/to/pose_embeddings_yolo11.pkl models/
cp /path/to/knn_model_yolo11na.pkl models/
cp /path/to/sample_pushups.mp4 samples/

git add models/ samples/
git commit -m "Add model weights and sample video (tracked by LFS)"
git push origin main
```

**Option B — attach to a GitHub Release (recommended if you don't want LFS):**

1. Push code without binaries to the repository.
2. Create a Release in GitHub UI (or use `gh` CLI) and upload the `.pkl` and `.mp4` as release assets.

Using `gh` (GitHub CLI):

```
gh release create v1.0 models/pose_embeddings_yolo11.pkl models/knn_model_yolo11na.pkl samples/sample_pushups.mp4 \
  --title "v1.0 - model + sample" --notes "Trained embeddings and KNN model + sample video"
```

After attaching assets to a release, users can download them and place them into `models/` and `samples/`.

---

# Example: Full local flow (create repo + upload using Git LFS)

```bash
# 1) Create repo on GitHub (web UI) then:
git clone https://github.com/<YOU>/<REPO>.git
cd <REPO>
# 2) Add files
mkdir models samples
# copy your precomputed files into models/ and samples/
# 3) Install & configure LFS
git lfs install
git lfs track "*.pkl"
git lfs track "*.mp4"
git add .gitattributes
# 4) stage & commit
git add yolo-main.py README.md requirements.txt models samples .gitignore
git commit -m "Initial commit: core script, readme, requirements, models"
# 5) push
git push origin main
```

If you prefer not to use LFS, skip the `git lfs` steps and instead upload the binaries under a GitHub Release.



---


## Author
Amirhosein Hajizadeh  
📧 amirhoseinhajizadeh90@gmail.com


---

