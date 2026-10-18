# Native and reference runtime evaluation

Last reviewed: 2026-10-18

This document defines the first feasibility gate. It is not a promise that any
candidate runtime supports every PocketTTS fine-tune.

## Runtime candidates

| Runtime | Intended role | Main question |
| --- | --- | --- |
| original Python PocketTTS | quality and behavior reference | does the exact bundle load and stream repeatably? |
| PocketTTS.cpp | native C++/ONNX candidate | do Russian exports, tokenizer, voice cloning, and INT8 preserve behavior? |
| sherpa-onnx | mature ONNX infrastructure candidate | does the supported PocketTTS export match the selected checkpoint revision? |

A reported C++ implementation or framework feature is an investigation lead. It
becomes a supported backend only after the exact bundle is exercised.

## Compatibility gate

For each model candidate, run the same text, voice condition, seed policy, and
prepared-text profile through every runtime that claims compatibility.

Record:

- model repository, revision, and file digests;
- tokenizer and configuration digests;
- voice/reference asset digest;
- runtime version or commit;
- export format and opset where applicable;
- precision and device;
- sample rate, channels, sample format, and PCM framing;
- first-audio latency;
- model load, warmup, and total synthesis times;
- real-time factor;
- peak RSS and VRAM;
- cancellation behavior and terminal event timing;
- a WAV or PCM artifact for listening and regression;
- quality metrics and a fixed text corpus.

## Quality and performance separation

A faster runtime is not accepted if it changes pronunciation, speaker identity,
truncates long text, or corrupts PCM. INT8 results require an explicit quality
comparison against the same model in FP32 or the accepted reference precision.

Do not compare a native runtime on one checkpoint with a Python runtime on
another and call the result a backend speedup. Do not transfer GPU RTF figures
to CPU hardware.

## First experiment matrix

Use at least:

| Scenario | Purpose |
| --- | --- |
| short emotional reply | first-audio latency and EOS |
| 20–30 second neutral paragraph | RTF and long-form stability |
| Russian homograph set | accent and text-preparation behavior |
| numbers, years, and mixed Latin | tokenizer/profile boundary |
| repeated requests in one worker | model reuse and state isolation |
| cancellation during active synthesis | lifecycle correctness |
| two different voice conditions | voice-state binding |

Run warm and cold cases separately. State thread count, CPU affinity,
environment variables, and whether model load and text preparation are included.

## Receipt format

A future machine-readable receipt should contain at least:

~~~json
{
  "model_bundle": {
    "repository": "...",
    "revision": "...",
    "weights_sha256": "...",
    "tokenizer_sha256": "...",
    "config_sha256": "..."
  },
  "runtime": {
    "name": "...",
    "version": "...",
    "export": "...",
    "precision": "fp32",
    "device": "cpu"
  },
  "input": {
    "profile": "...",
    "text_set": "...",
    "voice_condition": "..."
  },
  "metrics": {
    "first_audio_ms": 0,
    "rtf": 0.0,
    "peak_rss_mb": 0
  },
  "audio": {
    "sample_rate": 0,
    "channels": 0,
    "format": "pcm_s16le",
    "sha256": "..."
  },
  "status": "not-evaluated"
}
~~~

The zero values above are placeholders. A receipt with missing identity fields
must not be used as compatibility evidence.

## Known risks to test

- newer PocketTTS exports may differ from the export supported by an ONNX
  framework;
- tokenizer and voice-state assumptions may be embedded in the Python wrapper;
- native INT8 may improve throughput while changing timbre or pronunciation;
- a worker can report ready while the first request still fails during lazy
  processor loading;
- output labeled streaming may actually be generated as one complete buffer.

The acceptance result must preserve the failing artifact and the exact error
before any compatibility workaround is introduced.
