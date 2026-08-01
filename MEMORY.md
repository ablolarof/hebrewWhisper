# MEMORY.md — hebrewWhisper project log

Running record of decisions, findings, and open questions. Update at the end of
each working session. Newest entries at the top of the log.

See `CLAUDE.md` for the working rules (notably: hypotheses before bug fixes).

---

## Status at a glance

| | |
|---|---|
| **Phase** | **Phase 0 complete** — notebook runs end to end on Kaggle. Moving to Phase 1 (quality) |
| **Active file** | `Kaggle/hebrew-diarization.ipynb` |
| **Blocking next step** | Confirm whether the 3rd detected speaker is real or an artifact |
| **Last updated** | 2026-08-01 (session 3) |

---

## Verified vs unverified

Keep this table honest. As of 2026-08-01 the environment and install are
confirmed on real Kaggle hardware; the transcription and diarization steps are
still unrun.

### Verified

| Claim | How |
|---|---|
| Notebook is valid nbformat 4 JSON | `json.load` + structural assertions |
| All 11 code cells are syntactically valid Python | `ast.parse` per cell, magics stubbed |
| Word-to-speaker overlap assignment is correct | 23 unit tests: clean spans, boundary-straddling words, diarization gaps, leading gaps, empty turn lists, overlapping turns, block grouping |
| txt/srt writers produce correct structure | 10 integration tests with mock segments: cue numbering, arrow format, blank-segment skipping, RLM marks, speaker-name overrides |
| pyannote 4.x uses `token=` not `use_auth_token=` | Read from the community-1 model card |
| community-1 exposes `.exclusive_speaker_diarization` | Same |
| Correct ivrit-ai model id is `ivrit-ai/whisper-large-v3-turbo-ct2` | Read from the model card |
| Fork was identical to upstream at clone time | Both at `d8e7174` |
| **The install cell works on Kaggle** | Run 2026-08-01. `%pip install` completed, all four packages import |
| **cuDNN is compatible — no pin needed** | cuDNN 9.10.2 (`91002`) against ctranslate2 4.8.1, which wants cuDNN 9. They match |
| **float16 is safe on the assigned GPU** | Tesla T4, compute capability 7.5, has FP16 tensor cores |
| Kaggle free tier gives 2x Tesla T4, 15 GB VRAM each | `nvidia-smi`, 2026-08-01 |
| **The notebook runs end to end on Kaggle** | Full run 2026-08-01. 3.5 min WhatsApp screen recording in, txt + srt out, downloaded successfully |
| ffmpeg correctly strips video and keeps audio | The input was an `.mp4` screen recording; cell 6 reported 3.5 min of audio |
| **Throughput is ~9.5x realtime end to end** | 3.5 min audio → 22s total (9s transcribe, 12s diarize) on one T4. Extrapolates to ~6 min per hour of audio |
| Both models fit comfortably on one T4 | No OOM at 15 GB with both loaded |

Environment as actually observed on Kaggle, 2026-08-01:

```
torch          2.10.0+cu128     (driver CUDA 13.0 — newer driver, older runtime, fine)
faster-whisper 1.2.1
pyannote.audio 4.0.7
ctranslate2    4.8.1
cuDNN          9.10.2
GPU            Tesla T4 x2
```

### Unverified — do not present these as working

| Claim | Why it's uncertain |
|---|---|
| Transcription quality on Hebrew | The pipeline produced output; the *accuracy* of that output has not been judged yet. Phase 1. |
| Whether automatic speaker counting is trustworthy | First run detected 3 speakers in a WhatsApp screen recording. Unknown whether that is correct. See open question 1. |
| large-v3-turbo + community-1 fit together in 16 GB VRAM | Should be comfortable on paper (~1.6 GB + ~1 GB), never measured. |
| Hebrew transcription quality is actually better than vanilla Whisper | Claimed by ivrit.ai and third-party comparisons; not measured on the user's own audio. |
| Diarization auto-detects the right speaker count on real interviews | Depends entirely on recording quality. |

---

## Open questions

1. **Is the 3rd speaker in the first test run real?** The recording is a WhatsApp
   screen capture; pyannote reported `SPEAKER_00/01/02`. Two candidate
   explanations, and a cheap test that separates them: print blocks and total
   airtime per speaker. A sub-second single block points at the NaN-embedding
   artifact behind the `std(): degrees of freedom is <= 0` warning; meaningful
   airtime means a genuine third voice, plausibly someone on a phone speaker.
   If artifact, `NUM_SPEAKERS = 2` is the workaround, and a minimum-segment-length
   filter is the real fix.
2. Should the other three Colab notebooks (`Whisper_Audio`, `Whisper_Video`,
   `Whisper_from_Youtube`) get the same treatment, or is diarization the only
   one the user actually needs?
3. Is the WER-check idea from `kaggle-whisper-audio.ipynb` worth porting into
   the diarization notebook, so quality can be measured rather than asserted?
4. Should the tested helper logic move out of notebook cells into a small
   importable module, so it can be tested in CI rather than by extracting cells?
