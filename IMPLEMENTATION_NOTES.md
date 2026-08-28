# Implementation notes and trade-offs

This document records repository changes made with Codex, including their purpose, validation, and important trade-offs. Update it whenever another implementation change is made.

## 2026-08-28: Bonsai dataset handling

### Changes

- Downloaded and verified the Mip-NeRF 360 Bonsai COLMAP dataset locally under `data/bonsai`.
- Added Bonsai dataset ignore rules to `.gitignore` while keeping `data/bonsai/README.md` trackable.
- Added `data/bonsai/README.md` with dataset download, verification, and fusion commands.

### Reason

The existing `bonsai` directory contains a trained Gaussian checkpoint and rendered predictions, but not the source photographs or COLMAP camera reconstruction required by the scene loader.

### Validation

- Confirmed 292 downloaded photographs and 292 registered COLMAP image records.
- Confirmed the required `cameras.bin`, `images.bin`, and `points3D.bin` files.
- Confirmed a supported PINHOLE camera model.
- Confirmed that all 37 checkpoint ground-truth frame names exist in the downloaded dataset.
- Confirmed Git ignores the downloaded data while exposing `data/bonsai/README.md` for tracking.

### Trade-offs

- The scene-only Hugging Face mirror avoids downloading Google's approximately 12.5 GB archive containing all Mip-NeRF 360 scenes, but it introduces a dependency on a third-party mirror.
- Keeping the dataset out of Git prevents a large repository and slow clones, but each user must download the dataset separately.
- The local verification used the 2× images at 1559×1039. The Colab workflow instead uses the 4× images at 779×519 to reduce memory use.

## 2026-08-28: Colab Bonsai fusion notebook

### Changes

- Replaced the mixed and partially broken `run.ipynb` workflow with an ordered Bonsai-only pipeline.
- Added an early GPU check so the notebook refuses to continue on a CPU server.
- Removed duplicate clone steps, malformed shell commands, stale outputs, and unrelated ScanNet preparation notes.
- Added restart-safe checks for the repository clone, Miniforge installation, Conda environment, and requirements.
- Ensured fusion runs through the pinned `sega` Conda environment rather than the notebook's base Python.
- Added checks for CUDA and the three compiled CUDA extensions before any fusion work begins.
- Added download and verification steps for the Bonsai dataset, iteration-30,000 Gaussian checkpoint, and OpenSeg SavedModel.
- Configured fusion to use `images_4`, a 779×519 fusion resolution, and two data-loader workers.
- Configured the notebook to copy fused `.pt` output into Google Drive.

### Reason

The previous notebook recorded a CPU runtime, invoked fusion with the wrong Python environment, contained broken and duplicate download commands, and did not make the untracked dataset or checkpoint available inside a fresh Colab server.

### Validation

- Parsed `run.ipynb` successfully as notebook-format JSON.
- Compiled all 11 Python code cells without syntax errors.
- Removed all saved cell outputs so old CPU results cannot be mistaken for current runtime results.
- Ran `git diff --check` successfully.

The complete workflow has not yet been executed end to end on a fresh Colab GPU server. Network downloads, Google authentication, available GPU memory, and upstream archives must therefore still be confirmed during the first live run.

### Trade-offs

- Using 4× images substantially reduces feature-map memory and runtime compared with 2× images, but loses some fine pixel-level semantic detail.
- Two data-loader workers reduce Colab CPU-memory pressure and worker instability, but can make image loading slower.
- The pinned Conda environment and compiled CUDA extensions improve repository compatibility, but rebuilding them on every fresh Colab server takes significant time.
- OpenSeg uses 768 semantic dimensions per Gaussian. Even at the lower image resolution, a free T4 GPU may run out of VRAM because the semantic tensor scales with the number of Gaussians.
- Automatic downloads improve reproducibility, but rely on the continued availability and unchanged structure of external Google Drive, Hugging Face, and CIIRC files.

## 2026-08-28: Persistent Bonsai files in Google Drive

### Changes

- Updated `run.ipynb` to store the dataset under `MyDrive/semantic-gaussians/data/bonsai`.
- Updated it to store the pretrained checkpoint under `MyDrive/semantic-gaussians/bonsai/checkpoint`.
- Kept fused output under `MyDrive/semantic-gaussians/outputs/bonsai/fused`.
- Changed the fusion arguments to read the dataset and checkpoint directly from these persistent Drive paths.

### Reason

Colab's `/content` filesystem is temporary. Removing or replacing a Colab server deletes local downloads, which would otherwise require downloading the Bonsai dataset and approximately 1.3 GB checkpoint again.

### Validation

- Revalidated notebook JSON and Python syntax after changing the paths.
- Confirmed the fusion command receives the persistent dataset and checkpoint paths.
- Confirmed there are no remaining temporary Bonsai dataset or checkpoint paths in the notebook.

### Trade-offs

- Persistent Drive storage avoids repeated large downloads and survives server removal.
- Reading images and a large point cloud through the mounted Drive filesystem is slower than reading from `/content`.
- Dataset and checkpoint files consume the user's Google Drive quota.
- The environment, compiled extensions, repository clone, and OpenSeg model remain temporary, so those setup steps still repeat on a completely new Colab server.

## Maintenance convention

For each future implementation change, add a dated section containing:

- Files and behavior changed.
- Reason for the change.
- Validation actually performed.
- Known limitations and trade-offs.
- Any required migration or rerun steps.
