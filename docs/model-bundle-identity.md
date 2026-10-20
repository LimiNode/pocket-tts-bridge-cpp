# Model-bundle identity and synthesis receipts

Last reviewed: 2026-10-20

A model artifact, a runtime configuration, a voice condition, and a prepared
text are different identities. Keeping them separate prevents a CPU/GPU change
or a tokenizer profile change from looking like a new model, while still making
the complete synthesis result reproducible.

## Identity objects

### ModelBundleIdentity

This is the immutable model artifact identity:

- source repository and exact revision or content digest;
- weight files and SHA-256 digests;
- architecture and immutable model configuration;
- tokenizer family, vocabulary, files, and digests;
- model metadata needed to interpret the checkpoint.

ModelBundleIdentity does not include the runtime, device, precision, thread count,
voice condition, or request text.

### ExecutionProfile

This describes how a bundle is executed:

- runtime name and exact version or commit;
- export or conversion format, opset, and conversion digest;
- precision, device, accelerator, and thread settings;
- model-load and warm-up policy;
- runtime generation defaults that are fixed for this execution profile.

Changing the execution profile does not change ModelBundleIdentity. It can change
the output and must therefore be present in a synthesis receipt.

### VoiceConditionIdentity

This identifies the conditioning input and its interpretation:

- preset voice identifier, or reference-audio digest;
- conditioning algorithm and revision;
- extracted voice-state digest when a state is cached;
- authorization-policy reference, without storing raw consent material in a
  synthesis cache key.

Raw reference audio and derived voice states are user data. Their storage and
retention rules are described in the provenance document.

### TextPreparationIdentity

This identifies how input text was prepared:

- profile name and revision;
- tokenizer-facing normalization policy;
- accent, number, Latin, punctuation, and segmentation rules;
- prepared-text digest and provenance marker.

The same source text under two profiles is not the same synthesis input.

### SynthesisReceipt

A receipt references the four identities above and records the actual operation:

- request ID and timestamp;
- request-level generation parameters and seed policy;
- text and voice-condition references;
- first-audio latency, total duration, and real-time factor;
- sample rate, channels, format, chunk count, and terminal outcome;
- output PCM or WAV digest;
- diagnostics and cancellation state.

A receipt may contain redacted references. It must not require raw reference audio
or private text to be persisted.

## Cache compatibility

Use the smallest key that fully determines the cached object:

| Cached object | Required key components |
| --- | --- |
| loaded model | ModelBundleIdentity + ExecutionProfile |
| extracted voice state | ModelBundleIdentity + VoiceConditionIdentity + conditioning revision |
| prepared text | source-text digest + TextPreparationIdentity |
| final PCM | all identities + request generation parameters + seed policy |

A device or runtime change invalidates a loaded-session cache, but it does not
create a new model artifact. A changed tokenizer, architecture configuration,
conversion digest, or conditioning algorithm is a compatibility failure unless
the backend explicitly declares the relationship.

Unknown, missing, or partially verified identity fields are cache misses and
must fail closed. Do not silently fall back to a nearest model, tokenizer, or
voice state.

## Serialization and diagnostics

The protocol should carry stable identity references and capability revisions,
not unbounded path strings or raw audio. Diagnostics should state which identity
failed validation and whether the failure happened during bundle loading,
execution-profile setup, voice conditioning, text preparation, or synthesis.

The public C++ API may initially expose opaque IDs and receipts. It does not need
to expose the internal hash layout before the persistence and cache semantics
are implemented.
