<p align="center"><img src="https://raw.githubusercontent.com/go-avkit/brand/main/social/go-avkit.png" alt="go-avkit" width="640"></p>

<h1 align="center">go-avkit</h1>
<p align="center"><strong>Pure-Go (CGO=0) audio/video toolkit — containers and bitstreams, with no libav/ffmpeg linkage and no external binaries.</strong></p>

<p align="center">
  🌐 <a href="https://go-avkit.github.io">Website</a> ·
  📚 <a href="https://go-avkit.github.io/docs/">Documentation</a>
</p>

<p align="center">
  <a href="https://go-avkit.github.io/docs/"><img alt="Docs" src="https://img.shields.io/badge/docs-mkdocs--material-0D9488?style=flat-square"></a>
  <a href="https://github.com/go-avkit/avkit/blob/main/LICENSE"><img alt="License: BSD-3-Clause" src="https://img.shields.io/badge/license-BSD--3--Clause-blue?style=flat-square"></a>
  <img alt="Go 1.26.4+" src="https://img.shields.io/badge/go-1.26.4%2B-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img alt="Coverage 100%" src="https://img.shields.io/badge/coverage-100%25-1a7f37?style=flat-square">
</p>

---

Seven modules, in two layers.

### Containers

| | |
|---|---|
| [`avkit`](https://github.com/go-avkit/avkit) | sniff and demux MP4/ISO-BMFF, Matroska/WebM and MPEG-TS onto one format-neutral model; mux fragmented and progressive MP4 and MPEG-TS; copy, cut, join, concatenate and drop tracks — **no re-encoding anywhere**. Also `ConformHVC1`, which is what makes macOS show an HEVC file at all |

### Bitstreams

| | |
|---|---|
| [`bitstream`](https://github.com/go-avkit/bitstream) | what every H.264/H.265-family reader needs first: the bit reader with the formats' Exp-Golomb, both NAL framings, and the unescaping a payload is meaningless without |
| [`h264`](https://github.com/go-avkit/h264) | the H.264/AVC bitstream **and the derivations of clause 8.2** — picture order counts, the reference picture set, the reference lists a slice predicts from, the prediction weights |
| [`h265`](https://github.com/go-avkit/h265) | the H.265/HEVC bitstream: NAL layer, parameter sets, slice segment header, picture boundaries |
| [`boolcoder`](https://github.com/go-avkit/boolcoder) | the boolean arithmetic decoder VP8 and VP9 both read their compressed partitions with |
| [`vp8`](https://github.com/go-avkit/vp8) | the VP8 frame tag and key-frame header |
| [`vp9`](https://github.com/go-avkit/vp9) | the VP9 uncompressed header and superframe index |

### ⛔ None of these is a decoder

Nothing in this organisation turns a bitstream into pixels. There is no entropy
decoding of residual, no transform, no motion compensation and no deblocking.
These read the syntax, and derive what the syntax implies — which is what a
decoder needs before it can start, and what a tool describing or remuxing a file
needs instead of one.

Two of the repositories used to claim otherwise in their own descriptions, and
that cost a reader a trip through the source to find out. They no longer do.

### How it is verified

100% statement coverage per function, gated in CI, on Windows, macOS and Linux
and six 64-bit architectures. Where another implementation can answer the same
question, it is asked: the reference set against `ffmpeg -debug mmco` (3110
reference pictures across 101 streams, zero disagreements), the reference lists
against an independent pure-Go decoder (785 lists, 18 streams, zero
disagreements), and the implicit bi-prediction weights over all 65536 pairs the
clause clips to.
