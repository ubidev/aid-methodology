# Task Specification: F9 — File Import

*Produced by AID wf-detail phase*

## Summary

| Field | Value |
|-------|-------|
| Feature | F9: File Import |
| Delivery | 2d |
| Priority | P1 |
| Estimated effort | ~4 hours (AI-assisted) |
| Dependencies | None (independent of F10) |

## Description

Users should be able to import existing audio and video files for transcription, instead of only recording live audio. The import flow: select file → validate → convert to 16kHz mono WAV → store → enqueue for transcription.

## Interfaces

### IFileImportService
```csharp
public interface IFileImportService
{
    Task<ImportResult> ImportFileAsync(string filePath, IProgress<ImportProgress>? progress = null, CancellationToken ct = default);
}

public record ImportResult(bool Success, Recording? Recording, string? Error);
public record ImportProgress(ImportPhase Phase, double Percentage, string StatusText);
public enum ImportPhase { Converting, Transcribing }
```

### Supporting Services
- `FileValidator` — static class: `IsSupportedAudioVideo(string)`, `ValidateMagicBytes(string)`, `GetFileSize(string)`
- `IDialogService` — add `ShowOpenFileDialogAsync()` returning `string?`

## Models

### RecordingSource (new enum)
```csharp
public enum RecordingSource { Microphone, FileImport }
```

### Recording (modified)
Add properties:
- `RecordingSource Source { get; set; }` (default: Microphone)
- `string? SourceFilePath { get; set; }`

### EF Migration
- `20260316000000_AddFileImportFields` — adds Source and SourceFilePath columns

## UI Changes

### MainWindow.axaml
- Import button in toolbar (📂 icon) next to record button
- Import progress bar (same style as transcription progress)

### RecordingsListView.axaml
- Source badge on imported recordings (📂 vs 🎤)

### RecordingDetailView.axaml
- Source info section showing original file path (if imported)

## Supported Formats

| Category | Extensions |
|----------|-----------|
| Audio | .wav, .mp3, .m4a, .ogg, .flac, .wma |
| Video | .mp4, .mkv, .webm, .avi |

## Validation

1. Extension check against supported list
2. Magic bytes validation (first 12 bytes)
3. File size check (configurable max, default 2GB)

## Acceptance Criteria

- [ ] User can import audio files via toolbar button
- [ ] User can import files via drag-and-drop
- [ ] Unsupported formats show clear error message
- [ ] Import progress is visible (converting → transcribing)
- [ ] Imported recordings show source badge in list
- [ ] Imported recordings show original file path in detail view
- [ ] Cancel button works during conversion
- [ ] Unit tests for FileValidator, FileImportService, MainViewModel import commands
- [ ] All existing tests still pass

## Files to Create/Modify

**New files:**
- `Services/FileImport/IFileImportService.cs`
- `Services/FileImport/FileImportService.cs`
- `Services/FileImport/FileValidator.cs`
- `Services/FileImport/AudioConverterService.cs`
- `Models/RecordingSource.cs`
- `Migrations/20260316000000_AddFileImportFields.cs`
- `Tests/Services/FileImport/FileValidatorTests.cs`
- `Tests/Services/FileImport/FileImportServiceTests.cs`
- `Tests/ViewModels/MainViewModelImportTests.cs`

**Modified files:**
- `Models/Recording.cs` — add Source, SourceFilePath
- `ViewModels/MainViewModel.cs` — add import commands
- `Views/MainWindow.axaml` — import button
- `Views/RecordingsListView.axaml` — source badge
- `Views/RecordingDetailView.axaml` — source info
- `Services/DialogService.cs` — add ShowOpenFileDialogAsync
- `App.axaml.cs` — DI registration
- `AppDbContext.cs` — entity configuration
