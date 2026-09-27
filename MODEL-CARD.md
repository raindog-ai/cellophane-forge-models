# HTDemucs ONNX — Portable Audio Separation

A static ONNX export of the base four-source HTDemucs model for audio separation projects, including personal use and aural research. It separates **drums, bass, other instruments, and vocals** when integrated with a compatible audio-processing runtime.

The export is designed for efficient inference and portability through ONNX tooling. These are design goals, not measured speedups or a guarantee of compatibility with every device or execution provider. This is model data, not a standalone audio-file processor or hosted processing service. This release converts upstream pretrained weights; no new training or fine-tuning was performed for this export. It is the base HTDemucs model, not the fine-tuned four-head version.

## Download and identity

[Download the ONNX model](https://github.com/raindog-ai/cellophane-forge-models/releases/download/htdemucs-onnx-v1/htdemucs_static.onnx) · [Release files](https://github.com/raindog-ai/cellophane-forge-models/releases/tag/htdemucs-onnx-v1) · [Demucs software license and attribution](https://github.com/raindog-ai/cellophane-forge-models/blob/main/LICENSE.demucs)

| Property | Value |
| --- | --- |
| File | `htdemucs_static.onnx` |
| Release tag | `htdemucs-onnx-v1` |
| Size | **174,266,705 bytes** |
| SHA-256 | `9938e9460cb3433932b3a44796c01eec543c3d7d3776f369dee5ad2d7eadfad3` |
| Native source order | **drums, bass, other, vocals** |
| Native audio | **44,100 Hz, stereo** |
| Static inference window | **343,980 frames (7.8 seconds)** |

Verify the full byte count and SHA-256 before loading this artifact. Keep the verified identity with any processing results so a later model update cannot silently change their provenance.

## Runtime contract

The graph exposes float32 inputs `mix` and `z_cac`, and outputs `time_out` and `zout_cac`. It separates the learned network from the external audio DSP. A host integration must supply the matching waveform and complex-as-channels (CAC) spectrogram preparation, normalization, STFT/iSTFT, window padding, output reconstruction, and overlap-add scheduling. Loading a WAV directly as a single model input is not sufficient.

The qualified reference integration used right-zero-padded 343,980-frame windows, a 257,985-frame stride (25% overlap), triangular overlap-add with transition power 1, and unit-gain float32 output. Correct source ordering and exact output-frame trimming are part of that integration contract. Different preprocessing, scheduling, resampling or channel handling requires separate validation.

ONNX Runtime execution-provider support is operator- and platform-dependent. The tested CoreML provider configuration used supported graph partitions with CPU fallback for the remaining operations; it was not exclusive GPU or Neural Engine execution. No universal hardware compatibility or performance benchmark is claimed.

## Source and conversion provenance

Upstream: [Meta's Demucs](https://github.com/facebookresearch/demucs).

- Base checkpoint: `955717e8-8726e21a.th`
- Checkpoint SHA-256: `8726e21a993978c7ba086d3872e7608d7d5bfca646ca4aca459ffda844faa8b4`
- Audited `demucs_export.py` SHA-256: `773dd6661225589e5573d4c95765e3dd6afcfef4c03f597353f9f4de5f5ce659`
- Audited `reexport_static.py` SHA-256: `9d546892d6e19cba36080a43d3dd663916e3470a487f3cc875b7940d02510aba`

These recipe hashes identify the inspected conversion scripts. They do not reconstruct an otherwise unrecorded historical export commit. The conversion retains the base model's four-source task; it does not establish new training or a new separation-quality benchmark.

## License, intended use, and training provenance

The upstream **Demucs software** uses the MIT license. `LICENSE.demucs` preserves that software copyright and license notice; it must not be interpreted as a verified MIT grant for pretrained model weights.

The upstream author [states that the weights are not covered by MIT and are provided for scientific purposes](https://github.com/facebookresearch/demucs/issues/327#issuecomment-1134828611). That statement predates HTDemucs v4; we have not verified a superseding redistribution grant for this converted artifact. Separately, the author's [FT model card removed its MIT license metadata](https://huggingface.co/adefossez/HTDemucs-ft/commit/d74ac89c3a1e874fc78f152555cf4d8533f06cd4); that is FT-specific evidence, not a new license for this base export. **Weight redistribution permission remains unresolved. This card does not grant it.**

Personal audio projects and aural research describe the intended use, not a new weight license or permission to redistribute the artifact. User acknowledgements and separate downloading do not establish upstream permissions. Use recordings you own or have permission to process; separating audio does not grant permission to publish or redistribute someone else's music.

Upstream describes training on MUSDB-HQ and an additional 800 songs. The [MUSDB18 dataset page](https://sigsep.github.io/datasets/musdb.html) documents separate recording terms and academic-use access conditions. Those are training-data provenance, not a substitute model license. No training audio is included in this release, and this distribution grants no rights in the recordings a user supplies.

## Numerical qualification and limitations

A reference audio integration was compared with native PyTorch HTDemucs using identical deterministic stereo input spanning two inference windows (400,123 frames). The comparison used Demucs 4.0.1 / PyTorch 2.11.0 and ONNX Runtime 1.24.2, including the host DSP and reconstruction path.

CPU ONNX and the tested CoreML-provider configuration passed checks for all four source roles, unit gain, overlap boundaries, and the padded final window. No fitted gain correction, time shift, or source permutation was used to make the comparison pass. Negative controls for half gain, swapped vocal/other outputs, a one-sample shift, and a missing quiet vocal output failed as expected.

This establishes numerical agreement on the tested generated fixture, **not perceptual quality on arbitrary music**. Its vocal output was quiet and was checked explicitly. The comparison does not qualify other models, alternate runtime integrations, every sample rate or channel conversion, native `.mlpackage` conversion, or default Demucs CLI scheduling. It does not establish wall-clock efficiency, memory use, device thermal behavior, or universal deployment compatibility. Those require measurements in the target integration.
