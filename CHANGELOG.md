# Changelog

## v2.0.0 — 2026-08-01

Complete rewrite. The repository now contains two Kaggle notebooks and nothing else.

The inherited Colab notebooks had stopped working entirely and could not be repaired in place —
their install cell pinned `torch==1.13.1`, `torchvision==0.14.1` and `torchaudio==0.13.1`, whose
wheels only exist for Python ≤ 3.10, while Colab and Kaggle both moved to 3.11+. pip failed before
any transcription code ran. Rather than patch a dead notebook, the working parts were rebuilt around
current libraries and a Hebrew-tuned model.

**Everything below has been run and verified on Kaggle's free tier**, except where noted.

---

### Added

**Two notebooks, split by whether you need speaker labels**

- `Kaggle/hebrew-transcription.ipynb` — transcription only. **Requires no account beyond Kaggle**:
  no Hugging Face signup, no token, no terms to accept. This is the one to hand to other people.
- `Kaggle/hebrew-diarization.ipynb` — transcription plus speaker identification.

**Automatic input discovery**

`INPUT_DIR` defaults to `None`, and the notebook searches every attached Kaggle Dataset plus a
`/kaggle/working/audio/` folder it creates. Dataset names no longer have to match anything, and no
path needs typing. An explicit path still works for narrowing the search, and a path to a single file
is accepted as well as a folder.

**Five output formats**

Selected with `OUTPUT_FORMATS = "all"` or a list such as `["txt", "srt"]`:

| Format | Purpose |
|---|---|
| `txt` | Readable transcript — paragraphs, or speaker blocks |
| `srt` | Subtitles for video players |
| `vtt` | Subtitles for the web |
| `tsv` | One row per segment, opens in Excel, times in whole milliseconds |
| `json` | Structured data, both the readable grouping and raw segments |

An unrecognised format name raises immediately with the valid set rather than silently producing
nothing.

**Optional timestamps**

`TXT_TIMESTAMPS = False` produces clean prose with no `[00:00:00]` markers. Scoped to `txt` alone —
the subtitle and data formats are defined by their timestamps.

**Word-level speaker assignment**

Whisper is asked for word-level timestamps; each word is matched to the diarization turn it overlaps
most, and consecutive same-speaker words are merged into readable blocks. This allows a speaker
change mid-sentence to be represented, which the previous one-speaker-per-segment approach could not
express at all.

**Paragraph grouping** (transcription notebook)

With no speaker changes to break on, Whisper's raw segments are a wall of fragments — 68 of them for
3.5 minutes of audio. Paragraphs now break after a 2-second pause or once one exceeds 45 seconds.

**Automatic audio conversion**

ffmpeg converts any audio or video input to 16 kHz mono. The original notebooks required users to
manually convert recordings to mono WAV first, using external software.

**Open in Kaggle badges**, in both notebooks and the README.

**Diagnostic output and actionable errors**

- The install cell prints resolved versions of torch, faster-whisper, pyannote, ctranslate2, plus
  CUDA availability, GPU name and cuDNN version. A cuDNN mismatch is the classic faster-whisper
  failure and its error names a library file rather than the real problem, so the number is on screen
  before it can be needed.
- A missing GPU raises with the exact menu path, a note that changing it restarts the session, the
  likely cause if the options are greyed out, and why continuing on CPU is not worth it — instead of
  `/bin/bash: line 1: nvidia-smi: command not found`.
- A missing input folder lists the datasets that *are* attached, distinguishing "nothing attached"
  from "attached under a different name".
- Installing numpy over a running kernel is detected and reported as a `STOP` box with numbered
  steps, at the point of cause, rather than surfacing four cells later as
  `ImportError: cannot import name '_center' from 'numpy._core.umath'`.

---

### Changed

| | Before | After |
|---|---|---|
| Platform | Google Colab | Kaggle |
| Transcription engine | `openai-whisper` | `faster-whisper` (CTranslate2), ~4× faster |
| Model | Whisper `tiny.en` as committed | `ivrit-ai/whisper-large-v3-turbo-ct2`, Hebrew-tuned |
| Language setting | `en` as committed | `he`, passed explicitly |
| Diarization | ECAPA embeddings + `AgglomerativeClustering` | `pyannote/speaker-diarization-community-1` |
| Speaker count | Required as input | Detected automatically |
| Storage | Google Drive mount | Kaggle Dataset in, `/kaggle/working/` out |
| Input format | Mono WAV, converted by hand | Any format ffmpeg reads |
| Credentials | Google Drive authorisation | Kaggle Secrets (diarization only) |

