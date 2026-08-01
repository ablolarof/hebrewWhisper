# MEMORY.md — hebrewWhisper project log

Running record of decisions, findings, and open questions. Update at the end of
each working session. Newest entries at the top of the log.

See `CLAUDE.md` for the working rules (notably: hypotheses before bug fixes).

---

## Status at a glance

| | |
|---|---|
| **Phase** | **Phase 0 and 3.1 done; both notebooks feature-complete and verified on Kaggle.** What remains is measurement, not construction |
| **Active files** | `Kaggle/hebrew-diarization.ipynb` (speakers, needs HF token), `Kaggle/hebrew-transcription.ipynb` (no account needed) |
| **Blocking next step** | Nothing is blocked. Next value is Phase 1.2 (WER) and 1.3 (ivrit-ai vs plain large-v3) |
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
| **Automatic speaker counting was correct** | Detected 3 speakers with no hint given; user confirmed the call genuinely had 3. This is the payoff for community-1 over the old fixed-`num_speakers` approach, which would have forced 3 people into however many buckets were guessed |
| **`hebrew-transcription.ipynb` runs end to end on Kaggle** | Confirmed by the user 2026-08-01, the run after it was built. Both notebooks are now proven on real hardware |
| **Automatic input discovery works on Kaggle** | Confirmed 2026-08-01. `INPUT_DIR = None` found the recordings with no path typed |
| **All five output formats are produced** | Confirmed 2026-08-01 with `OUTPUT_FORMATS = "all"`. The writing cell lists each file it creates |
| **The GPU check cell behaves correctly** | Confirmed in both directions: it raised with instructions on a CPU-only session, and passed once the accelerator was set |
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
| Hebrew accuracy as a **number** | User read the first transcript and called it "pretty good" — a real signal, and the first evidence that the Hebrew works at all, but not a measurement. Phase 1.2 (WER against a ground truth) is still the thing that would make this a claim rather than an impression. |
| Whether the ivrit-ai finetune beats plain `large-v3` | Still untested. "Pretty good" does not tell us whether the standard model would have been equally good. Phase 1.3. |
| Speaker counting on *harder* audio | It got 3/3 right on this call. One sample. Two speakers with similar voices, or heavy background noise, are the cases that would break it. |
| That `NUM_SPEAKERS = 1` is accepted by pyannote | Suggested as the single-speaker workaround but never tried. Some clustering implementations reject `n_clusters=1`. Moot now that a dedicated notebook exists, but the claim was made and never checked. |
| Paragraph grouping reads well on *long* audio | The 45s / 2s pause thresholds are unit-tested and were fine on a short file. Whether they produce sensible paragraphs across a two-hour lecture is a judgement call nobody has made yet. |
| **That the STOP box actually appears** | The numpy-change detection is straightforward code but has not run on Kaggle. If `importlib.invalidate_caches()` is insufficient to see the new version from the same process, the box would silently not print and we would be back to the bare ImportError. |
| That `vtt`, `tsv` and `json` load correctly in their **target applications** | The files are written and their structure is unit-tested, but nobody has opened a `.vtt` in a video player, a `.tsv` in Excel, or parsed the `.json` from another program. Format bugs of the kind unit tests miss (encoding, BOM expectations, header quirks) would only show up there. |
| Whether files can actually be dropped into `/kaggle/working/audio` via the UI | The folder is created and searched, so it costs nothing if unsupported, but nobody has confirmed Kaggle's file browser allows uploads there. Datasets remain the documented route. |
| Behaviour when several datasets are attached at once | Auto-discovery searches all of `/kaggle/input`, so unrelated attached datasets containing media would also be transcribed. Not yet seen in practice. |
| large-v3-turbo + community-1 fit together in 16 GB VRAM | Should be comfortable on paper (~1.6 GB + ~1 GB), never measured. |
| Hebrew transcription quality is actually better than vanilla Whisper | Claimed by ivrit.ai and third-party comparisons; not measured on the user's own audio. |
| Diarization auto-detects the right speaker count on real interviews | Depends entirely on recording quality. |

---

## Open questions

1. ~~Is the 3rd speaker in the first test run real?~~ **Resolved 2026-08-01: yes.**
   The call genuinely had 3 participants and pyannote found all 3 unprompted. The
   `std(): degrees of freedom is <= 0` warning was therefore *not* responsible for
   a spurious cluster — it fired on some short segment without affecting the
   outcome. Treat it as noise unless a future run shows a speaker with negligible
   airtime.
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

### 2026-08-01 — Session 7: numpy root cause found and pinned

**The numpy ImportError is not a one-off.** It recurred on a second cold session
of the diarization notebook, which is what justified diagnosing it properly
rather than documenting a restart.

**Diagnosis, confirmed by measurement:**

```
before cell 2:  numpy on disk 2.0.2   already loaded True
after  cell 2:  numpy on disk 2.5.1   already loaded True
```

