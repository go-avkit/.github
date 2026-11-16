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

Nine modules, in three layers.

### Containers

| | |
|---|---|
| [`avkit`](https://github.com/go-avkit/avkit) | sniff and demux MP4/ISO-BMFF, Matroska/WebM and MPEG-TS — and, read whole, ADTS, MP3, FLAC, Ogg, WAV and AVI — onto one format-neutral model; mux fragmented and progressive MP4, Matroska/WebM and MPEG-TS; carry subtitles (WebVTT, SRT, TTML, 3GPP tx3g, PGS), edit lists and metadata between MP4 and Matroska/WebM; carry ALAC and Dolby TrueHD audio (not in MPEG-TS) and chapters (MP4 `chpl` and QuickTime track, Matroska); cut a file into HLS and DASH segments (the `media` module writes the playlists and manifest with go-streamkit; `media.Keys` states the key of an encrypted presentation: `EXT-X-KEY`, `EXT-X-SESSION-KEY`, `ContentProtection`); extract an audio track as ADTS, MP3, FLAC, Ogg or WAV; cut a file into aligned CMAF/fMP4 and WebVTT segments for HLS and DASH (v0.14.0, VOD; the playlists and MPD are go-streamkit's), encrypted with Common Encryption `cenc` or `cbcs` and a caller-supplied key from v0.15.0; copy, cut, join, concatenate and drop tracks — **no re-encoding anywhere**. Since v0.17.0 a Matroska file that would unlace more frames than the bounds allow is refused with `ErrReaderLimit` before anything is allocated (`ReaderLimits` lowers the bounds). Also `ConformHVC1`, which is what makes macOS show an HEVC file at all |

### Bitstreams

| | |
|---|---|
| [`bitstream`](https://github.com/go-avkit/bitstream) | what every H.264/H.265-family reader needs first: the bit reader with the formats' Exp-Golomb, both NAL framings, and the unescaping a payload is meaningless without |
| [`h264`](https://github.com/go-avkit/h264) | the H.264/AVC bitstream **and the derivations of clause 8.2** — picture order counts, the reference picture set, the reference lists a slice predicts from, the prediction weights |
| [`h265`](https://github.com/go-avkit/h265) | the H.265/HEVC bitstream: NAL layer, parameter sets, slice segment header, picture boundaries; since v0.14.0 a decoder of our own (full slice-data syntax, intra reconstruction with deblocking and SAO, the DPB), with inter prediction since v0.15.0: 7147 hashed conformance pictures verified, 6638 of them inter, 0 mismatched, the 3091 pictures of the 59 hashless streams unverified (h265#48); a public `Decoder` API since v0.16.0, which also fixes a panic on a hostile SPS in v0.14.0 and v0.15.0; faster from v0.17.0 to v0.19.0 (HEVC kernels of go-videodsp/kernels) with the output unchanged, yet still slower than the third-party decoder on 8-bit streams (0.86 to 0.90 of its speed, an open defect) and far slower than ffmpeg (1080p, one core of an AMD Ryzen 9 7950X, load 0.15 to 1.06, 2026-11-16, ffmpeg 6.1.1 with one thread: Main 76.1 fps against 87.4 for the third-party decoder and 395 for ffmpeg; Main 10 79.3 against 30.0 and 303, 2.6 times as fast as the third-party decoder); and `Unit.TemporalID` is now TemporalId, not TemporalId + 2 |
| [`boolcoder`](https://github.com/go-avkit/boolcoder) | the boolean arithmetic decoder VP8 and VP9 both read their compressed partitions with |
| [`vp8`](https://github.com/go-avkit/vp8) | the VP8 frame tag and key-frame header |
| [`vp9`](https://github.com/go-avkit/vp9) | the VP9 uncompressed header and superframe index |

### Decoders

| | |
|---|---|
| [`decode`](https://github.com/go-avkit/decode) | H.264 and HEVC, pure Go. H.264 wraps [oops1/go.264](https://github.com/oops1/go.264): Constrained Baseline, Baseline without FMO/ASO, Main progressive and High 4:2:0 8-bit progressive. HEVC wraps [gen2brain/h265](https://github.com/gen2brain/h265): Main and Main 10, 4:2:0. Every other profile is refused with `ErrUnsupported`. Takes a `TrackConfig`, an `avcC` or hvcC record, or Annex B; returns display-order planar YUV with its PTS, and `PosterFrame` returns one picture without decoding the track. 73 of the 81 JVT H.264.1 conformance streams give the same pictures as ffmpeg (8 refused); 143 of the 147 JCT-VC HEVC_v1 streams do, 3 more are decided by the suite's own MD5 (ours matches, ffmpeg differs) and 1 is refused by both; audio: FLAC (bit-exact against ffmpeg; 32-bit refused), MP3 (maximum error 1.07e-6 against ffmpeg; MP2 not decoded) and Opus (passes all 12 RFC 8251 vectors with `opus_compare`; mono and stereo, 5.1 refused); Vorbis is not decoded; since v0.9.0, linear PCM (WAVE) decodes with its width and byte order from the `pcmC` record (64-bit float is rounded to `f32le`, lossy) and an unsupported video codec is refused by name (`codec "vp09" is not supported`) instead of being reported as a corrupt sample; since v0.6.0, AAC-LC and HE-AAC v1 (SBR) decode too, with MP4 gapless information (`iTunSMPB`, edit lists) applied as ffmpeg does, and, since v0.7.0, HE-AAC v2 (Parametric Stereo), with the full profile (procedure B, IPD/OPD) by default since v0.8.0, all 7 of 7 ISO vectors matching the `_ur` references (AAC speed, decode-only benchmark against ffmpeg's whole-process utime, Go 1.27.4, Apple M4 Max, one core: 5.1 at 0.966 of ffmpeg, stereo at 0.655; HE-AAC v2 is slower, about 1.6 times ffmpeg's utime on an x86 Zen 4 core and about 1.1 times on an Apple M4 (indicative), each row with its own load and ffmpeg version in the decode README, an open defect, decode#77); since v0.5.0 an MP4 edit list is applied by default, as ffmpeg does (`Config.IgnoreEditList` returns the raw track); **slower than ffmpeg on every video input measured, an open defect** (v0.3.0, single core, Apple M4 Max, ffmpeg 9.0.2 with one thread: H.264 High 1080p 36.5 against 173.1 fps, 0.21x; HEVC Main 10 2160p 10.3 against 27.9 fps, 0.37x; the committed streams 0.21x to 0.41x), with the hot spots mostly in the wrapped libraries; FLAC, MP3 and Opus speed is not measured |
| [`worker`](https://github.com/go-avkit/worker) | (v0.1.2) a `js/wasm` Web Worker that answers **poster** (a JPEG of the frame at a time) and **probe** (container, codec, size, duration) requests for a page that links neither a container reader nor a decoder; MP4 and WebM/Matroska, H.264 |

### ⛔ The bitstream modules are not decoders

`bitstream`, `h264`, `boolcoder`, `vp8` and `vp9` turn nothing into
pixels: no entropy decoding of residual, no transform, no motion compensation
and no deblocking. `h265` is the exception in progress: since v0.14.0 it
reconstructs intra pictures and since v0.15.0 inter ones too (with deblocking and
SAO), and since v0.16.0 it has a public `Decoder` API, but `decode` does not use it. The others read the syntax, and derive what the syntax implies —
which is what a decoder needs before it can start, and what a tool describing or
remuxing a file needs instead of one. Only `decode`, and only for H.264 and HEVC, produces
pictures, and only for FLAC, MP3 and Opus (and, since v0.6.0, AAC-LC and HE-AAC v1, and since v0.7.0 HE-AAC v2) does it produce sound.

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
