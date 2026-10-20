# Roadmap and acceptance gates

Last reviewed: 2026-10-18

The roadmap deliberately starts with evidence and lifecycle contracts. Native
inference is valuable only after the reference behavior is observable.

## Phase 0 — documentation foundation

- maintain the model-family matrix and bundle identity rules;
- define runtime, text, audio, provenance, and evidence contracts;
- record external benchmark numbers as external observations;
- keep unresolved compatibility and licensing gates visible.

Exit: a new model can be described without inventing support or mixing its
tokenizer, configuration, voice state, and license with another model.

## M0 — bridge core contract

- define model-bundle, execution, voice-condition, text-preparation, and
  synthesis-receipt identities;
- implement protocol framing and a mock worker;
- implement the persistent local transport and asynchronous client lifecycle;
- cover request IDs, queueing, cancellation, backpressure, and terminal events.

Exit: a mock worker can exercise the complete client contract without any model
weights or native inference dependency.

## M1 — Python reference backend

- integrate the original Python PocketTTS runtime behind the backend interface;
- load one named checkpoint and record its complete bundle identity;
- run a fixed Russian/English corpus with a declared text-preparation profile;
- save PCM/WAV artifacts and environment receipts.

Exit: the first usable bridge is Python-backed, persistent, asynchronous, and
reproducible. Native acceptance is not required for this exit.

## M2 — model and native compatibility research

- extend the model matrix to xVibe, Shtorm fast/slow, and permitted research
  checkpoints;
- test PocketTTS.cpp against exact selected bundles;
- test sherpa-onnx only for matching exports and tokenizer paths;
- compare FP32 and INT8 where the runtime supports both;
- preserve failed exports and lazy-loading errors as receipts.

Native feasibility may proceed in parallel with M1. Its result determines which
optional profiles can be promoted; it does not block the Python-backed bridge.

## M3 — optional native release profiles

- promote a native backend only after PCM, lifecycle, and quality gates pass for
  a named bundle;
- advertise backend capabilities and unsupported profiles explicitly;
- keep native and Python receipts comparable without conflating their execution
  identities.

Exit: at least one native profile is accepted as an optional backend. The
Python-backed profile remains an independent release path.

## M4 — packaging and release

- package the accepted worker without committing model weights;
- provide explicit model-location and offline modes;
- reproduce the selected receipt on a clean target machine;
- publish attribution and model-license information next to the release.

Exit: the release artifact, model bundle, runtime, and license obligations are
independently identifiable.

## Definition of done for the first usable version

The first usable version is complete when:

- M0 and M1 pass for one named PocketTTS bundle;
- the worker remains alive between requests;
- asynchronous submission returns promptly;
- PCM chunks arrive before terminal completion;
- sample metadata and sequence numbers are valid;
- active and queued cancellation have defined behavior;
- repeated requests do not cross-contaminate voice state;
- model, tokenizer, configuration, voice assets, runtime, and license are
  recorded;
- a reproducible receipt and audio artifact are available;
- setup and offline operation are documented.

Native acceptance belongs to M3 and is not a prerequisite for the first usable
Python-backed bridge.

## Open questions

- which Russian checkpoint should be the first release profile;
- whether native streaming matches reference chunk cadence or only final audio;
- which newer exports remain loadable by sherpa-onnx;
- whether INT8 preserves the required speaker similarity and pronunciation;
- how much text segmentation should happen before the model tokenizer;
- which voice assets may be redistributed with a release.