Kaggle ships numpy **2.0.2**; installing `pyannote.audio` upgrades it to
**2.5.1**. The kernel has numpy loaded *before any notebook code runs* — the
first probe reported `already loaded: True` having imported nothing but `sys`
and `importlib`. So the process holds 2.0.2's compiled extension while 2.5.1's
Python files sit on disk, and the halves disagree.

This also explains the asymmetry: the transcription notebook installs only
faster-whisper and has never hit it. pyannote is what drags numpy forward.

**A hypothesis that the measurement killed.** One candidate fix was to drop the
`import torch` from the install cell, on the theory that our own import was what
loaded numpy first. `already loaded: True` on the very first probe refutes that
— Kaggle preloads numpy at kernel startup, so nothing the notebook does or
avoids doing changes it.

**Fix applied: pin numpy to the version already installed**, dynamically rather
than hardcoding 2.0.2, so a future Kaggle image update does not turn the pin
into a forced downgrade. Applied to both notebooks — the transcription one has
never triggered this, but the pin costs nothing and prevents it starting.

Plus a verification step, which is the part likely to outlast the fix: after
installing, the cell re-reads numpy's version and prints a loud, plain-language
warning if it changed anyway. That converts any future recurrence from an
`ImportError` five cells away into an instruction at the point of cause.
`importlib.invalidate_caches()` is required before that re-read or the value is
stale.

**The pin was then reverted — it was the wrong fix.** On the next cold session
the install appeared to hang and was cancelled after a minute. It was not hung:
pip was **backtracking**. The tell is in the download sizes, `80.3 kB` and
`52.8 kB` each appearing twice, which is metadata for successive candidate
versions being tried. Compare the unpinned run's 35.5 MB / 39.5 MB / 19.2 MB
wheels downloaded once each. Pinning numpy forced a search for some combination
satisfying both `numpy==2.0.2` and `pyannote.audio>=4.0.0`.

The decisive argument does not even require knowing whether that search would
have terminated: **even a five-minute resolve is worse than a thirty-second
restart, and it would happen on every cold session.** The pin optimised away a
minor annoyance and bought a larger one.

**What shipped instead:** no pin, and the version check kept and made loud. The
install runs at full speed, and when it moves numpy the cell prints a STOP box
with numbered steps, at the point of cause rather than four cells downstream.
The restart is now documented as a normal expected step rather than an error.

*Lesson worth keeping:* the restart was never the problem. Its only real fault
was being announced in a markdown cell that is easy to skim past. Making the
message unmissable was the cheap fix available from the start, and two rounds of
cleverness were spent avoiding it.

### 2026-08-01 — Session 6: usability changes verified on Kaggle

A single re-import and run confirmed the whole of sessions 5 and 6 at once:
automatic input discovery, all five output formats, the timestamps switch, the
Kaggle badges, and the GPU check.

The GPU check was confirmed in **both** directions, which is the more useful
result: it raised with instructions on a CPU-only session, then passed once the
accelerator was set. A guard that has only ever been seen not firing is not
really tested.

**Both notebooks are now feature-complete and verified.** Everything the user
asked for is built and working on free hardware. What is left in the unverified
table has narrowed to genuinely secondary questions — whether `vtt`/`tsv`/`json`
behave in their target applications as opposed to being structurally correct,
and what happens with several datasets attached at once.

The project has moved from construction to measurement. Phases 1.2 and 1.3 (word
error rate, and ivrit-ai against plain `large-v3`) are the only items left with
real value, and both are optional.

### 2026-08-01 — Session 5: usability pass on both notebooks

Four user-requested changes, applied to both notebooks.

**1. Input files are found automatically.** `INPUT_DIR = None` is now the
default, and the notebook searches every attached Dataset plus a
`/kaggle/working/audio/` folder it creates. Previously the user had to type a
path that had to match a slugified dataset name exactly — and in practice they
got it wrong twice, once pointing at a file instead of a folder and once using
the website URL form rather than the mount path. Both failures were the design's
fault, not theirs: the notebook had all the information needed to find the files
and was asking anyway. Explicit paths still work for narrowing the search.

**2. Five output formats.** `txt`, `srt`, `vtt`, `tsv`, `json`, selected with
`OUTPUT_FORMATS = "all"` or a list. Unknown names raise immediately with the
valid set rather than silently producing nothing.

**3. Timestamps optional in txt.** `TXT_TIMESTAMPS = False` gives clean prose.
Deliberately scoped to txt only — the subtitle and data formats are *defined* by
their timestamps, so a global switch would be meaningless for four of five.

**4. "Open in Kaggle" badges**, matching the old Colab badges, in both notebooks
and in the README table.