**Speaker detection no longer requires knowing the answer in advance.** The old method took
`num_speakers` as a required input, so a wrong guess silently forced the wrong number of buckets with
nothing in the output indicating a problem. Verified on a three-person call: all three found,
unprompted.

**Output is RTL-correct.** Human-readable formats carry U+200F marks so Hebrew renders properly in
editors that default to left-to-right. `tsv` and `json` deliberately omit them, since software
reading those files would treat an invisible character as data.

---

### Fixed

- **The install failure that killed the original notebooks.** `torch` is now never installed or
  pinned; the platform's build is used as shipped.
- **SRT timestamps were off by one millisecond on essentially every cue.** `int((2.4 - 2) * 1000)`
  truncates to 399, not 400. Timestamps are now computed in integer milliseconds throughout.
- **Speaker tie-breaking preferred the wrong turn.** When a word overlapped two diarization turns
  equally, the longer, less specific turn won. The tighter turn now wins.
- **A module-shadowing bug in the original:** `def time(secs)` shadowed the `time` module imported
  earlier, so re-running the transcribe cell after the save cell raised a `TypeError`.
- **`speechbrain` was required but never installed** in the original notebook.
- **`device=torch.device("cuda")` was hardcoded**, crashing on any CPU runtime. Device and compute
  type are now selected at runtime.
- **An unused dependency**, `srt`, was installed and never imported. Removed, and the build now
  checks that every pip-installed package is actually referenced.
- **`INPUT_DIR` pointing at a file rather than a folder** produced a "no files found" error naming a
  path that plainly existed, because `Path.exists()` is true for files. A single file is now valid
  input.

---

### Removed

- `Colab/` — all four original notebooks. They do not run and cannot be trivially repaired.
- `OtherASRs/` — unrelated experiments (Meta MMS, SpeechBrain Amharic).
- `Kaggle/kaggle-whisper-audio.ipynb` — superseded by `hebrew-transcription.ipynb`.
- `Resources/` — images used only by the previous README.
- Two configuration toggles, `WRITE_SRT` and `RTL_MARKS`, which gated behaviour nobody would switch
  off.

Everything removed remains available in the [`v1-legacy`](https://github.com/ablolarof/hebrewWhisper/tree/v1-legacy)
branch.

---

### Documentation

The README was rewritten from scratch: notebook comparison, step-by-step setup including the
Hugging Face steps people most often miss, a full configuration reference, output format guide,
measured performance figures, a troubleshooting table covering every failure encountered during
development, and a technical explanation of the pipeline.

---

### Verified on Kaggle

| | |
|---|---|
| Environment | Tesla T4 ×2, torch 2.10.0+cu128, cuDNN 9.10.2, ctranslate2 4.8.1, pyannote.audio 4.0.7, faster-whisper 1.2.1 |
| Throughput | ~9.5× realtime with diarization; ~4 min per hour of audio without |
| Memory | Both models fit comfortably in one T4's 15 GB |
| Speaker detection | 3 of 3 speakers found on a real call, unprompted |
| Hebrew quality | Judged good on reading. Not yet measured as a word error rate. |

**Not yet verified:** that `vtt`, `tsv` and `json` load correctly in their target applications
(structure is unit-tested, but no file has been opened in a video player or spreadsheet); and whether
the Hebrew finetune measurably outperforms plain `large-v3`, which remains an untested assumption.

### Testing

110 unit tests across five suites cover the speaker-overlap assignment, paragraph grouping, all five
output writers, format selection, and file discovery — including boundary-straddling words,
diarization gaps, nested and uppercase file extensions, and overlapping search roots. Notebook
structure and per-cell Python syntax are validated on every build.

---

## v1.0.0 — inherited

The original [Sourasky-DHLAB/Whisper](https://github.com/Sourasky-DHLAB/Whisper) repository as forked:
four Colab notebooks, one Kaggle notebook, and two other-ASR experiments. Last upstream commit
April 2025. Preserved in the [`v1-legacy`](https://github.com/ablolarof/hebrewWhisper/tree/v1-legacy)
branch.
