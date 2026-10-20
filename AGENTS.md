# AGENTS.md

This file is the repository-wide contract for work on pocket-tts-bridge-cpp.

## Project objective

Build a C++17 client bridge for a family of compact Pocket TTS checkpoints. The
public layer should be model-neutral and asynchronous. A persistent local worker
owns model loading and inference; the application-facing layer owns request
lifecycle, transport supervision, protocol validation, cancellation, and ordered
PCM delivery.

The first implementation must not attempt a ground-up C++ rewrite of the neural
network. Keep a reference runtime available while native backends are evaluated.

## Invariants

- Treat a checkpoint as a complete bundle: weights, tokenizer, architecture
  configuration, generation settings, voice-conditioning assets, and provenance.
- Never reuse a voice state across different bundle identities without an
  explicit compatibility proof.
- Keep transport, worker supervision, protocol, request state, and backend
  inference as separate responsibilities.
- Keep the worker persistent between requests; do not reload the model for each
  utterance.
- Make asynchronous submission return promptly. Deliver PCM through callbacks or
  an equivalent event stream and emit exactly one terminal event per request.
- Keep cancellation request-scoped and define what happens to PCM already queued.
- Do not call buffered synthesis streaming unless chunks are actually emitted
  before terminal completion.
- Do not commit model weights, generated builds, virtual environments, or
  packaged runtime artifacts.
- Do not turn an external benchmark into a local performance claim.
- Keep model code and model license terms separate from the bridge code license.

## Evidence policy

Documentation and code comments must identify whether a statement is:

- measured by this repository;
- reproduced from an external source;
- an engineering decision;
- an unverified hypothesis or an open compatibility gate.

Performance claims require hardware, software, model identity, precision, batch
size, text set, warm-up policy, and measurement method. Quality claims require a
fixed test set and a reproducible receipt.

## Implementation order

1. define model-bundle, execution, voice-condition, text-preparation, and
   synthesis-receipt identities;
2. implement protocol framing, a mock worker, and the persistent local
   transport;
3. implement the asynchronous C++ client, request state, cancellation, and
   terminal-event rules;
4. integrate the Python reference backend and one named checkpoint;
5. evaluate native/runtime adapters as an independent compatibility track;
6. package the first accepted Python-backed profile, then optional native
   profiles after their own gates.

The first usable bridge may be Python-backed. Successful native or ONNX
acceptance is not a prerequisite for that milestone.

## Documentation routing

Detailed, durable evidence belongs under docs/. Keep README.md concise and link
to the relevant report. New research must update the documentation index, state
its source and date, and preserve unresolved gates instead of hiding them.

Before changing code, search for an existing owner and prefer reuse over a
parallel abstraction. Keep C++17 code explicit, dependency additions narrow,
and public thread-safety/blocking behavior documented.
