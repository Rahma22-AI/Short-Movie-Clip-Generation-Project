# Final submission checklist (rubric mapping)

Use this to verify the project matches the course requirements before hand-in.

## Core project

- [ ] **Theme** is finalized in writing and reflected in prompts, demo, and evaluation prompts (`src/data/prompts.py` + report).
- [ ] **Text-to-video pipeline** runs end-to-end: train → checkpoint → generate MP4.
- [ ] **Temporal coherence** is addressed in code *and* in the report (what you did, why it helps, evidence).
- [ ] **Text–video alignment** is addressed in code *and* in the report (CFG, text encoder choice, qualitative + CLIP-SIM).

## Mathematics (paper)

- [ ] **DDPM forward/reverse** explained with intuition tied to your implementation.
- [ ] **Score** \(\nabla_x \log p(x)\): intuition + link to noise prediction.
- [ ] **ELBO → MSE** key idea (no need for full derivation if rubric says so).
- [ ] **Noise schedules**: compare at least two (e.g. linear vs cosine) and relate to stability/quality.
- [ ] **Video modeling**: why naive frame-wise generation is weak; what you use for temporal consistency.
- [ ] **Loss variants** \(\epsilon\), \(x_0\), \(v\): compare; state which you use (`diffusion.prediction_target`).
- [ ] **CFG**: mechanism, guidance scale trade-off (alignment vs diversity).
- [ ] **Sampling**: DDPM vs DDIM (and optional DPM-Solver) trade-offs; state what you use (`sample.sampler`).

## Architecture (paper Section 2)

- [ ] **3D U-Net** path documented (spatial/temporal down/up, residuals) — map to `src/models/video_diffusion.py`.
- [ ] **Attention**: spatial/temporal/cross (if cross not in code, explain gap or planned upgrade).
- [ ] **Temporal position encoding** (what you use: sinusoidal on \(t\) step vs frame index — clarify in report).
- [ ] **Conditioning**: text encoder (hash vs CLIP in `model.text_encoder`) and CFG dropout.
- [ ] **Latent vs pixel**: state you are in **pixel space** unless you add a VAE; if you add VAE later, update this section.

## Loss functions (paper Section 3)

- [ ] Full **mathematical form** for each variant you discuss.
- [ ] Justify **your** objective (`eps` / `x0` / `v`) with implementation reference.

## Weaknesses + fixes (paper Section 4) — highest weight

Document **at least two** weaknesses with evidence and mitigations. Suggested mapping to this repo:

1. **Temporal flicker / weak temporal consistency**  
   - Mitigation implemented: `train.temporal_smooth_weight` (penalizes large frame-to-frame changes in the prediction target).
2. **Weak semantic alignment from simple text embeddings**  
   - Mitigation implemented: optional **CLIP** text encoder (`model.text_encoder: clip`, `model.text_emb_dim: 512`).
3. (Optional third) **Limited architecture capacity** — ablate channels/attention; discuss in report.

For each weakness: observation → hypothesis → change → before/after (metrics and/or visuals).

## Evaluation (paper Section 5)

Run `python -m src.evaluate` with dependencies installed (see `requirements.txt`). Fill the table with **your** numbers.

| Metric | Status in repo | Notes |
|--------|----------------|-------|
| FVD | **Proxy** (`FVD_proxy`) | Inception-feature Fréchet between pred vs GT frame pools — cite as proxy; true FVD often uses I3D video features. |
| IS | Implemented (optional lib) | Needs `torchmetrics`. |
| CLIP-SIM | Implemented (optional lib) | Needs `open-clip-torch`; pass `--text_prompt` or use filename stem. |
| SSIM / PSNR | Implemented | `skimage` |
| LPIPS (temporal) | Implemented (optional lib) | Needs `lpips` |
| Flow warping error | Optional | Uses torchvision RAFT if available; else skipped. |
| User study | **You run** | ≥10 raters, 1–5 scale; store raw CSV + summary stats. |

## Bonus: text-to-audio

Implemented with **Edge-TTS** (Microsoft neural voices, no API key). Meets **≥2** rubric-style features:

- [x] **Neural TTS** (Edge neural voices)
- [x] **Multi-voice** (`audio.voice` / `--voice`, e.g. `en-US-AriaNeural`, `en-US-GuyNeural`)
- [x] **Emotion-controlled prosody** (`emotion`: neutral / happy / excited / sad / calm → rate/pitch/volume)
- [x] **Adjustable parameters** (`rate`, `pitch`, `volume` in YAML or CLI)
- [x] **Chunk-based synthesis** for long scripts (sentence-aware splitting + ffmpeg concat)
- [x] **Video integration**: `generate.py` writes `*_narrated.mp4` or use `python -m src.add_narration`

Requires **network** during synthesis (Edge endpoint) and **ffmpeg** (bundled via `imageio-ffmpeg` or system PATH).

## Artifacts to submit

- [ ] Final PDF report (from `paper_template.md` or LaTeX).
- [ ] Code zip / repo + `README` run instructions.
- [ ] Checkpoint(s) or training config to reproduce best run.
- [ ] Generated demo videos + evaluation CSV/JSON from `src/evaluate.py`.
- [ ] Ablation outputs: `outputs/ablation/*.csv` from `src.ablate_targets`.

## Quick commands

```bash
pip install -r requirements.txt
python -m src.train --config configs/base.yaml
python -m src.generate --config configs/base.yaml --checkpoint checkpoints/epoch_5.pt --prompt "your prompt"
python -m src.evaluate --pred_dir outputs/generated --gt_dir data/reference --text_prompt "your prompt"
python -m src.ablate_targets --config configs/base.yaml
```
