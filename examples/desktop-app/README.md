# Example: Desktop Transcription App

## Context

A Windows desktop application for audio transcription built with:
- .NET 10 / C# / Avalonia UI (cross-platform MVVM)
- Whisper (local, offline speech-to-text)
- Phi-3 (local AI enhancement/correction)
- sherpa-onnx (speaker diarization)
- SQLite (local storage)

**Goal:** Ship incremental deliveries, each self-contained and fully tested.

## AID Phases Applied

This project demonstrates the **Plan→Detail→Implement→Review→Test→Deploy** cycle repeated across multiple deliveries.

### Delivery Structure

| Delivery | Focus | Tests Added | Total Tests |
|----------|-------|-------------|-------------|
| 2a | Core transcription + UI | 782 | 782 |
| 2b | AI enhancement + settings | 144 | 926 |
| 2c | Auto-correct pipeline + raw/enhanced separation | 8 | 934 |
| 2d | File import + speaker diarization | 97 | 1,031 |
| 2e | E2E tests (3-tier: Mock/Integration/Full) | 153 | 1,184 |

### How Plan→Detail Works

**Plan (aid-plan):** Defines delivery scope — e.g., "Delivery 2d: Import & Diarization" with two features (F9: File Import, F10: Speaker Diarization).

**Detail (aid-detail):** Breaks each feature into task specs with:
- Interfaces to implement
- Models and migrations needed
- UI changes with element specifications
- Acceptance criteria (testable)
- File list (which files to create/modify)

### Quality Gates

- **Review (built into aid-implement):** Every delivery reviewed before merge. Graded A+ to F. P1/P2 issues fixed inline, P3+ documented.
- **Test (aid-test):** Three-tier E2E testing:
  - **Mock:** All services mocked, CI-safe, fast (~80 tests)
  - **Integration:** Real data services + filesystem, hardware mocked (~67 tests)
  - **Full:** Real models, requires hardware (`[Trait("Requires", "Whisper")]`)

## Key Files

- [delivery-plan.md](delivery-plan.md) — Example delivery plan (Delivery 2d)
- [task-spec.md](task-spec.md) — Example task specification (File Import feature)

## Lessons Learned

1. **Incremental deliveries beat big-bang releases.** Each delivery was mergeable independently. If any failed review, only that delivery was affected.
2. **Task specs are the real unit of work.** The coding agent reads the task spec, implements it, and the review checks against the spec. No ambiguity.
3. **Three-tier testing catches real bugs.** Integration tests found a SQLite `OrderBy(TimeSpan)` incompatibility that Mock tests missed.
