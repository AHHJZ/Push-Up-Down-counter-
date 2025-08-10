# Push-Up-Down-counter-
YOLO11-based push-up counter — pose estimation with YOLO11-nano + a lightweight KNN classifier. Counts push-ups from webcam, video file, or RTSP stream.

YOLO Push‑Up Counter (yolo-main.py)

A lightweight push‑up counter that uses YOLO11-nano pose estimation to extract person keypoints, embeds them into a compact vector and classifies frame posture with a K‑Nearest Neighbors (KNN) model. A small state machine with geometric checks and debouncing turns frame-level predictions into robust push‑up counts.

This repository contains the inference script yolo-main.py, helper artifacts (model embeddings and KNN model) and a sample video. The code is designed to be run on a CPU or edge device (OpenVINO models supported by the Ultralytics wrapper).

Key features

Pose extraction via YOLO11-nano (pose model).

Compact keypoint embedding (centering & scale normalization).

KNN classifier trained on ~200 labeled images (two classes: pushups_down, pushups_up).

Robust push‑up counter state machine with:

minimum visible parts checks,

torso horizontal / size checks,

elbow‑angle verification,

temporal debouncing and a short cooldown between reps.

Two run modes:

Local webcam/video processing (GUI window)

Streaming server via FastAPI (/ endpoint returns MJPEG stream)


What you will find in this repo
/ (repo root)
├─ yolo-main.py                # main script (rename your file to this name)
├─ README.md                   # this file
├─ requirements.txt            # pip requirements
├─ models/
│  ├─ pose_embeddings_yolo11.pkl  # optional: precomputed embeddings (binary)
│  └─ knn_model_yolo11na.pkl      # optional: trained KNN model (binary)
├─ samples/
│  └─ sample_pushups.mp4        # optional: sample video demonstrating the model
├─ .gitattributes              # (recommended if using Git LFS)
├─ .gitignore
└─ LICENSE
