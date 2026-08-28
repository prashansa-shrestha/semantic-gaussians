# Mip-NeRF 360 Bonsai dataset

This directory contains the downsampled Bonsai images and COLMAP reconstruction used with the existing checkpoint in `../../bonsai/checkpoint`.

Run all commands from the repository root.

## 1. Install the Hugging Face CLI

```bash
pip install -U huggingface_hub
```

This installs the `hf` command used to download files from Hugging Face.

## 2. Download the dataset metadata

```bash
hf download rishitdagli/nerf-gs-datasets \
  bonsai/nb-info.json \
  --repo-type dataset \
  --local-dir data
```

The metadata identifies the scene as Mip-NeRF 360 Bonsai and specifies the 2x-downsampled images.

## 3. Download the images

```bash
hf download rishitdagli/nerf-gs-datasets \
  --repo-type dataset \
  --include 'bonsai/images_2/*' \
  --local-dir data
```

This downloads the 292 source photographs into `data/bonsai/images_2`.

## 4. Download the COLMAP reconstruction

```bash
hf download rishitdagli/nerf-gs-datasets \
  --repo-type dataset \
  --include 'bonsai/sparse/0/*' \
  --local-dir data
```

This downloads the camera parameters, camera poses, and sparse point cloud required by the scene loader.

## 5. Verify the required files

```bash
find data/bonsai/images_2 -maxdepth 1 -type f | wc -l
find data/bonsai/sparse/0 -maxdepth 1 -type f -printf '%f\n'
```

The first command should print `292`. The second should list `cameras.bin`, `images.bin`, and `points3D.bin`.

## 6. Run semantic fusion

```bash
python fusion.py \
  scene.scene_path=./data/bonsai \
  scene.colmap_images=images_2 \
  model.model_dir=./bonsai/checkpoint \
  model.load_iteration=30000 \
  fusion.img_dim='[1559,1039]' \
  fusion.out_dir=./bonsai/fused
```

This loads the Bonsai scene and the existing iteration-30,000 Gaussian checkpoint, then saves the projected semantic features under `bonsai/fused`.