---

## Known debt

None outstanding. The rule 2 / rule 3 cleanup was completed on 2026-07-30 —
see the log entry below.

---

## Log

### 2026-08-01 — Session 3b: Phase 0 complete, full run succeeded

**The notebook works end to end.** A 3.5 minute WhatsApp screen recording went
in; 68 segments / 394 words were transcribed, 114 diarization turns were
merged into 27 speaker blocks, and txt + srt came out and downloaded cleanly.

**The numpy failure was H1, confirmed by the cheapest test.** Cell 7 died with
`ImportError: cannot import name '_center' from 'numpy._core.umath'` — numpy's
Python files and its compiled extension disagreeing, because the kernel had
imported the old numpy before our install upgraded it to 2.5.1. Restarting the
session fixed it. No pin, no force-reinstall, no repo change. That is now twice
in one session that the diagnostic beat the plausible-looking fix.

Consequence: **the notebook needs a documented restart step after the install
cell.** Any user hitting this cold will see the same crash. Not yet actioned.

**Two warnings in cell 9, both benign:**

- *TF32 disabled (ReproducibilityWarning).* pyannote turns TF32 off and suggests
  re-enabling it. On this hardware the suggestion is meaningless: TF32 requires
  Ampere (compute 8.0+), and the T4 is Turing at 7.5. There is no TF32 to enable.
  Do not follow the advice.
- *`std(): degrees of freedom is <= 0`.* A segment collapsed to a single frame
  after downsampling, so its embedding is NaN. Note that the original notebook's
  `np.nan_to_num(embeddings)` was papering over exactly this failure mode. It is
  the leading suspect for the possibly-spurious third speaker.

**Measured throughput: ~9.5x realtime**, 22s for 3.5 min of audio on one T4.
About 6 minutes per hour of audio, so a 2 hour interview is ~13 minutes. The
30 hr/week quota will not be the binding constraint.

### 2026-08-01 — Session 3: first Kaggle run, cuDNN risk resolved

**The cuDNN risk did not materialise.** Diagnostic run before any change, as
rule 1 requires. Result: cuDNN 9.10.2 against ctranslate2 4.8.1, which wants
cuDNN 9 — they match. H1 refuted. H2 refuted too, on hardware grounds: the T4 is
compute capability 7.5 with FP16 tensor cores, so `float16` is natively
supported. H3 confirmed, no change needed.

Worth recording *why* the protocol paid here. The pre-written fix for H1 was to
pin `ctranslate2<4.5`. Applying that speculatively would have **downgraded the
environment into the exact failure H1 described**, since 4.8.1 against cuDNN 9
is already the correct pairing. The diagnostic cost seconds. The speculative fix
would have cost a debugging session and looked like progress while doing it.

Consequence: the `int8_float16` troubleshooting row is now known to be
unnecessary on this hardware. Harmless as a fallback for other GPUs, so it
stays, but it is no longer an open question.

**`import google.colab` is not a valid Kaggle/Colab discriminator.** Kaggle's
current image ships the `google-colab` package *and* a `/content` directory, so
that import succeeds on Kaggle. A platform test built on it reported "Google
Colab" on a machine that was definitively Kaggle, and cost a round trip.
Reliable signals: `/kaggle` existing, or `KAGGLE_KERNEL_RUN_TYPE` in the
environment. The `/kaggle exists: True` line in the same output was correct and
was the one to trust.

**The pip dependency-conflict wall is noise.** Those conflicts are between stock
image packages (`bigframes`, `dopamine-rl`, `ydata-profiling`, `google-colab`)
and predate our install — nothing in our chain is named. The one exception is
numpy 2.5.1 vs numba, now in the unverified table.

**Process note.** Editing this file with a PowerShell
`Get-Content -Raw` / `Set-Content` round trip corrupted it: PS 5.1 reads a
BOM-less file as Windows-1252, so writing it back double-encoded every em-dash
and the Hebrew RLM character. Reverted with `git checkout` and redone with the
editor. Use the file-editing tool on these documents, not shell round trips.

### 2026-07-30 — Session 2: rules 2 and 3, and the cleanup pass

**Rules added** (full text in `CLAUDE.md`)

- Rule 2, prefer the simpler and shorter solution. Includes the explicit
  provision that "more extensible" or "more general" is not an acceptable
  justification for the longer option, and that a rejected shorter alternative
  should be named when proposing work.
- Rule 3, comment the reasoning rather than the mechanics, pitched at an amateur
  coder. Capped deliberately: a paragraph above every line would itself violate
  rule 2.

**Cleanup pass — notebook brought in line with both**

Rule 2, two config toggles deleted:

- `WRITE_SRT` and `RTL_MARKS` gated behaviour nobody would realistically switch
  off. Removing them took out two config lines, a conditional expression
  (`RLM = "‏" if RTL_MARKS else ""`) and an `if` branch in the writer loop.
  The SRT file and the RTL marks are now simply always produced.

