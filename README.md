# PocketTTS Bridge

C++17 bridge for the Pocket TTS model family. The project is designed for
compact local speech synthesis with a persistent worker, incremental PCM
delivery, voice conditioning, and interchangeable reference/native runtimes.

The repository is currently in the architecture and feasibility phase. The
documentation defines the contracts that the implementation must satisfy; it
does not claim that a model backend or a public C++ API is already complete.

## Design target

The bridge should expose one model-neutral asynchronous surface while keeping
model-specific code behind a backend adapter:

~~~text
application
    |
    v
PocketTtsClient
    |
    v
persistent worker + request lifecycle
    |
    +--> reference PocketTTS runtime
    +--> native C++/ONNX runtime
    +--> other compatible runtime
    |
    v
ordered PCM chunks + one terminal result
~~~

The first transport is expected to be a local process boundary. It keeps
PyTorch or ONNX dependencies out of the application process, allows a Python
runtime to remain a quality reference, and leaves room for a native backend
without changing the client contract.

The bridge is intentionally a **family bridge**, not a wrapper for one
checkpoint. A Pocket TTS model is loadable only when its complete model bundle
is identified and validated:

~~~text
weights + tokenizer + architecture/config + model-specific defaults
       + voice-conditioning assets + provenance/license metadata
~~~

Runtime, device, precision, conversion, and request generation parameters are
recorded separately in the execution and synthesis identities described in
[docs/model-bundle-identity.md](docs/model-bundle-identity.md).

A voice state produced for one bundle must never be silently reused with another
bundle.

## Candidate model families

| Family | Role in the feasibility phase | Current status |
| --- | --- | --- |
| Kyutai PocketTTS | reference English model and baseline semantics | reference path to validate |
| xVibePocketTTS | Russian fine-tune candidate with fast local inference claims | candidate; compatibility and terms require verification |
| Shtorm-PocketTTS-RU | Russian fast/slow profiles for longer or calmer speech | candidate; compatibility and terms require verification |
| pocket-tts-ru | research training example with a separate tokenizer/teacher setup | research input; not a release default |

The external comparison quoted in the model matrix is useful for choosing
experiments, but it is not a project benchmark and must not be presented as a
guarantee for a particular CPU, GPU, driver, quantization mode, or text mix.

## Runtime strategy

The implementation should compare these paths before committing to a large
native inference layer:

1. the original Python runtime as the quality and behavior reference;
2. a native C++/ONNX runtime such as PocketTTS.cpp;
3. sherpa-onnx where its PocketTTS export and tokenizer compatibility match the
   selected checkpoint.

A runtime is not accepted because it can load an English checkpoint. It must
load the exact Russian or English bundle under test, produce valid PCM, and
pass the lifecycle and quality gates in
[docs/native-runtime-evaluation.md](docs/native-runtime-evaluation.md).

## Text and audio contracts

Text preparation is a model-facing boundary. The bridge must carry an explicit
TextPreparationProfile (or an equivalent internal value) so that accent
notation, numbers, Latin fragments, punctuation, pauses, and sentence length
are adapted to the selected tokenizer. A prepared string must not be silently
run through a second accenting pass.

The normalized audio surface should expose sample rate, channel count, sample
format, request ID, sequence number, and a terminal reason. Streaming means
ordered PCM chunks arrive before the terminal event; a full-buffer result
relabelled as streaming is not sufficient.

## What is deliberately out of scope for the first slice

- reimplementing the Pocket TTS neural network in C++;
- committing model weights or a Python environment;
- assuming that all PocketTTS exports share a tokenizer or voice state;
- selecting a commercial-use license from a code license alone;
- making Supertonic or VITS-based systems part of this bridge's checkpoint
  contract;
- exposing a network service before the local lifecycle and provenance rules
  are stable.

## Documentation map

- [Documentation index](docs/README.md)
- [Model and benchmark matrix](docs/model-family-matrix.md)
- [Model-bundle identity and receipts](docs/model-bundle-identity.md)
- [Training and fine-tune notes](docs/training-and-finetune-notes.md)
- [Runtime boundary ADR](docs/architecture/adr-runtime-boundaries.md)
- [Native/runtime evaluation](docs/native-runtime-evaluation.md)
- [Text preparation profiles](docs/text-preparation-profiles.md)
- [Model provenance and licensing](docs/model-provenance-and-licensing.md)
- [Roadmap and acceptance gates](docs/roadmap.md)

## References

- [Kyutai PocketTTS](https://github.com/kyutai-labs/pocket-tts)
- [GenVoice xVibePocketTTS](https://github.com/GenVoice/xVibePocketTTS)
- [Shtorm-PocketTTS-RU](https://huggingface.co/ArtShtorm/Shtorm-PocketTTS-RU)
- [PocketTTS.cpp](https://github.com/VolgaGerm/PocketTTS.cpp)
- [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx)

All model, runtime, and benchmark references remain subject to their own
versions, export formats, and licenses.
