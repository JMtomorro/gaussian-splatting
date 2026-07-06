# 2026-07-06 T&T Train 3DGS Smoke Test 1000 Iterations

## Goal
This smoke test verifies that the Docker-based 3D Gaussian Splatting environment can complete the official train-render-metrics pipeline on a real COLMAP dataset.
This is an environment validation run, not a paper-quality reconstruction run.

## Environment
- Repository: JMtomorro/gaussian-splatting
- Local path: /home/sjm/Habitat/project/gaussian-splatting
- Docker image: gaussian-splatting:cu128
- Base image: pytorch/pytorch:2.7.1-cuda12.8-cudnn9-devel
- GPU: NVIDIA RTX PRO 6000 Blackwell Max-Q Workstation Edition
- PyTorch: 2.7.1+cu128
- CUDA runtime: 12.8

## Dataset
- Dataset archive: tandt_db.zip
- Container dataset root: /data
- Detected scenes:
  - /data/db/drjohnson/sparse
  - /data/db/playroom/sparse
  - /data/tandt/train/sparse
  - /data/tandt/truck/sparse
- Smoke-test scene: /data/tandt/train

## Training
Command:
```bash
python train.py -s /data/tandt/train -m output/tandt_train_smoke --eval --iterations 1000 --test_iterations 1000 --save_iterations 1000 --disable_viewer
```

Key results:
- Cameras: 301
- Initial points: 182686
- Iterations: 1000
- Runtime: about 16 seconds
- Speed: about 62.32 it/s
- Final log loss: 0.1743759
- Test PSNR during training: 17.271107924611943
- Train PSNR during training: 17.734425735473632

## Rendering
Command:
```bash
python render.py -m output/tandt_train_smoke
```
- Loaded model iteration: 1000
- Rendered train views: 263
- Rendered test views: 38

## Metrics
Command:
```bash
python metrics.py -m output/tandt_train_smoke
```

Final metrics:
```json
{
 "ours_1000": {
  "SSIM": 0.5886368155479431,
  "PSNR": 17.25019645690918,
  "LPIPS": 0.48269110918045044
 }
}```

Additional notes:
- VGG16 checkpoint downloaded: vgg16-397923af.pth
- VGG16 checkpoint size: 553433881 bytes
- LPIPS weight downloaded: vgg.pth

## Output Paths
- Inside container: /workspace/gaussian-splatting/output/tandt_train_smoke
- On host: /home/sjm/Habitat/project/gaussian-splatting/output/tandt_train_smoke

Important outputs:
- output/tandt_train_smoke/results.json
- output/tandt_train_smoke/per_view.json
- output/tandt_train_smoke/point_cloud/iteration_1000/point_cloud.ply
- output/tandt_train_smoke/train/ours_1000/renders/
- output/tandt_train_smoke/train/ours_1000/gt/
- output/tandt_train_smoke/test/ours_1000/renders/
- output/tandt_train_smoke/test/ours_1000/gt/

## Interpretation
This smoke test confirms that the Docker environment can complete the official 3DGS train-render-metrics loop on a real COLMAP dataset.
The visual quality is not expected to be high because only 1000 iterations were used.

Recommended next steps:
1. run 7000 iterations for a medium-quality sanity check;
2. run 30000 iterations for a formal reproduction-style experiment;
3. record runtime, metrics, output size, and visual quality;
4. then return to VLFM / 3DGSNav integration.
