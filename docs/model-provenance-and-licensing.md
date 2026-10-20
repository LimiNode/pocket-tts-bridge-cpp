# Model provenance and licensing

Last reviewed: 2026-10-20

The bridge code, runtime dependencies, model weights, fine-tune data, and
voice assets are separate legal and provenance objects. A permissive code
license does not grant rights to redistribute or commercially use a checkpoint.

## Inventory from the initial investigation

| Object | Reported terms or status | Required action |
| --- | --- | --- |
| bridge code | intended to be MIT-licensed | keep notices and dependency licenses complete |
| base Kyutai weights | CC BY 4.0 in the referenced material | preserve attribution and verify the exact revision |
| Shtorm v2 weights | CC BY 4.0 in the referenced material | preserve attribution and verify the model card |
| amekhrishvili fine-tune | CC BY-NC 4.0 in the referenced material | exclude from a commercial bundle unless terms change |
| xVibe fine-tune | code and base-model terms do not by themselves prove fine-tune rights | inspect the exact model card and redistribution terms |
| user-supplied model | depends on the user's rights | support bring-your-own-model with an explicit acknowledgement |

These entries are an investigation snapshot, not legal advice or a substitute
for reading the current model card and repository notices.

## Provenance record

For every model that can be loaded, store or print:

- source URL and exact revision;
- download date;
- file names and SHA-256;
- base model and fine-tune relationship;
- tokenizer/config/voice asset sources;
- license identifiers and required attribution;
- whether redistribution is allowed;
- whether commercial use is allowed;
- any dataset or speaker restrictions;
- runtime/export conversion history.

A missing license field is an unresolved gate. It is not permission.

## Bring-your-own-model boundary

A BYOM mode may let a user point the bridge at locally obtained weights without
redistributing them. It still must validate the bundle and display provenance
metadata. BYOM does not erase the user's obligations or allow the project to
repackage a non-commercial checkpoint.

The packaging system should keep model files external to the bridge binary and
should not silently download a checkpoint. Downloads, when implemented, must
require an explicit source, revision, and digest, with an offline/no-download
mode.

## Voice reference authorization and privacy

Reference audio and derived voice states are separate user data. Permission to
use model weights does not authorize cloning an arbitrary person's voice.

The bridge must require an explicit lawful authorization policy for a reference
voice and must not infer consent from public availability, a file path, or the
license of the model. The policy should be attached to the
VoiceConditionIdentity as a reference, not stored as raw consent material in
ordinary synthesis caches.

Default handling:

- do not persist raw reference audio;
- do not persist derived voice states beyond the worker session unless the user
  explicitly opts in;
- do not write raw audio or voice embeddings to routine logs;
- use a digest and policy revision in receipts where reproducibility requires a
  reference;
- never reuse a voice state across users, model bundles, or conditioning
  algorithms without an explicit compatibility and authorization check.

If a runtime or model card imposes stricter requirements, the stricter rule
wins. An authorization failure is a fail-closed request error.

## Related but separate families

Supertonic and VITS-based Russian systems may be useful future backends, but
language coverage alone does not make them PocketTTS checkpoints. Adding one
would require a separate capability contract, export matrix, and license review.
