# MFAA Face-Verification Adaptation

This folder documents the repository adaptation of **MFAA**: *Enhancing the Transferability of Adversarial Attacks via Multi-Feature Attention*.

## Suitability

MFAA is a reasonable attack to evaluate in this repository because its core idea is CNN-compatible: it uses intermediate feature maps and attention-style guidance rather than relying on Vision Transformer tokens. The original method is designed for image classification, so this implementation adapts the objective to face verification by replacing class-logit guidance with the repository's embedding-similarity objective.

## Adaptation details

- Attack name: `MFAA`
- Surrogate models: the existing DeepFace-backed attacker models in `core.transfer_attack_core.ATTACKER_MODELS`
- Objective:
  - `impersonation_attack`: increase source-to-target embedding similarity
  - `dodging_attack`: decrease source-to-target embedding similarity through the existing dodging objective
- Feature guidance:
  - select spatial intermediate layers from the surrogate CNN
  - estimate multi-feature guidance using random input masking
  - combine the feature-attention objective with a small embedding-similarity objective for stability
- Perturbation budget and steps follow the shared repository constants unless changed in `core/transfer_attack_core.py`.

## Expected performance

This adaptation should be treated as an experimental baseline, not a guaranteed improvement. MFAA may improve transferability when intermediate features are useful and stable across face-recognition CNNs, but its original paper and public implementations target classification models. Face verification uses embedding similarity rather than class logits, so performance must be measured against the official vanilla baseline CSVs.

## Suggested evaluation

Generate adversarial images on the small subset with:

```bash
python experiments/run_vanilla_subset_generation.py \
  --input-csv docs/subset_input_pairs.csv \
  --dataset-root /content/face_module/dataset_extractedfaces \
  --output-root /content/mfaa_outputs \
  --attacker-model ArcFace \
  --attacks MFAA
```

Repeat for `Facenet512`, `GhostFaceNet`, and `VGG-Face`, then evaluate with the same victim-model similarity pipeline used for the official baseline summaries.
