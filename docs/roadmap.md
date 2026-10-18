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

## Phase 1 — reference harness

- implement a mock worker and protocol framing;
- add a persistent-worker lifecycle and request-state model;
- run the original Python runtime on a fixed short corpus;
- save PCM/WAV artifacts and environment receipts.

Exit: repeated requests, cancellation, terminal events, and PCM metadata are
deterministic enough to compare backends.

## Phase 2 — model matrix

- load Kyutai, xVibe, Shtorm fast/slow, and the research checkpoint where
  access and terms permit;
- record tokenizer/config/voice bundle identities;
- test text-preparation profiles and long-form behavior;
- mark each model inventory, reproduced, compatible, blocked, or not evaluated.

Exit: exact checkpoint compatibility is known for each selected runtime.

## Phase 3 — native feasibility

- test PocketTTS.cpp against the exact selected bundles;
- test sherpa-onnx only for matching exports and tokenizer paths;
- compare FP32 and INT8 where the runtime supports both;
- preserve failed exports and lazy-loading errors as receipts.

Exit: at least one native path passes PCM correctness, lifecycle behavior, and
the selected quality threshold for a named bundle.

## Phase 4 — bridge implementation

- implement the public async C++17 client;
- add request IDs, cancellation, bounded queues, worker supervision, and
  structured diagnostics;
- keep backend capability negotiation explicit;
- add a WAV example and a no-model mock path.

Exit: an application can submit multiple requests to one worker and receive
ordered PCM plus exactly one terminal result per request.

## Phase 5 — packaging and release

- package the accepted worker without committing model weights;
- provide explicit model-location and offline modes;
- reproduce the acceptance receipt on a clean target machine;
- publish attribution and model-license information next to the release.

Exit: the release artifact, model bundle, runtime, and license obligations are
independently identifiable.

## Definition of done for the first usable version

The first usable version is complete when:

- one named PocketTTS bundle loads through a documented backend;
- the worker remains alive between requests;
- asynchronous submission returns promptly;
- PCM chunks arrive before terminal completion;
- sample metadata and sequence numbers are valid;
- active and queued cancellation have defined behavior;
- repeated requests do not cross-contaminate voice state;
- model, tokenizer, configuration, voice assets, runtime, and license are recorded;
- a reproducible receipt and audio artifact are available;
- setup and offline operation are documented.

## Open questions

- which Russian checkpoint should be the first release profile;
- whether native streaming matches reference chunk cadence or only final audio;
- which newer exports remain loadable by sherpa-onnx;
- whether INT8 preserves the required speaker similarity and pronunciation;
- how much text segmentation should happen before the model tokenizer;
- which voice assets may be redistributed with a release.
