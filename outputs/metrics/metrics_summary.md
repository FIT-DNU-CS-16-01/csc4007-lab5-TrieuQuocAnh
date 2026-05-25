# Metrics summary

- Dataset: `imdb`
- Model: `distilbert-base-uncased`
- Fine-tuning mode: `frozen_encoder_classifier_head_only`
- Seed: `42`
- Device: `cuda`
- Max length: `256`
- Trainable params: `592130` / `66955010`

## Validation

- Loss: `0.3985`
- Accuracy: `0.8236`
- Macro-F1: `0.8236`

## Test

- Loss: `0.3996`
- Accuracy: `0.8228`
- Macro-F1: `0.8228`

## Notes

Raw text -> pretrained tokenizer -> pretrained Transformer -> classification head. Best checkpoint selected by validation macro-F1.
