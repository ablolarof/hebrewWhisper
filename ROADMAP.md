# ROADMAP — reviving speaker diarization

Phases are ordered by dependency, not ambition. Phase 0 blocks everything else:
until the notebook is known to run, every later task is built on a guess.

Status lives in `MEMORY.md`. Working rules live in `CLAUDE.md`.

---

## Phase 0 — Prove it runs ✅ COMPLETE (2026-08-01)

**Goal:** one short file goes in, a transcript with speaker labels comes out.
**Blocks:** everything.

**Outcome:** passed. A 3.5 min recording produced txt + srt at ~9.5x realtime.
Two failures were hit and resolved without any repo change — a read-only
`/kaggle/input` misunderstanding, and a stale-numpy `ImportError` fixed by
restarting the session. The cuDNN risk below never materialised.

**Carried forward:** the notebook still lacks a documented "restart the session
after installing" step, which every cold user will need. See Phase 2.4.

| # | Task | Done when |
|---|---|---|
| 0.1 | Kaggle account phone-verified (required for notebook internet access) | The Internet toggle can be switched on |
| 0.2 | HF account created, community-1 terms accepted, read token made | Token exists |
| 0.3 | Token stored as Kaggle Secret named `HF_TOKEN`, attached to the notebook | Cell 3 prints "Token loaded" |
| 0.4 | A **short** test file uploaded as Dataset `audiofiles` | Cell 5 lists it |
| 0.5 | Environment diagnostic run (ctranslate2 / cuDNN / GPU versions) | Output captured |
| 0.6 | Notebook runs start to finish | A `.txt` and `.srt` land in `/kaggle/working/transcriptions/` |

**Use a 2-3 minute clip with two clearly different voices for 0.4.** Not a real
interview. The point is a fast failure loop — a two-hour file turns every
debugging cycle into an hour of waiting, and the first run is the most likely
to fail.

**Exit criteria:** a transcript exists, the Hebrew is readable, and the speaker
turns are roughly in the right places. Accuracy is Phase 1's problem.

### Known risk: cuDNN

The most likely first failure. `faster-whisper` pulls `ctranslate2`, which needs
a cuDNN version matching Kaggle's image. Per `CLAUDE.md` rule 1, the hypotheses
and the test that distinguishes them are written down *before* any fix:

| | Hypothesis | Expected symptom | Fix if confirmed |
|---|---|---|---|
| H1 | ctranslate2 4.x needs cuDNN 9, image has cuDNN 8 | `Could not load library libcudnn_ops_infer.so.8`, thrown at first *transcribe*, not at import | Pin `ctranslate2<4.5` |
| H2 | cuDNN fine, GPU rejects float16 | Error naming the compute type, not a library | `compute_type = "int8_float16"` |
| H3 | Neither — it works | — | — |

Discriminating test, run **before** changing anything:

```python
import ctranslate2, torch
print("ct2  ", ctranslate2.__version__)
print("cudnn", torch.backends.cudnn.version())
print("gpu  ", torch.cuda.get_device_name(0))
```

cuDNN `8xxx` with ctranslate2 4.x confirms H1. cuDNN `9xxx` refutes H1 and
points at H2. This matters: the notebook's troubleshooting table currently
suggests the H2 fix, which does nothing if H1 is the real cause.

---

## Phase 1 — Find out if it is any good

**Goal:** replace "should be accurate" with a measured number.
**Depends on:** Phase 0.

| # | Task | Notes |
|---|---|---|
| 1.1 | Run on a real interview | First contact with messy audio |
| 1.2 | Port the WER check from `Kaggle/kaggle-whisper-audio.ipynb` | Needs a hand-made ground-truth transcript of ~5 minutes. Tedious, and the only way to get a real number |
| 1.3 | Compare `ivrit-ai/whisper-large-v3-turbo-ct2` against plain `large-v3` on the same file | Decides whether the Hebrew finetune earns its place. Currently an assumption |
| 1.4 | Assess automatic speaker counting | Does it get 2 speakers right? When does `NUM_SPEAKERS` become necessary? |
| 1.5 | Record the recommended settings in `CLAUDE.md` | So the answer is not re-derived later |

**Exit criteria:** a WER figure for Hebrew, and a documented default config with
evidence behind it.

---

## Phase 2 — Survive real use

**Goal:** works on a two-hour interview, not just a demo clip.
**Depends on:** Phase 1.

| # | Task | Why |
|---|---|---|
| 2.1 | Test a full-length recording (60-120 min) | Memory ceiling and the 12-hour session limit are untested |
| 2.2 | Cache the transcription step to disk | If diarization or a later cell fails, an hour of GPU time should not be lost |
| 2.3 | Verify multi-file batching | The loop exists but has never run with more than one file |
| 2.4 | Improve error messages using real Phase 0-1 failures | Guesses replaced by things that actually happened |
| 2.5 | Decide the practical length ceiling and document it | Users should know before they start, not after 11 hours |

---

## Phase 3 — Restore the rest of the repo

**Goal:** the fork is usable, not just the one notebook.
**Depends on:** Phase 1. Independent of Phase 2.

| # | Task | Notes |
|---|---|---|
| 3.1 | Kaggle version of `Whisper_Audio.ipynb` | Simplest case, no diarization. Mostly a cut-down of the working notebook |
| 3.2 | Kaggle version of `Whisper_Video.ipynb` | ffmpeg handling already solved in the diarization notebook |
| 3.3 | Decide on `Whisper_from_Youtube.ipynb` | **Flagged:** YouTube blocks datacentre IPs, so `yt-dlp` frequently fails from Kaggle regardless of code quality. Also a ToS question. May be better dropped than half-fixed |
| 3.4 | Rewrite the Hebrew README around the new notebooks | Currently only a banner points at the new work |
| 3.5 | Decide the fate of `Colab/` | Delete, or keep clearly marked as broken reference |

---

## Phase 4 — Optional

Only worth doing if the project outlives its first real use.

| # | Task | Value |
|---|---|---|
| 4.1 | Extract notebook helpers into an importable module with pytest + CI | Tests currently work by scraping cells out of the `.ipynb`, which is fragile |
| 4.2 | Hugging Face Space with a file-upload UI | Removes Kaggle knowledge from the user's path. Free tier is CPU-only, so likely needs a small model or paid ZeroGPU |
| 4.3 | Offer the work upstream to Sourasky-DHLAB | Upstream looks abandoned, so expect no response. Low cost to try |
| 4.4 | Speaker-name assistance | e.g. show a sample utterance per speaker so labelling is guesswork-free |

---

## Cross-cutting risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| cuDNN / ctranslate2 mismatch | High | Hypotheses and test written above, before Phase 0 starts |
| Kaggle changes its base image and breaks the install | Medium, ongoing | Do not pin torch; pin added libraries with `>=` minimums only |
| pyannote gating changes or terms are re-accepted per version | Low-medium | Error message already points at the terms page |
| Hebrew accuracy disappoints on real audio | Medium | Phase 1.3 comparison exists precisely to find this out early |
| 30 GPU-hr/week quota becomes the bottleneck | Medium | Phase 2.2 caching; keep test clips short |
| Kaggle account never gets phone-verified | Low | Blocks everything. Lightning AI Studio is the fallback platform |
