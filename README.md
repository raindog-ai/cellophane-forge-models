# Cellophane Forge — HTDemucs ONNX v1

Optional separation-model data for Cellophane. Downloaded separately after the app's **ONE SMALL FORMALITY** screen; never bundled in the application. Intended for creative aural research. You know. Research.

This release contains the base four-source HTDemucs model converted to a static ONNX graph. It is not the fine-tuned four-head model. User audio is processed on the user's device or explicitly paired Mac; this repository is a model download host, not a processing service.

## Artifact identity

- File: `htdemucs_static.onnx`
- Size: **174,266,705 bytes**
- SHA-256: `9938e9460cb3433932b3a44796c01eec543c3d7d3776f369dee5ad2d7eadfad3`
- Native source order: **drums, bass, other, vocals**
- Native sample rate: **44,100 Hz**, stereo
- Static inference window: **343,980 frames** (7.8 seconds)
- The companion runtime performs the STFT/CAC preparation, overlap-add reconstruction and final selection. This graph is not a standalone audio-file application.

## Source and conversion

Upstream: [Meta's Demucs](https://github.com/facebookresearch/demucs).
Checkpoint: `955717e8-8726e21a.th`, SHA-256 `8726e21a993978c7ba086d3872e7608d7d5bfca646ca4aca459ffda844faa8b4`.

The static export separates the learned network from the external STFT/iSTFT used by the companion runtime. The audited conversion recipe files have these SHA-256 values:

- `demucs_export.py`: `773dd6661225589e5573d4c95765e3dd6afcfef4c03f597353f9f4de5f5ce659`
- `reexport_static.py`: `9d546892d6e19cba36080a43d3dd663916e3470a487f3cc875b7940d02510aba`

These hashes identify the inspected recipes; they do not reconstruct an otherwise unrecorded historical export commit.

## License and training provenance

The upstream Demucs release uses the MIT license. The accompanying `LICENSE.demucs` preserves the Meta copyright and full license notice.

Upstream describes training on MUSDB-HQ and an additional 800 songs. [MUSDB18's dataset page](https://sigsep.github.io/datasets/musdb.html) documents the training recordings' separate terms and academic-use access conditions. This release includes no training audio and grants no rights in recordings users choose to process or in someone else's music. The app's research confirmation does not replace these notices.

## Qualification and limitations

The actual Cellophane runtime was compared against native PyTorch HTDemucs using identical deterministic stereo input spanning multiple inference windows. Checks cover all four source roles, unit gain, overlap boundaries and a padded final window. CPU ONNX and the app's CoreML-provider configuration passed. Negative controls for swapped vocal/other outputs, half gain and a one-sample shift failed as expected.

The CoreML provider executes supported graph partitions and uses CPU fallback for remaining operations. This is not an all-GPU claim. Qualification establishes numerical agreement on the tested fixture; it is not a perceptual score or a promise of perfect separation for arbitrary music.

Cellophane verifies the full byte count and SHA-256 before activation. Crate management, supplied stems, playback and manual loop preparation work without this download. Installing an update never changes the model identity pinned to an accepted Forge job.