Rule 3, reasoning added to the four cells the audit marked weak or poor:

| Cell | What the comments now explain |
|---|---|
| Config | Why `/kaggle/working` specifically; why `language` is passed explicitly (ivrit-ai's language *detection* is degraded by its Hebrew training); what the three speaker knobs actually differ in; that speaker labels are arbitrary until you read the preview |
| Find files | Why video extensions are accepted; why `rglob` and not `listdir` (zip uploads keep their folder structure); why both checks fail loudly rather than yielding a confusing empty result later |
| ffmpeg prep | Why 16 kHz mono rather than restating the flags — both models want it and would convert anyway, so doing it once removes the manual conversion the original notebook demanded *and* guarantees both models see identical audio, so their timestamps line up; why a temp dir rather than the output folder |
| Load models | What `compute_type` means and why float16 on GPU, int8 on CPU |

Also improved: the RLM constant now explains what U+200F is and why every line
needs it (timestamps mix digits into Hebrew text), and `write_srt` explains why
its cues follow Whisper segments rather than the regrouped speaker blocks — a
block can be a minute long, which reads fine on a page but is useless as a
subtitle.

**Verification**

All 33 tests still pass unchanged (23 helper, 10 output), plus the nbformat and
per-cell `ast.parse` checks. The pass touched comments, two config lines and one
branch, so no test needed changing — which is the point of having had them.

Net diff: +66 / -21. Longer in lines because comments were the deliverable;
shorter in moving parts, which is what rule 2 actually asks for.

One stale reference was caught on review: the troubleshooting table still told
users to check `RTL_MARKS = True` after the variable had been deleted. Rewritten
to describe the behaviour instead. Confirmed zero remaining occurrences of either
removed name.

### 2026-07-30 — Session 1

**Done**

- Cloned the fork to `C:\Users\andre\Claude\hebrewWhisper`; added `upstream`.
  Fork was in sync with upstream, both at `d8e7174` (April 2025).
- Audited `Colab/Whisper_Speaker_Diarization.ipynb` cell by cell. Found 7
  concrete defects plus one architectural limitation. Root cause of total
  failure is the torch pin in cell 6. Details in `CLAUDE.md`.
- Established that local execution is impossible: AMD iGPU, no CUDA, 6 GB RAM.
  This is why Kaggle was chosen over a local rewrite.
- Wrote `Kaggle/hebrew-diarization.ipynb` (24 cells) and committed as `7336155`,
  along with a README pointer to it.

**Decisions and rationale**

- *Kaggle over Colab / Lightning / HF Spaces.* Kaggle gives a predictable ~30
  GPU-hr/week counter rather than Colab's opaque quota, the repo already had a
  `Kaggle/` folder to match, and no Drive mount is needed. HF Spaces free tier is
  CPU-only, too slow for large-v3.
- *New notebook rather than repairing the Colab one.* Keeps the original as a
  reference and avoids diverging from upstream inside shared files.
- *pyannote community-1 over the original ECAPA + AgglomerativeClustering.* The
  old method assigns exactly one speaker per Whisper segment, so a speaker change
  mid-segment is structurally unrepresentable — which is most of what happens in
  a real interview. It also required knowing the speaker count in advance.
  Cost of the change: pyannote is gated, so users need a HF signup and token.
- *Word-level overlap matching.* Ask Whisper for word timestamps, match each word
  to the diarization turn it overlaps most, then regroup consecutive same-speaker
  words. This is what makes mid-segment speaker changes representable.
- *Do not pin torch.* This is the lesson of the original's failure, promoted to a
  standing rule in `CLAUDE.md`.

**Bugs found and fixed during testing** (both caught by tests, not by reading)

- SRT timestamps were off by one millisecond on essentially every cue:
  `int((2.4 - 2) * 1000)` truncates to 399, not 400. Rewritten to compute in
  integer milliseconds throughout.
- Speaker tie-break was wrong for overlapping turns. When a word overlapped two
  turns equally, the longer, less specific turn won. Now the tighter turn wins.
  Only reachable via the `.speaker_diarization` fallback path, since the
  exclusive view is non-overlapping — but that fallback exists, so it mattered.

**Not done**

- Nothing pushed to GitHub. Commit `7336155` is local only, `main` is 1 ahead of
  `origin/main`. The fork is public, so pushing is the user's call.
- No real execution. See the unverified table above.

**New rules established by the user this session**

- Hypotheses before bug fixes, with an explicit test plan, before touching the
  repo. (`CLAUDE.md` rule 1)
- Prefer the simpler, shorter solution; justify the longer one if chosen.
  (`CLAUDE.md` rule 2)
- Comment the reasoning, not the mechanics, aimed at an amateur coder.
  (`CLAUDE.md` rule 3)
- Maintain `CLAUDE.md` and this file, updated periodically.

Rules 2 and 3 arrived after `hebrew-diarization.ipynb` was written, so the
notebook does not yet fully satisfy them. Gaps recorded under Known debt above
rather than quietly fixed, since that is a scoped piece of work of its own.
