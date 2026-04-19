# Experiment Log

## Baseline
Name: ende_transformer_baseline_v1
Language pair: English -> German
Framework: JoeyNMT
Model: Transformer
Metric: BLEU

## Upgraded mtdata from 0.4.0 to 0.5.0
We initially tried mtdata==0.4.0 to match the provided reference, but switched to the current release because of compatibility issues related to deprecated pkg_resources behavior in the local Python environment.

## Did not use newstest_deen 2021
Dataset: 'Statmt-newstest_deen-2021-deu-eng' had an AssertionError where source and target files aren't aligned.

## Dataset and Data Files
Training datasets:
- Statmt-europarl-10-deu-eng
- Statmt-news_commentary-18.1-deu-eng

Development/Validation dataset:
- Statmt-newstest_deen-2019-deu-eng

Test dataset:
- Statmt-newstest_deen-2020-deu-eng

Location: data/deu-eng/

Train:
- train.eng
- train.deu

Dev:
- dev.eng
- dev.deu

Test:
- test.eng
- test.deu

## Baseline:

### Baseline Tokenization decision:

- Baseline: word-level
- Planned tuning experiment: BPE/subword
Reason:
- word-level is easier to debug and explain for the first end-to-end build
- BPE will be evaluated later as a likely improvement for English-German translation

### Baseline Data preprocessing / vocabulary settings

- `lowercase: false`: preserve original casing for easier interpretation and to retain capitalization information.
- `max_sent_length: 100`: filter very long sentences to make the first run more stable and easier to debug.
- `vocab_max_size: 30000`: keep vocabulary large enough for a word-level baseline, but still manageable.
- `vocab_min_freq: 2`: remove extremely rare tokens so the vocabulary does not become unnecessarily noisy.

### Baseline model settings

- `type: transformer`  
  We use a Transformer encoder-decoder architecture to satisfy the project’s attention requirement.

- `num_layers: 4`  
  We start with 4 encoder layers and 4 decoder layers to keep the proof-of-concept model smaller and easier to train than a larger research-scale setup.

- `num_heads: 4`  
  We use 4 attention heads as a moderate baseline that still allows multi-head attention behavior without making the model too heavy.

- `hidden_size: 256`  
  We use a hidden size of 256 to keep the first model computationally manageable.

- `ff_size: 1024`  
  We use a larger feed-forward layer inside each Transformer block to give the model enough internal capacity while staying proportionate to the hidden size.

- `dropout: 0.2`  
  We use dropout for regularization to reduce overfitting risk in the baseline.

- `embedding_dim: 256`  
  We match the embedding size to the hidden size for a simple and consistent baseline.

- `tied_embeddings: false`, `tied_softmax: false`  
  We leave weight tying off in the proof-of-concept model to keep the initial setup easier to interpret before tuning.

### Baseline training settings

- `optimizer: adam`  
  We use Adam as a stable baseline optimizer for the proof-of-concept run.

- `learning_rate: 0.0005`  
  We start with a moderate learning rate to prioritize stable initial training over aggressive tuning.

- `scheduling: plateau`  
  We use a plateau scheduler so the learning rate decreases when validation performance stops improving.

- `patience: 5` and `decrease_factor: 0.7`  
  These settings reduce the learning rate gradually rather than too quickly.

- `loss: crossentropy`  
  We use cross-entropy because translation training predicts the next target token at each step.

- `label_smoothing: 0.0`  
  We leave label smoothing off in the proof of concept and reserve it for tuning later.

- `batch_size: 2048`, `batch_type: token`  
  We use token-based batching because sentence lengths vary in translation tasks.

- `clip_grad_norm: 1.0`  
  We use gradient clipping to improve training stability.

- `epochs: 20`  
  We limit the initial run to a manageable number of epochs for baseline verification.

- `validation_freq: 1000`, `logging_freq: 100`  
  We validate and log regularly so we can monitor learning progress during the first run.

- `early_stopping_metric: bleu`  
  We monitor BLEU because it is the assignment’s main evaluation metric.

- `keep_best_ckpts: 3`  
  We keep a few strong checkpoints without accumulating too many files.

### Baseline testing settings

- `batch_size: 1000`, `batch_type: token`  
  We use token-based batching during validation and testing to stay consistent with translation-style variable-length batching, while keeping test-time batches moderate for stability.

- `max_output_length: 100`  
  We cap generated translation length at 100 tokens to prevent runaway decoding.

- `eval_metrics: ["bleu"]`  
  We use BLEU as the primary evaluation metric because it is the assignment’s required metric.

- `beam_size: 1`  
  We use greedy decoding for the proof-of-concept baseline to keep inference simple and fast.

- `beam_alpha: -1`  
  We do not apply length penalty in the baseline because beam search is reserved for later tuning.

- `sacrebleu.lowercase: false`  
  We keep BLEU evaluation case-sensitive to match the uncased-preserving baseline data setup.

### Data repair note
The original `train.eng` file contained embedded Unicode line separator characters (`U+2028`) inside sentence text. JoeyNMT uses Python `splitlines()` when loading plain text, so those characters were interpreted as extra line breaks and caused source-target misalignment. We normalized the files and rebuilt the final dataset in `data/deu-eng-fixed/`, where train/dev/test source and target files have matching line counts.

Source files can be found in 'others/Misalignment in JoeyNMT plain-text loading'

