# Training and fine-tune notes

Last reviewed: 2026-10-18

This document records architectural implications of the published Russian
PocketTTS fine-tunes. It does not prescribe a training recipe where the source
material does not provide one.

## A fine-tune is an architecture package

A Russian checkpoint cannot be described only by its number of layers or its
weight file. Its effective behavior depends on:

- student network topology and hidden dimensions;
- teacher network and any distillation path;
- tokenizer type, vocabulary, and normalization;
- conditioning representation for speaker or reference audio;
- generation profile and guidance parameters;
- export format and runtime assumptions;
- training-time text preparation conventions.

A bridge therefore identifies an inference profile, not just a tensor archive.

## Observations from the candidate family

### xVibe

The published material describes Russian fine-tuning with several voices,
voice cloning, manual and automatic accent support, streaming, and 24 kHz mono
PCM. These are behavior claims to reproduce. The bridge must preserve explicit
accent input and must not assume that the automatic accent tool is safe to run
after caller-provided accents.

### Shtorm

The referenced v2 material has fast and slow profiles and a Russian SentencePiece
tokenizer with a vocabulary of about 4000 tokens. The profiles should be treated
as separate generation configurations even if they share most weights. A runtime
cache key must include the profile revision.

### pocket-tts-ru research example

The referenced training example publishes a six-layer student and a 24-layer
teacher. The teacher is described as using classifier-free guidance, while the
ordinary inference path does not use that guidance. A poor CER result from the
teacher row alone therefore does not establish that the student inference model
is unusable. The evaluation harness must record which model and guidance path
produced each score.

## Dataset and transcript contract

A reproducible fine-tune needs a written corpus contract:

- speaker and recording provenance;
- train, validation, and test split policy;
- text normalization and punctuation policy;
- explicit treatment of ё, combining accents, and plus-notation accents;
- numbers, abbreviations, Latin fragments, and technical tokens;
- duration, clipping, silence, and sample-rate filters;
- speaker leakage checks between splits;
- transcript checksum or immutable manifest revision.

The model matrix must not compare CER or speaker similarity across datasets whose
text normalization and split policy are unknown.

## Student, teacher, and inference identity

Receipts for a trained model must distinguish:

- the student checkpoint used for ordinary synthesis;
- the teacher checkpoint used for distillation or evaluation;
- whether classifier-free guidance was active;
- guidance scale and conditioning dropout settings;
- tokenizer and vocabulary revision;
- generation profile and random-seed policy.

A teacher checkpoint is not automatically an inference backend. A model that
requires guidance during training may still have a different ordinary inference
contract.

## Evaluation layers

Use separate gates for:

1. **text fidelity** — CER or an equivalent transcription metric on a fixed,
   held-out corpus;
2. **naturalness** — UTMOS or human ratings with the model and text set named;
3. **speaker identity** — speaker similarity with a documented reference;
4. **audio distribution** — FAD or a comparable metric with the codec and
   preprocessing stated;
5. **systems behavior** — first audio, RTF, memory, streaming cadence, and
   cancellation.

No single score can replace the others. A fast model with good CER may still
have unacceptable timbre; a high UTMOS value does not prove correct numbers or
homographs.

## Architecture consequences for the bridge

The worker/backend contract must be able to advertise:

- tokenizer and text-preparation profile;
- generation profile and guidance support;
- voice-cloning or preset-voice capabilities;
- maximum context or segmentation policy;
- sample format and nominal sample rate;
- supported precision and export format.

The client should receive a capability mismatch before synthesis starts. It
should not discover at runtime that a profile requires a tokenizer or guidance
mode the selected backend cannot provide.

## Open training gates

- Which corpus and normalization recipe produced each Russian checkpoint?
- Are fast and slow profiles separate weights or generation-only variants?
- Does native INT8 preserve CER, speaker similarity, and accent behavior?
- Are voice-cloning assets distributable under the same terms as the weights?
- Which export revisions preserve the student/teacher and tokenizer assumptions?