**On rules 2 and 3 pulling in opposite directions here.** Session 2 deleted
`WRITE_SRT` and `RTL_MARKS` as speculative toggles; this session adds
`OUTPUT_FORMATS` and `TXT_TIMESTAMPS`. That is not a reversal. The deleted ones
gated behaviour nobody asked for and nobody would switch off. These come from a
stated need with concrete uses — different downstream tools want different
formats, and a prose transcript reads very differently from a timestamped one.
Demonstrated need versus speculation is precisely the line rule 2 draws.

**Also refactored:** per-segment speaker labels are now computed once after
diarization and passed into the writers, rather than each writer re-deriving
them. With one writer that was invisible; with four it would have been four
passes over the same data.

**Test coverage grew to 110 across five suites** (helpers 23, output 10,
transcription 24, formats 36, discovery 17). The discovery tests build a fake
Kaggle tree in a temp folder and check nested files, ignored extensions,
uppercase extensions, single-file input, and that overlapping search roots do
not produce duplicates.

**Untested on Kaggle.** Every change here is verified only by unit test.

### 2026-08-01 — Session 4b: transcription notebook verified

**`hebrew-transcription.ipynb` runs end to end on Kaggle**, confirmed by the
user on the first attempt. Both notebooks are now proven on real hardware.

That closes the last *structural* unknown in the project. Everything still in
the unverified table is now about **quality** rather than whether things work:
the Hebrew word error rate, whether the ivrit-ai finetune actually beats plain
`large-v3`, whether speaker counting holds up on harder audio, and whether the
paragraph thresholds read well across a two-hour recording.

Worth stating plainly, because it changes what "done" means from here: the
project is **usable today**. Nothing is blocked. Phase 1 is no longer about
making it work, it is about finding out how good it is — which is optional in a
way that Phase 0 never was.

### 2026-08-01 — Session 4: token-free transcription notebook

**Built `Kaggle/hebrew-transcription.ipynb`** — transcription only, no speaker
labels, and critically **no Hugging Face account required**.

*Why it was built at all.* The first answer to "can the diarization notebook
handle a single speaker?" was: yes, set `NUM_SPEAKERS = 1`, and don't build a
second notebook until the need is demonstrated rather than assumed. The user
then gave the one reason that setting cannot address — they want to share this
with other people. A token-free notebook is a different product, not a config
flag: `NUM_SPEAKERS = 1` still requires an HF account, terms acceptance and a
Kaggle secret before anything runs. That justified the build; the earlier
reasoning would still have been right without it.

*What it drops:* pyannote entirely, the token cell, and word-level timestamps
(only needed for matching words to speaker turns). Roughly 4 min per hour of
audio versus 6.

*What it adds:* paragraph grouping. With no speakers to break on, one line per
Whisper segment is a choppy wall of fragments — 68 of them for 3.5 minutes. New
paragraphs start after a 2s pause or once a paragraph exceeds 45s. 24/24 unit
tests, including that a whitespace-only segment cannot silently extend a
paragraph's end timestamp.

**Found an unused dependency in the diarization notebook.** It installed
`srt>=3.5` but never imported it — `write_srt` writes the format by hand.
Removed from both, and the validator now checks that every pip-installed package
is actually referenced, so this cannot recur silently.

**Added the numpy restart note to both notebooks**, clearing carried debt from
session 3b. Written conditionally ("if a later cell fails with...") rather than
as a mandatory step, since forcing a restart on every user for something that may
be specific to a cold session would be worse than explaining it.

**Documented the duplication decision in `CLAUDE.md`.** The two notebooks repeat
ffmpeg prep, file discovery, timestamps and RTL output rather than importing a
shared module, because a Kaggle notebook must be self-contained — a module would
force users to clone a repo first, defeating the purpose of the token-free
notebook. The cost is that shared fixes must be applied twice, and that is now
written where it will be seen.

**Not run on Kaggle.** Its logic is tested and its shared parts are lifted from
the proven notebook, but it has never executed. Recorded as unverified.

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

### 2026-08-01 — Session 3c: first results confirmed, branch merged to main

**Speaker detection was correct.** The call really had 3 participants, and
pyannote found all 3 without being told how many to look for. This is the
clearest vindication so far of replacing the original ECAPA + AgglomerativeClustering
approach: that method took `num_speakers` as a *required input*, so a user who
guessed 2 would have had three people silently forced into two buckets with
nothing in the output indicating a problem.

**Hebrew transcription judged "pretty good" by the user.** First evidence the
Hebrew path works at all. Deliberately *not* moved to the verified table as a
quality claim — an impression from one 3.5 minute recording is not a WER, and
it says nothing about whether plain `large-v3` would have done equally well.
Phases 1.2 and 1.3 remain the things that would turn this into a real claim.

**Merged `revive-diarization` into `main` and pushed.** The working notebook,
CLAUDE.md, MEMORY.md and ROADMAP.md are now on the default branch, which also
fixes Kaggle's GitHub import — it browses the default branch, and previously
found nothing there.

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
