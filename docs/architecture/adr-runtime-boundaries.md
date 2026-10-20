# ADR: runtime and bridge boundaries

- Status: accepted direction
- Date: 2026-10-18
- Scope: initial PocketTTS bridge architecture

## Context

PocketTTS now exists as a family of checkpoints and exports rather than one
stable artifact. Russian fine-tunes can carry their own tokenizer, generation
profiles, accent conventions, and voice-conditioning assets. Native C++/ONNX
implementations may provide lower startup and streaming overhead, while the
Python runtime remains the most useful behavior reference.

A bridge that lets model-specific details leak into the application would make
every new checkpoint a public API break. A bridge that hides all lifecycle
events would make cancellation, backpressure, and first-audio latency
unobservable.

## Decision

Use a persistent local worker and keep four boundaries explicit:

~~~text
application API
    -> transport/protocol
        -> worker supervisor
            -> backend adapter
                -> model bundle
~~~

### Application API

The future C++17 API owns request submission, request IDs, callback dispatch,
PCM metadata, cancellation, and terminal results. Submission is asynchronous
and returns before inference completes. The API must not expose Python, ONNX
session objects, tokenizer internals, or a model-specific voice class.

### Transport and protocol

The first transport is a local process boundary. The protocol carries:

- protocol version and capabilities;
- request ID and model-bundle identity;
- prepared text and text-profile identity;
- voice/reference selector or conditioning digest;
- PCM chunks with sample rate, channels, format, sequence number, and byte count;
- structured diagnostics and one terminal outcome.

A future WebSocket transport may reuse this protocol. It is a connection choice,
not a reason to redesign the public synthesis contract.

### Worker supervisor

The supervisor starts one worker, waits for a ready state, preserves the
loaded model between requests, captures stderr separately, and owns graceful
shutdown followed by a forced-termination fallback. It enforces startup,
request, and shutdown timeouts and turns unexpected worker exit into a
structured terminal error.

### Backend adapter

The adapter translates the neutral request into the selected runtime. Initial
adapters are:

1. Python reference runtime;
2. native C++/ONNX runtime;
3. sherpa-onnx where the exact export and tokenizer are compatible.

The adapter owns tokenizer invocation, model loading, generation parameters,
voice-state construction, and runtime-specific streaming. It reports
capabilities instead of pretending that every backend supports every request.

## Identity and receipt boundaries

The bridge does not treat every reproducibility field as one model identifier.
Use [Model-bundle identity and synthesis receipts](../model-bundle-identity.md):

- ModelBundleIdentity is the immutable checkpoint and tokenizer artifact;
- ExecutionProfile is the runtime, export, precision, device, and thread setup;
- VoiceConditionIdentity is the preset/reference conditioning identity;
- TextPreparationIdentity is the tokenizer-facing text profile;
- SynthesisReceipt references all of them and records the actual request/output.

Loaded-model caches, voice-state caches, prepared-text caches, and final-audio
caches must use the smallest complete key for the object they hold. Unknown or
partially verified identity fields fail closed.

## Audio event semantics

For each request:

1. zero or more ordered PCM events may be emitted;
2. sequence numbers increase without gaps;
3. cancellation or failure stops future chunks;
4. exactly one terminal event is emitted;
5. no chunk is delivered after the terminal event.

The bridge does not own a physical audio sink. Playback, mixing, and presentation
consume the PCM event stream in their own queues.

## Consequences

This design permits a Python quality reference and a native runtime to coexist.
It also makes first-audio latency and runtime capability differences visible.
The cost is an explicit protocol, model receipts, and more acceptance work than a
single hard-coded wrapper.

## Rejected shortcuts

- **Direct in-process model ownership in the first slice:** couples the
  application to PyTorch/ONNX and makes crashes or allocator state harder to
  isolate.
- **Weights-only model loading:** cannot prove tokenizer/config/voice
  compatibility.
- **One public voice type for every fine-tune:** hides incompatible conditioning
  semantics.
- **Buffered output called streaming:** gives a false latency contract.
- **Broad support for unrelated TTS families:** expands the API before the
  PocketTTS family contract is stable.
