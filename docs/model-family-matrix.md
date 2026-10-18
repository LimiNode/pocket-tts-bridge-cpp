# PocketTTS model family matrix

Last reviewed: 2026-10-18

This is an investigation registry, not a release compatibility list. A row
moves to supported only after the exact model revision, tokenizer, configuration,
voice assets, runtime, and acceptance receipt are recorded.

## Candidate models

| Candidate | Observations from external material | What must be verified here |
| --- | --- | --- |
| Kyutai PocketTTS | baseline PocketTTS behavior and reference implementation | reference audio semantics, output format, streaming boundary, reproducibility |
| GenVoice xVibePocketTTS | Russian fine-tune; three voices, cloning, manual and automatic accent support, streaming API, PCM 24 kHz mono are described by its project materials | exact checkpoint/export, tokenizer, voice state, license terms, CPU/GPU latency, long-text behavior |
| ArtShtorm Shtorm-PocketTTS-RU | Russian SentencePiece tokenizer; v2 fast and slow profiles; intended for longer or calmer speech | profile differences, tokenizer and config identity, streaming behavior, voice state, license and attribution |
| amekhrishvili pocket-tts-ru | published six-layer model and 24-layer teacher; teacher path uses classifier-free guidance while ordinary inference does not | checkpoint contract, inference path, accent/number preparation, non-commercial restriction |
| Supertonic 3 | separate multilingual ONNX TTS family with Russian claims | not a PocketTTS checkpoint; evaluate only as a future generic backend |
| TeraTTS | separate Russian VITS project in the referenced material | not a confirmed Supertonic or PocketTTS derivative; outside this bridge's checkpoint contract |

The final two rows are recorded to prevent accidental family mixing. Similar
language coverage does not establish architecture or checkpoint compatibility.

## Third-party benchmark excerpt

The following values were transcribed from an external comparison covering a
dataset described as 1270 files. They are **external observations** and have not
been independently reproduced by this repository.

| Model | CER down | UTMOS up | Speaker Sim up | EnCodec FAD down | xRT GPU 1080 down |
| --- | ---: | ---: | ---: | ---: | ---: |
| Qwen3-TTS 0.6B | 1.10 | 3.733 | 0.822 | 1.02 | 0.91 |
| Qwen3-TTS 1.7B | 0.90 | 3.756 | 0.826 | 1.28 | 2.25 |
| xVibePocketTTS | 2.00 | 3.404 | 0.733 | 5.81 | 0.17 |
| PocketTTS Shtorm 1.75 | 3.86 | 3.511 | 0.721 | 9.85 | 0.17 |
| PocketTTS Shtorm 2.7 slow | 2.60 | 3.494 | 0.732 | 6.91 | 0.17 |
| PocketTTS Shtorm 2.7 fast | 2.71 | 3.486 | 0.736 | 6.17 | 0.17 |

The table is useful for experiment prioritization:

- xVibe is the strongest first Russian candidate in this excerpt by CER;
- Shtorm has a slightly higher UTMOS value in the listed rows, but that metric
  alone does not establish text accuracy, speaker identity, or dialogue
  suitability;
- the xRT column is explicitly GPU 1080 data. It cannot be transferred to an
  arbitrary CPU or GPU;
- the EnCodec FAD gap means PocketTTS should be evaluated as a compact and fast
  option, not assumed to dominate a larger model on maximum fidelity.

The meaning of xRT must also be confirmed by the source. If it is generation
time divided by audio duration, 0.17 would correspond to approximately 5.9x
real time and 0.91 to approximately 1.1x real time. That conversion is an
interpretation, not a local measurement.

## Published speed figures to reproduce

The xVibe project materials report the following conditions:

| Hardware | Reported RTF | Reported first audio |
| --- | ---: | ---: |
| Ryzen 7 3700X, CPU | 0.412 | about 193 ms |
| RTX 3090, GPU | 0.138 | about 37.5 ms |

The reported run is FP32, batch 1, warmed up, and uses one text of about
29 seconds. Model load and RUAccent time are excluded. Reproduce these
conditions before comparing them with this bridge. A Xeon or another CPU must
receive its own receipt.

## Complete model-bundle identity

Every experiment and cache entry must identify at least:

- model repository and exact revision or digest;
- weight files and SHA-256;
- architecture/configuration revision;
- tokenizer type, files, vocabulary, and SHA-256;
- generation parameters and random-seed policy;
- voice profile, reference-audio digest, or other conditioning asset;
- runtime name and exact version/commit;
- precision, device, batch size, and thread settings;
- text-preparation profile;
- license/provenance record.

A file named model.safetensors is not a sufficient identity. Loading weights
without the matching tokenizer or configuration is a failed compatibility
check, not a best-effort mode.

## Status vocabulary

Use these labels in future reports:

- **inventory** — found in external material, not loaded here;
- **reproduced** — loaded and exercised with a local receipt;
- **compatible** — exact bundle passes backend and lifecycle checks;
- **quality accepted** — compatible and passes the selected quality gate;
- **blocked** — a concrete incompatibility is recorded;
- **not evaluated** — no local evidence yet.
