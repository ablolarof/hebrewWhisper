# CLAUDE.md — hebrewWhisper

Guidance for Claude Code when working in this repository.

---

## Working rules

### 1. Debugging protocol — hypotheses before changes

**Do not edit the repo to fix a bug until the hypothesis step is done.**

When a bug or failure appears, respond in this order:

1. **Hypotheses.** State one or more concrete, specific candidate causes. Not
   "something is wrong with the install" — rather "pip resolved ctranslate2 4.x,
   which requires cuDNN 9, but the Kaggle image ships cuDNN 8".
   If there are several plausible causes, list them and say which is most likely
   and why.
2. **Proposed fix per hypothesis.** What change would follow if that hypothesis
   is the right one.
3. **How to test.** Give the exact command, cell, or check that discriminates
   between the hypotheses — ideally something that produces a different result
   depending on which one is true. Say what output would confirm and what would
   refute.
4. **Wait for the test result.**
5. **Only then** change the repo.

The point is to avoid shotgun fixes that change several things at once and leave
nobody sure which one mattered. If a fix is applied without a confirming test,
label it explicitly as unverified — in the commit message and in `MEMORY.md`.

This rule applies to bug fixes. It does not apply to new features, refactors,
docs, or work the user has explicitly scoped.

### 2. Prefer the simpler, shorter solution

When two approaches both work, take the smaller one. Fewer lines, fewer
dependencies, fewer moving parts, fewer concepts the reader has to hold at once.

If the longer option is chosen anyway, say why in one line — usually it will be
correctness, a real edge case, or a clear error message that the short version
loses. "It's more extensible" or "it's more general" is **not** a good enough
reason here. This is a transcription notebook for researchers, not a framework;
speculative generality is a cost with no payoff.

Applied to this project specifically:

- No class hierarchies or abstraction layers for things a plain function does.
- No config framework. One config cell with plain variables is correct.
- Don't add a dependency to save a handful of lines.
- Don't handle cases that cannot occur. Do handle cases that produce a confusing
  failure for the user.

When proposing a fix or feature, if a shorter alternative exists and was
rejected, mention it and why. That is more useful than presenting one option as
though it were the only one.

### 3. Explain the reasoning, not the mechanics

Assume the reader is an amateur coder — comfortable running a notebook, not
fluent in Python. Comments should explain **why** the code is shaped this way,
because the *what* is usually already readable.

Not this:

```python
# loop over the words
for w in words:
```

But this:

```python
# A word can fall in a gap between diarization turns (a pause, a cough).
# Rather than drop it or label it unknown, it inherits the previous speaker,
# which is nearly always right in a two-person interview.
```

Prioritise explaining:

- **Non-obvious choices.** Why `exclusive_speaker_diarization` and not the plain
  view. Why 16 kHz mono. Why `language` is passed explicitly instead of
  autodetected.
- **Things that look wrong but are deliberate.** Integer-millisecond timestamp
  maths, the `break` in the overlap loop, not pinning torch.
- **Anything a past bug touched.** If it was fixed once, the comment should say
  what went wrong, so it isn't reintroduced.
- **Units and formats.** Seconds vs milliseconds, 0-based vs 1-based.

Markdown cells carry the user-facing explanation in Hebrew; code comments carry
the developer-facing reasoning in English. Both should be present. Keep comments
short — a dense paragraph above every line is its own kind of unreadable, and
would violate rule 2.

### 4. Distinguish verified from unverified

This project cannot be fully tested on the user's machine (see Environment).
Anything not actually executed must be described as untested. Do not let a
plausible-looking fix be reported as a working one. `MEMORY.md` tracks which
claims have been confirmed by a real run.

### 5. Don't pin `torch` — and be careful pinning anything else

Pinning is rarely free on Kaggle. The image ships hundreds of packages, and an
exact pin can force pip into a backtracking search for a compatible combination
that takes minutes, or does not terminate usefully at all. The tell is repeated
identical download sizes in pip's output: that is metadata for successive
candidate versions being tried, not progress.

- **Never pin `torch`.** The original notebooks died precisely because they
  pinned an old torch against a newer platform. Use the host's build.
- **Prefer `>=` minimums** on the few libraries layered on top.
- **Do not pin to avoid a restart.** This was tried with numpy and reverted; the
  resolve cost more than the restart it saved. If a package upgrade destabilises
  the running kernel, detect it and tell the user to restart. That is cheap,
  reliable, and does not fight the resolver.

The original notebooks died precisely because they pinned an old torch against a
newer platform. Use whatever PyTorch build the host (Kaggle) ships. Install only
libraries layered on top, and pin those with `>=` minimums rather than exact
versions.

---

## What this repo is

