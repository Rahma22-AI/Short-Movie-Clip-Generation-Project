# Short Movie Clip Generation (Diffusion)

Starter project for a text-to-video generation pipeline with focus on:
- temporal coherence
- text-video semantic alignment
- clear mathematical and architectural documentation

## Project Structure

- `configs/base.yaml` - training, sampling, and model hyperparameters
- `src/models/video_diffusion.py` - compact 3D U-Net style denoiser
- `src/data/video_dataset.py` - real video loader (mp4/gif/mov/avi/mkv) with prompt extraction
- `src/train.py` - training loop with configurable prediction target (`eps`/`x0`/`v`)
- `src/generate.py` - prompt-to-video sampling script with CFG (DDPM or DDIM)
- `src/ablate_targets.py` - auto-runs objective ablation (`eps`, `x0`, `v`) and exports JSON/CSV
- `src/plot_ablation.py` - bar chart from ablation CSV
- `SUBMISSION_CHECKLIST.md` - rubric mapping and what is done vs. left for the report
- `src/evaluate.py` - evaluation entrypoint (FVD/IS/CLIP-SIM placeholders + basic metrics)
- `src/data/prompts.py` - theme-driven prompt set
- `paper_template.md` - report structure mapped to your rubric
- `src/audio/` - **bonus**: Edge-TTS narration + ffmpeg mux (`generate.py --narration`, `src/add_narration.py`)

## Quick Start

1) Create environment

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

2) Train baseline model

```bash
python -m src.train --config configs/base.yaml
```

3) Generate sample clips

```bash
python -m src.generate --config configs/base.yaml --prompt "A neon cyberpunk city with flying cars at dusk"
```

4) Run evaluation

```bash
python -m src.evaluate --pred_dir outputs/generated --gt_dir data/reference --text_prompt "your prompt" --output_json outputs/eval_report.json
```

Optional: plot ablation losses

```bash
python -m src.plot_ablation --csv outputs/ablation/target_ablation_summary.csv
```

See `SUBMISSION_CHECKLIST.md` for metric notes (FVD proxy vs. full I3D FVD, user study, etc.).

5) Run prediction-target ablation

```bash
python -m src.ablate_targets --config configs/base.yaml
```

Outputs:
- `outputs/ablation/target_ablation_summary.json`
- `outputs/ablation/target_ablation_summary.csv`

## Bonus: narration (text-to-audio)

Uses **edge-tts** (neural voices) and **ffmpeg** to mux AAC audio into the MP4.

```bash
python -m src.generate --config configs/base.yaml --prompt "A rainy neon street..." ^
  --narration "In the distance, hover-trains slice through the rain." ^
  --voice en-US-GuyNeural --emotion calm
```

Or attach narration to an existing clip:

```bash
python -m src.add_narration --video outputs/generated/sample.mp4 --text_file narration.txt --config configs/base.yaml
```

Enable defaults in `configs/base.yaml` under `audio:` (`enable`, `narration_from_prompt`, `voice`, `emotion`, …). List voices: `edge-tts --list-voices`.

## Notes

- This is a research-oriented starter, not a final SOTA implementation.
- Place videos inside `data/videos` (subfolders are supported). File names are used as prompt text.
- You can switch objective in `configs/base.yaml` via `diffusion.prediction_target`.
- Add stronger temporal modules (e.g., temporal attention blocks, optical-flow regularization).
- Complete missing advanced metric backends as needed for your report.
