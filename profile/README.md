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
  <img alt="Go 1.27.1+" src="https://img.shields.io/badge/go-1.27.1%2B-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img alt="Coverage 100%" src="https://img.shields.io/badge/coverage-100%25-1a7f37?style=flat-square">
</p>

---

Eight modules, in three layers.

### Containers

| | |
|---|---|
| [`avkit`](https://github.com/go-avkit/avkit) | sniff and demux MP4/ISO-BMFF, Matroska/WebM and MPEG-TS — and, read whole, ADTS, MP3, FLAC, Ogg, WAV and AVI — onto one format-neutral model; mux fragmented and progressive MP4, Matroska/WebM and MPEG-TS; carry subtitles (WebVTT, SRT, TTML, 3GPP tx3g, PGS), edit lists and metadata between MP4 and Matroska/WebM; carry ALAC and Dolby TrueHD audio (not in MPEG-TS) and chapters (MP4 `chpl` and QuickTime track, Matroska); cut a file into HLS and DASH segments (the `media` module writes the playlists and manifest with go-streamkit; `media.Keys` states the key of an encrypted presentation: `EXT-X-KEY`, `EXT-X-SESSION-KEY`, `ContentProtection`); extract an audio track as ADTS, MP3, FLAC, Ogg or WAV; cut a file into aligned CMAF/fMP4 and WebVTT segments for HLS and DASH (v0.14.0, VOD; the playlists and MPD are go-streamkit's), encrypted with Common Encryption `cenc` or `cbcs` and a caller-supplied key from v0.15.0; copy, cut, join, concatenate and drop tracks — **no re-encoding anywhere**. Also `ConformHVC1`, which is what makes macOS show an HEVC file at all |

### Bitstreams

| | |
|---|---|
| [`bitstream`](https://github.com/go-avkit/bitstream) | what every H.264/H.265-family reader needs first: the bit reader with the formats' Exp-Golomb, both NAL framings, and the unescaping a payload is meaningless without |
| [`h264`](https://github.com/go-avkit/h264) | the H.264/AVC bitstream **and the derivations of clause 8.2** — picture order counts, the reference picture set, the reference lists a slice predicts from, the prediction weights |
| [`h265`](https://github.com/go-avkit/h265) | the H.265/HEVC bitstream: NAL layer, parameter sets, slice segment header, picture boundaries |
| [`boolcoder`](https://github.com/go-avkit/boolcoder) | the boolean arithmetic decoder VP8 and VP9 both read their compressed partitions with |
| [`vp8`](https://github.com/go-avkit/vp8) | the VP8 frame tag and key-frame header |
| [`vp9`](https://github.com/go-avkit/vp9) | the VP9 uncompressed header and superframe index |

### Decoders

| | |
|---|---|
| [`decode`](https://github.com/go-avkit/decode) | H.264 and HEVC, pure Go. H.264 wraps [oops1/go.264](https://github.com/oops1/go.264): Constrained Baseline, Baseline without FMO/ASO, Main progressive and High 4:2:0 8-bit progressive. HEVC wraps [gen2brain/h265](https://github.com/gen2brain/h265): Main and Main 10, 4:2:0. Every other profile is refused with `ErrUnsupported`. Takes a `TrackConfig`, an `avcC` or hvcC record, or Annex B; returns display-order planar YUV with its PTS, and `PosterFrame` returns one picture without decoding the track. 73 of the 81 JVT H.264.1 conformance streams give the same pictures as ffmpeg (8 refused); 143 of the 147 JCT-VC HEVC_v1 streams do, 3 more are decided by the suite's own MD5 (ours matches, ffmpeg differs) and 1 is refused by both; speed against ffmpeg is not yet measured |

### ⛔ The bitstream modules are not decoders

`bitstream`, `h264`, `h265`, `boolcoder`, `vp8` and `vp9` turn nothing into
pixels: no entropy decoding of residual, no transform, no motion compensation
and no deblocking. They read the syntax, and derive what the syntax implies —
which is what a decoder needs before it can start, and what a tool describing or
remuxing a file needs instead of one. Only `decode`, and only for H.264 and HEVC, produces
pictures.

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
