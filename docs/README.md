# Documentation index

This documentation layer describes the intended architecture and the evidence
needed before the bridge becomes a usable runtime.

| Document | Purpose |
| --- | --- |
| [Model family matrix](model-family-matrix.md) | candidate models, external benchmark data, and bundle identity |
| [Training and fine-tune notes](training-and-finetune-notes.md) | student/teacher, tokenizer, corpus, and evaluation implications |
| [Runtime boundary ADR](architecture/adr-runtime-boundaries.md) | ownership between C++, worker, transport, and backend |
| [Native/runtime evaluation](native-runtime-evaluation.md) | reproducible comparison of Python, native C++, and ONNX runtimes |
| [Text preparation profiles](text-preparation-profiles.md) | model-specific pronunciation and tokenizer contracts |
| [Model provenance and licensing](model-provenance-and-licensing.md) | code, weights, fine-tune, and BYOM boundaries |
| [Roadmap](roadmap.md) | implementation order, acceptance gates, and unresolved questions |

Reports in this directory must label third-party observations separately from
measurements made by this repository.
