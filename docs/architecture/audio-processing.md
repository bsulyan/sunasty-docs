# WebAssembly Audio Processing

Audio editing and vocal extraction traditionally require heavyweight desktop digital audio workstations (DAWs) or server-side FFmpeg pipelines. Sending hours of private interview recordings or podcast stems to unknown third-party cloud backends raises substantial privacy and bandwidth concerns.

The [Sunasty Audio Suite](https://sunasty.com/audio/) and [Sunasty Video Suite](https://sunasty.com/video/) execute audio demuxing, stem processing, and normalization directly in the user's browser using compiled **WebAssembly (WASM)** modules and the Web Audio API.

## Client-Side Audio Pipeline

```
 User Audio / Video File (MP4, MKV, WAV, MP3)
                       │
             [ In-Memory Blob Stream ]
                       │
        WebAssembly Demuxer & Decoder (FFmpeg WASM)
                       │
       Decoded Raw PCM Stream (AudioBuffer)
                       │
      ┌────────────────┼────────────────┐
      ▼                ▼                ▼
  Vocal / Stem     Volume / EBU     ID3v2 Tag
   Isolation        R128 Norm        Editor
      │                │                │
      └────────────────┼────────────────┘
                       ▼
       WASM LAME / Opus / WAV Encoder
                       │
           Client-Side Download Stream
```

## Engineering Highlights

### 1. Zero-Upload Audio Extraction
Converting a 2 GB MP4 screen recording to an MP3 podcast episode with [Sunasty MP4 to MP3](https://sunasty.com/mp4-to-mp3/) takes seconds because no upload or download bandwidth is consumed. The browser demuxes the MP4 container, extracts the AAC or PCM audio track, passes it through an in-memory LAME MP3 encoder, and provides an instant download link.

### 2. EBU R128 Loudness Normalization
Podcasts require consistent audio levels across episodes to prevent sudden volume spikes for listeners. The Sunasty audio pipeline computes Integrated Loudness (LUFS) and Loudness Range (LU) entirely inside a Web Worker using K-weighting filtering according to ITU-R BS.1770-4:

- Pre-filter: High-shelf filter boosting frequencies above 1.5 kHz.
- RLB weighting: High-pass filter cutting sub-bass below 38 Hz.
- Gating: Rejects silent and low-level pauses to compute accurate speech loudness.

### 3. Local ID3v2 Tag Inspection & Modification
Embedded metadata in MP3 and FLAC files (artist, title, album, cover art, chapter markers) is parsed by traversing the ID3v2 binary frame headers (`TIT2`, `TPE1`, `TALB`, `APIC`). The tags can be sanitized or edited locally without recompressing the audio frames.

Explore client-side audio tools:
- [Sunasty Audio Hub](https://sunasty.com/audio/)
- [Sunasty Video Hub](https://sunasty.com/video/)
- [Sunasty MP4 to MP3 Converter](https://sunasty.com/mp4-to-mp3/)
- [Sunasty Vocal Remover & Stem Splitter](https://sunasty.com/vocal-remover/)
- [Sunasty Audio Tag Editor](https://sunasty.com/audio-tags/)
- [Sunasty Audio Speed & Pitch Changer](https://sunasty.com/audio-speed/)
- [Sunasty In-Browser Screen Recorder](https://sunasty.com/screen-recorder/)
- [Article: Cleaning Podcast Audio in the Browser](https://sunasty.com/articles/clean-audio-for-podcasts/)