A fork of [Sourasky-DHLAB/Whisper](https://github.com/Sourasky-DHLAB/Whisper),
a Hebrew audio/video transcription toolkit from Tel Aviv University's Sourasky
Central Library. Upstream went stale (last commit April 2025) and its notebooks
no longer run.

**Goal:** revive it, move off Google Colab onto a free platform, with
`Whisper_Speaker_Diarization.ipynb` as the priority.

### Layout

| Path | Status |
|---|---|
| `Kaggle/hebrew-diarization.ipynb` | **Active.** Transcription + speaker labels. Needs a HF token. |
| `Kaggle/hebrew-transcription.ipynb` | **Active.** Transcription only, no account needed. The one to hand to other people. |
| `Kaggle/kaggle-whisper-audio.ipynb` | Upstream. Plain transcription, no diarization. Mostly still sound. |
| `Colab/*.ipynb` | Upstream. **Broken.** Kept for reference only — do not try to repair in place. |
| `OtherASRs/*.ipynb` | Upstream. Unrelated experiments (Meta MMS, SpeechBrain Amharic). |
| `Resources/` | Images used by the README. |

### Why the Colab notebooks are dead

`Colab/Whisper_Speaker_Diarization.ipynb` fails at the install cell: it pins
`torch==1.13.1` / `torchvision==0.14.1` / `torchaudio==0.13.1`, whose wheels only
exist for Python <=3.10, while Colab and Kaggle now run 3.11+. Secondary
problems: `pyannote.audio` installed unpinned from git `main` (now 4.x, whose API
differs from the 2.x calls in the notebook), `speechbrain` required but never
installed, `device=torch.device("cuda")` hardcoded, and the committed config is
English (`tiny.en`, `lang='en'`) in a Hebrew notebook.

---

## Current stack

| Concern | Choice |
|---|---|
| Platform | Kaggle Notebooks (~30 GPU-hr/week free, T4 x2 / P100 16 GB, 12 h sessions) |
| ASR engine | `faster-whisper` (CTranslate2) |
| Model | `ivrit-ai/whisper-large-v3-turbo-ct2` — Hebrew-finetuned, CT2 format |
| Diarization | `pyannote/speaker-diarization-community-1` (pyannote.audio 4.x) |
| Audio prep | ffmpeg -> 16 kHz mono WAV, accepts any input format |
| I/O | Kaggle Dataset in, `/kaggle/working/` out |
| Secrets | HF token via Kaggle Secrets, name `HF_TOKEN` |

### API details worth remembering

- pyannote 4.x uses `Pipeline.from_pretrained(..., token=...)`. The 3.x name
  `use_auth_token=` is **wrong** for this version.
- community-1 returns an object exposing `.speaker_diarization` and
  `.exclusive_speaker_diarization`. Prefer the exclusive view: its turns are
  non-overlapping, which is what you want when reconciling against transcript
  timestamps.
- The ivrit-ai models have degraded language *detection* by design. Always pass
  `language="he"` explicitly; never rely on autodetect.
- pyannote models are gated. They are free, but need a HF account, a token, and
  a one-time click-through accepting the model terms. A 401 here usually means
  the terms were never accepted, not that the token is bad.

---

## Environment constraints

The user's machine is **AMD Radeon integrated graphics, no CUDA, 6 GB total
system RAM**. Consequences:

- Models cannot be run locally. Do not propose local inference as a solution.
- Notebooks cannot be executed end-to-end here. Verification is limited to
  static checks: JSON validity, `ast.parse` on each code cell, and unit tests of
  pure-Python logic extracted from cells.
- Real GPU behaviour (cuDNN compatibility, VRAM headroom, model download) can
  only be confirmed by the user running it on Kaggle.

Test scripts for the extractable logic live in the scratchpad, not the repo.
If they become worth keeping, move them to `tests/`.

---

## Git

- `origin` -> `https://github.com/ablolarof/hebrewWhisper.git` (the user's fork)
- `upstream` -> `https://github.com/Sourasky-DHLAB/Whisper.git`

Upstream is effectively abandoned; expect no incoming changes. Do not push
without the user asking — the fork is public.

---

## Conventions

- Hebrew UI text in notebooks is wrapped in `<div dir="rtl">`. Keep it.
- Written transcripts get a U+200F RLM prefix per line so Hebrew renders
  correctly in LTR-defaulting editors.
- Notebook markdown mixes Hebrew (user-facing instructions) and English
  (technical notes, troubleshooting). That split is intentional.
- Keep the config in one cell near the top. Users should not have to hunt
  through the notebook to change a path or a model.

---

## The two notebooks share code by duplication, not by import

`hebrew-transcription.ipynb` repeats the ffmpeg prep, file discovery, error
messages, timestamp formatting and RTL output from `hebrew-diarization.ipynb`.

That is deliberate. A Kaggle notebook has to be self-contained — a shared module
would mean users cloning a repo before they could run anything, which defeats
the point of the token-free notebook existing at all.

The cost is real: **a fix to shared logic must be applied to both.** The
duplicated parts are the ones with test coverage, so a change to timestamp
formatting or output writing should be made in both build scripts and re-tested
against both. If the duplication grows much past its current size, revisit —
but do not trade away self-containedness cheaply.
