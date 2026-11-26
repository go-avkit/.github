<p align="center"><img src="https://raw.githubusercontent.com/go-avkit/brand/main/social/go-avkit.png" alt="go-avkit" width="640"></p>

<h1 align="center">go-avkit</h1>
<p align="center"><strong>Pure-Go (CGO=0) audio/video toolkit — containers and bitstreams, with no libav/ffmpeg linkage and no external binaries.</strong></p>

<p align="center">
  🌐 <a href="https://go-avkit.github.io">Website</a> ·
  📚 <a href="https://go-avkit.github.io/docs/">Documentation</a>
</p>

<p align="center">
  <a href="https://go-avkit.github.io/docs/"><img alt="Docs" src="https://img.shields.io/badge/docs-hugo%20%2B%20hextra-0079A8?style=flat-square"></a>
  <a href="https://github.com/go-avkit/avkit/blob/main/LICENSE"><img alt="License: BSD-3-Clause" src="https://img.shields.io/badge/license-BSD--3--Clause-blue?style=flat-square"></a>
  <img alt="Go 1.27.1+" src="https://img.shields.io/badge/go-1.27.1%2B-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img alt="Coverage 100%" src="https://img.shields.io/badge/coverage-100%25-1a7f37?style=flat-square">
</p>

---

Nine modules, in three layers. The detail of each (versions, measurements, caveats) is in the
[documentation](https://go-avkit.github.io/docs/main/modules/#module-notes).

### Containers

| | |
|---|---|
| [`avkit`](https://github.com/go-avkit/avkit) | MP4, Matroska/WebM and MPEG-TS read and written, common audio formats read; copies, cuts and joins tracks with no re-encoding, cuts HLS/DASH segments (optionally encrypted); refuses hostile files within bounds. [Details](https://go-avkit.github.io/docs/main/modules/#module-notes) |

### Bitstreams

| | |
|---|---|
| [`bitstream`](https://github.com/go-avkit/bitstream) | the bit reader, NAL framings and unescaping |
| [`h264`](https://github.com/go-avkit/h264) | H.264/AVC syntax and the clause 8.2 derivations; decoding about 2.5 times faster since v0.11.0, still slower than ffmpeg (open defect). [Details](https://go-avkit.github.io/docs/main/modules/#module-notes) |
| [`h265`](https://github.com/go-avkit/h265) | H.265/HEVC syntax and a conformance-verified decoder with a public `Decoder` API; about 0.2 to 0.3 of ffmpeg's speed on one core (open defect). [History and measurements](https://go-avkit.github.io/docs/main/history/) |
| [`boolcoder`](https://github.com/go-avkit/boolcoder) | the boolean arithmetic decoder VP8 and VP9 share |
| [`vp8`](https://github.com/go-avkit/vp8) | the VP8 frame tag and key-frame header |
| [`vp9`](https://github.com/go-avkit/vp9) | the VP9 uncompressed header and superframe index |

### Decoders

| | |
|---|---|
| [`decode`](https://github.com/go-avkit/decode) | H.264 and HEVC to planar YUV, FLAC, MP3, Opus, AAC and linear PCM to PCM; anything else is refused by name. Slower than ffmpeg on video and on HE-AAC v2 (1.42 times its user time), open defects. [What it claims, how it was measured](https://go-avkit.github.io/docs/main/decode/) |
| [`worker`](https://github.com/go-avkit/worker) | a `js/wasm` Web Worker: **poster** returns a JPEG of a frame, **probe** returns container, codec, size and duration. [Details](https://go-avkit.github.io/docs/main/modules/#module-notes) |

### The bitstream modules are not decoders

`bitstream`, `h264`, `boolcoder`, `vp8` and `vp9` read syntax and derive what it implies; they produce
no pixels. `h265` carries a decoder of its own, which `decode` uses. Only `decode` produces pictures or
sound. [Why, and what is verified](https://go-avkit.github.io/docs/main/history/#verification).

### How it is verified

100% statement coverage per function, gated in CI, on Windows, macOS and Linux and six 64-bit
architectures, and another implementation asked the same question wherever one exists.
