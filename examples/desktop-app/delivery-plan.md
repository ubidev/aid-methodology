# Delivery Plan: Import & Diarization (Delivery 2d)

*Produced by AID wf-plan phase*

## Overview

| Field | Value |
|-------|-------|
| Delivery | 2d |
| Branch | `delivery-2d` |
| Base | `main` (after 2c merge, 934 tests) |
| Features | F9: File Import, F10: Speaker Diarization |
| Target | 1,031+ tests, zero warnings |

## Scope

### F9: File Import
Allow users to import existing audio/video files for transcription instead of recording live.

**Key decisions:**
- FFmpeg for format conversion (later replaced with NAudio/MediaFoundation for Windows Store compatibility)
- Supported formats: WAV, MP3, M4A, OGG, FLAC, WMA, MP4, MKV, WebM, AVI
- Magic bytes validation (not just extension)
- `RecordingSource` enum to distinguish recorded vs. imported

### F10: Speaker Diarization
Post-transcription speaker identification — "who said what."

**Key decisions:**
- sherpa-onnx with pyannote segmentation + 3D-Speaker embeddings (chosen via spike)
- Lives inside Transcript tab as post-processing, not separate tab
- "Identify Speakers" button visible only when diarization model is downloaded
- Two quality tiers planned: Quick (Silero VAD + WeSpeaker) and Accurate (Pyannote + 3D-Speaker)

## Dependencies

- F10 does NOT depend on F9 — diarization works on any recording with audio
- Both features require EF Core migrations
- Both features modify `MainViewModel` (coordinate changes)

## Parallel Execution

F9 and F10 can be implemented in parallel by separate agents since they touch different service layers. Shared files (MainViewModel, App.axaml.cs, Recording model) require merge coordination.

## Exit Criteria

- [ ] All new features have unit tests
- [ ] Build succeeds with zero warnings
- [ ] Code review grade ≥ A-
- [ ] E2E tests updated for new UI elements
- [ ] Branch ready for PR to main
