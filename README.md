<div dir="rtl">

# תמלול אודיו ווידאו בעברית

מחברות (notebooks) לתמלול אוטומטי של הקלטות בעברית, עם אפשרות לזיהוי דוברים.
רצות בחינם על [Kaggle](https://www.kaggle.com/) — אין צורך במחשב חזק, בכרטיס מסך או בהתקנה כלשהי.

מבוסס על [המאגר המקורי](https://github.com/Sourasky-DHLAB/Whisper) של
[הספרייה המרכזית ע"ש סוראסקי](https://cenlib.tau.ac.il/), אוניברסיטת תל אביב (עודד זרחיה),
שנכתב מחדש ב-2026 עבור Kaggle ועם מודלים עדכניים.

</div>

---

<div dir="rtl">

## איזו מחברת לבחור?

</div>

<table>
<thead>
<tr><th></th><th>תמלול בלבד</th><th>תמלול + זיהוי דוברים</th></tr>
</thead>
<tbody>
<tr>
  <td><b>המחברת</b></td>
  <td><a href="Kaggle/hebrew-transcription.ipynb">hebrew-transcription.ipynb</a></td>
  <td><a href="Kaggle/hebrew-diarization.ipynb">hebrew-diarization.ipynb</a></td>
</tr>
<tr>
  <td><b>פתיחה ב-Kaggle</b></td>
  <td><a href="https://kaggle.com/kernels/welcome?src=https://github.com/ablolarof/hebrewWhisper/blob/main/Kaggle/hebrew-transcription.ipynb"><img src="https://kaggle.com/static/images/open-in-kaggle.svg" alt="Open In Kaggle"/></a></td>
  <td><a href="https://kaggle.com/kernels/welcome?src=https://github.com/ablolarof/hebrewWhisper/blob/main/Kaggle/hebrew-diarization.ipynb"><img src="https://kaggle.com/static/images/open-in-kaggle.svg" alt="Open In Kaggle"/></a></td>
</tr>
<tr>
  <td><b>מתאים ל</b></td>
  <td>הרצאות, נאומים, פודקאסטים, הקלטות אישיות — כל מצב שבו לא חשוב מי מדבר</td>
  <td>ראיונות, שיחות, דיונים, פגישות — כשצריך לדעת מי אמר מה</td>
</tr>
<tr>
  <td><b>הרשמה נדרשת</b></td>
  <td><b>אין.</b> רק חשבון Kaggle חינמי</td>
  <td>גם חשבון <a href="https://huggingface.co">Hugging Face</a> חינמי</td>
</tr>
<tr>
  <td><b>מהירות</b></td>
  <td>כ-4 דקות לכל שעת אודיו</td>
  <td>כ-6 דקות לכל שעת אודיו</td>
</tr>
<tr>
  <td><b>הפלט</b></td>
  <td>תמליל רציף, מחולק לפסקאות</td>
  <td>תמליל מחולק לפי דוברים</td>
</tr>
</tbody>
</table>

<div dir="rtl">

**בספק?** התחילו ב-**hebrew-transcription** — היא פשוטה יותר ואינה דורשת דבר מלבד חשבון Kaggle.
תמיד אפשר לעבור אחר כך.

</div>

---

<div dir="rtl">

## התחלה מהירה

### שלב 1 — חשבון Kaggle

1. הירשמו ב-[kaggle.com](https://www.kaggle.com/) (חינם).
2. **אמתו את מספר הטלפון שלכם**: לחצו על תמונת הפרופיל ← Settings ← Phone Verification.
   בלי אימות טלפון המחברת לא תוכל להוריד את מודל התמלול מהאינטרנט, וזהו השלב שהכי הרבה
   אנשים מפספסים.

### שלב 2 — פתיחת המחברת

לחצו על כפתור **Open In Kaggle** בטבלה למעלה. המחברת תיפתח בחשבון שלכם.

### שלב 3 — הפעלת מעבד גרפי ואינטרנט

בסרגל הימני, תחת **Session options**:

- **Accelerator** ← `GPU T4 x2`
- **Internet** ← `On`

> **חשוב:** בדקו זאת גם אם נדמה שזה כבר מוגדר. Kaggle לעיתים קרובות מתעלם מהבקשה של המחברת
> ומפעיל אותה ללא מעבד גרפי. שינוי ההגדרה מאתחל את הסשן, לכן עשו זאת **לפני** שאתם מריצים משהו.

### שלב 4 — העלאת ההקלטות

בסרגל הימני: **Add Input** ← **Upload** ← **New Dataset** ← גררו את הקבצים ← **Create**.

**תנו לו איזה שם שתרצו.** המחברת סורקת את כל מאגרי הנתונים המחוברים אליה ומוצאת את הקבצים
בעצמה — אין צורך להקליד נתיב או לזכור שם.

### שלב 5 — הרצה

הריצו את התאים אחד אחרי השני (`Shift+Enter` בכל תא).

</div>

> ### ⚠️ שלב האתחול — קרו את זה
>
> <div dir="rtl">
>
> בהרצה ראשונה בסשן חדש, תא ההתקנה יסיים בהודעה גדולה שמתחילה ב-**`STOP`**.
> זה **תקין וצפוי**. עשו בדיוק מה שכתוב שם:
>
> 1. **Run ← Restart Session**
> 2. הריצו שוב מהתא הראשון
>
> בפעם השנייה ההתקנה מהירה (הכול כבר שמור), וזה קורה פעם אחת בלבד לכל סשן.
>
> **למה?** התקנת הספריות מעדכנת את `numpy`, אבל Kaggle כבר טען את הגרסה הישנה לזיכרון לפני
> שהקוד שלכם התחיל לרוץ. האתחול פשוט טוען מחדש את הגרסה הנכונה. אם תתעלמו מההודעה, המחברת
> לא תיכשל מיד — היא תיכשל ארבעה תאים אחר כך עם שגיאה שנראית לגמרי לא קשורה.
>
> </div>

---

<div dir="rtl">

## הגדרה נוספת — רק למחברת זיהוי הדוברים

מודל זיהוי הדוברים חינמי, אך דורש חשבון ואישור חד-פעמי.

### א. חשבון Hugging Face

הירשמו ב-[huggingface.co/join](https://huggingface.co/join) (חינם).

### ב. אישור תנאי השימוש במודל

היכנסו לעמוד [pyannote/speaker-diarization-community-1](https://huggingface.co/pyannote/speaker-diarization-community-1)
**כשאתם מחוברים** ולחצו על **Agree and access repository**.

> זהו השלב שהכי קל לפספס. אם המחברת נכשלת בשגיאת 401, כמעט תמיד הסיבה היא שהתנאים לא אושרו —
> ולא שהטוקן שגוי.

### ג. יצירת טוקן

היכנסו ל-[huggingface.co/settings/tokens](https://huggingface.co/settings/tokens),
צרו טוקן מסוג **Read**, והעתיקו אותו.

### ד. שמירת הטוקן ב-Kaggle

במחברת: **Add-ons** ← **Secrets** ← **Add a new secret**

- שם: `HF_TOKEN` (בדיוק כך)
- ערך: הטוקן שהעתקתם

**סמנו את התיבה ליד הסוד** כדי לחבר אותו למחברת. יצירת הסוד לבדה אינה מספיקה — בלי הסימון
המחברת לא רואה אותו.

</div>

---

<div dir="rtl">

## הגדרות

כל מה שכדאי לשנות נמצא בתא הגדרות אחד, בסמוך לראש המחברת.

</div>

<table>
<thead><tr><th>הגדרה</th><th>ברירת מחדל</th><th>מה זה עושה</th></tr></thead>
<tbody>
<tr><td><code>INPUT_DIR</code></td><td><code>None</code></td>
<td>השאירו <code>None</code> והמחברת תמצא את הקבצים לבד. הגדירו נתיב רק אם מחוברים כמה מאגרי נתונים ואתם רוצים לתמלל רק אחד מהם. אפשר גם נתיב לקובץ בודד.</td></tr>
<tr><td><code>LANGUAGE</code></td><td><code>"he"</code></td>
<td>שפת התמלול. <code>he</code> עברית, <code>ar</code> ערבית, <code>en</code> אנגלית. לשפות שאינן עברית החליפו גם את <code>WHISPER_MODEL</code>.</td></tr>
<tr><td><code>WHISPER_MODEL</code></td><td><code>ivrit-ai/whisper-large-v3-turbo-ct2</code></td>
<td>מודל התמלול. ברירת המחדל מכווננת לעברית. לשפות אחרות השתמשו ב-<code>large-v3</code>.</td></tr>
<tr><td><code>OUTPUT_FORMATS</code></td><td><code>"all"</code></td>
<td><code>"all"</code> לכל הפורמטים, או רשימה של מה שצריך: <code>["txt", "srt"]</code>.</td></tr>
<tr><td><code>TXT_TIMESTAMPS</code></td><td><code>True</code></td>
<td><code>False</code> מפיק תמליל רץ ונקי, בלי סימוני <code>[00:00:00]</code>. משפיע רק על קובץ ה-txt.</td></tr>
<tr><td colspan="3"><b>רק במחברת זיהוי הדוברים</b></td></tr>
<tr><td><code>NUM_SPEAKERS</code></td><td><code>None</code></td>
<td>השאירו <code>None</code> והמודל יזהה בעצמו כמה דוברים יש — בדרך כלל הוא מדייק. הגדירו מספר רק אם הזיהוי האוטומטי שגוי.</td></tr>
<tr><td><code>MIN_SPEAKERS</code> / <code>MAX_SPEAKERS</code></td><td><code>None</code></td>
<td>רמז עדין יותר מ-<code>NUM_SPEAKERS</code> — טווח במקום מספר מדויק. נעלמים אם <code>NUM_SPEAKERS</code> מוגדר.</td></tr>
<tr><td><code>SPEAKER_NAMES</code></td><td><code>{}</code></td>
<td>שמות אמיתיים במקום <code>SPEAKER_00</code>. הריצו פעם אחת, הביטו בתצוגה המקדימה כדי לראות מי זה מי, ואז מלאו: <code>{"SPEAKER_00": "מראיין"}</code>.</td></tr>
</tbody>
</table>

---

<div dir="rtl">

## פורמטים של פלט

הקבצים נשמרים ב-`/kaggle/working/transcriptions/`. להורדה: לחצו על **Output** בסרגל הימני.

</div>

<table>
<thead><tr><th>פורמט</th><th>למה הוא טוב</th></tr></thead>
<tbody>
<tr><td><code>txt</code></td><td>תמליל לקריאה. במחברת התמלול מחולק לפסקאות; במחברת הדוברים מחולק לפי דובר. זהו הפורמט לקריאה, לעריכה ולציטוט.</td></tr>
<tr><td><code>srt</code></td><td>כתוביות לנגני וידאו (VLC, YouTube, Premiere). קטע לכל משפט.</td></tr>
<tr><td><code>vtt</code></td><td>כתוביות לאינטרנט (HTML5 video).</td></tr>
<tr><td><code>tsv</code></td><td>שורה לכל קטע, מופרד בטאבים. נפתח ישירות ב-Excel. הזמנים במילישניות כדי שאפשר יהיה לחשב איתם.</td></tr>
<tr><td><code>json</code></td><td>כל המידע במבנה מסודר, לעיבוד בתוכנה אחרת. כולל גם את החלוקה לקריאה וגם את הקטעים הגולמיים.</td></tr>
</tbody>
</table>

<div dir="rtl">

### דוגמה לפלט txt עם זיהוי דוברים

</div>

```
‏[00:00:00] דובר 1:
‏שלום וברוכים הבאים לתוכנית.

‏[00:00:14] דובר 2:
‏תודה רבה שהזמנתם אותי.
```

<div dir="rtl">

### דוגמה לפלט txt ללא חותמות זמן (`TXT_TIMESTAMPS = False`)

</div>

```
‏דובר 1:
‏שלום וברוכים הבאים לתוכנית.

‏דובר 2:
‏תודה רבה שהזמנתם אותי.
```

---

<div dir="rtl">

## מה לצפות מבחינת זמנים

נמדד על Kaggle עם מעבד גרפי מסוג Tesla T4:

</div>

<table>
<thead><tr><th>אורך ההקלטה</th><th>תמלול בלבד</th><th>תמלול + דוברים</th></tr></thead>
<tbody>
<tr><td>5 דקות</td><td>~20 שניות</td><td>~30 שניות</td></tr>
<tr><td>שעה</td><td>~4 דקות</td><td>~6 דקות</td></tr>
<tr><td>שעתיים</td><td>~8 דקות</td><td>~13 דקות</td></tr>
</tbody>
</table>

<div dir="rtl">

בנוסף, בהרצה ראשונה יש כ-2-3 דקות להתקנת הספריות והורדת המודל.

Kaggle מעניק כ-30 שעות מעבד גרפי בשבוע בחינם, וסשן בודד יכול לרוץ עד 12 שעות. בקצב הזה
המכסה השבועית מספיקה לעשרות שעות של הקלטות.

</div>

---

<div dir="rtl">

## פתרון תקלות

</div>

<table>
<thead><tr><th>מה רואים</th><th>מה זה ומה עושים</th></tr></thead>
<tbody>
<tr><td><code>nvidia-smi: command not found</code><br>או הודעה על היעדר GPU</td>
<td>לא הוקצה מעבד גרפי. Session options ← Accelerator ← <code>GPU T4 x2</code>. שינוי ההגדרה מאתחל את הסשן. אם האפשרויות אינן זמינות — נגמרה המכסה השבועית, שמתאפסת אחת לשבוע.</td></tr>

<tr><td>הודעת <code>STOP</code> אחרי ההתקנה</td>
<td><b>תקין וצפוי.</b> Run ← Restart Session, ואז הריצו שוב מהתא הראשון. קורה פעם אחת לכל סשן.</td></tr>

<tr><td><code>ImportError: cannot import name '_center'</code></td>
<td>אותה בעיה, אחרי שהתעלמתם מהודעת ה-<code>STOP</code>. Run ← Restart Session והריצו מחדש מלמעלה.</td></tr>

<tr><td>קיר אדום של <code>ERROR: pip's dependency resolver...</code></td>
<td><b>אפשר להתעלם.</b> ה-image של Kaggle מכיל מאות חבילות שכבר לא תואמות זו לזו, ו-pip מדווח על כולן אחרי כל התקנה. הסימן שההתקנה הצליחה הוא טבלת הגרסאות הקטנה שמודפסת בסוף התא.</td></tr>

<tr><td><code>No audio or video files found</code></td>
<td>לא חובר מאגר נתונים, או שאין בו קבצי מדיה. השגיאה מפרטת מה כן מחובר. Add Input ← Upload ← New Dataset.</td></tr>

<tr><td>שגיאת <code>401</code> או <code>Could not load pyannote/...</code></td>
<td>כמעט תמיד: לא אישרתם את תנאי השימוש. היכנסו ל<a href="https://huggingface.co/pyannote/speaker-diarization-community-1">עמוד המודל</a> כשאתם מחוברים ולחצו <b>Agree and access repository</b>. אם אישרתם — ודאו שהסוד <code>HF_TOKEN</code> <b>מסומן</b> ומחובר למחברת.</td></tr>

<tr><td>הזיהוי מצא יותר מדי דוברים</td>
<td>רעש רקע או מוזיקה. הגדירו <code>MAX_SPEAKERS</code>, או <code>NUM_SPEAKERS</code> אם אתם יודעים בוודאות.</td></tr>

<tr><td>הזיהוי מצא מעט מדי דוברים</td>
<td>קולות דומים, או דובר שנשמע דרך רמקול טלפון. הגדירו <code>NUM_SPEAKERS</code> למספר הנכון.</td></tr>

<tr><td>אזהרת <code>TensorFloat-32 (TF32) has been disabled</code></td>
<td><b>אפשר להתעלם, ואל תפעילו אותה בחזרה.</b> TF32 קיים רק במעבדים גרפיים מדור Ampere ומעלה; ה-T4 של Kaggle ישן יותר ואין לו TF32 כלל, כך שההצעה חסרת משמעות כאן.</td></tr>

<tr><td>אזהרת <code>std(): degrees of freedom is &lt;= 0</code></td>
<td>אפשר להתעלם. קטע אודיו קצר במיוחד. לא נמצאה לכך השפעה על התוצאה.</td></tr>

<tr><td>העברית מוצגת הפוך</td>
<td>פתחו את הקובץ בעורך שתומך בכיווניות: Word, VS Code, Google Docs. הפלט כולל סימני RTL בלתי נראים שרוב התוכנות מכבדות — Notepad אינו אחת מהן.</td></tr>

<tr><td>איכות התמלול ירודה</td>
<td>ודאו ש-<code>LANGUAGE = "he"</code>. בנוסף, איכות ההקלטה היא הגורם המשמעותי ביותר: מיקרופון קרוב ומעט רעש רקע משנים את התוצאה יותר מכל הגדרה.</td></tr>

<tr><td>הסשן נסגר באמצע</td>
<td>Kaggle מגביל סשן ל-12 שעות. תמללו פחות קבצים בכל הרצה.</td></tr>
</tbody>
</table>

---

## How it works

For anyone who wants to know what happens between pressing Run and getting a transcript.

**1. Audio preparation.** Every input is converted with ffmpeg to 16 kHz mono WAV. Both models expect
that format and would convert internally anyway, but doing it once up front means any audio or video
format works with no manual preparation, and both models see byte-identical audio — which matters
because their timestamps are matched against each other later.

**2. Transcription.** [faster-whisper](https://github.com/SYSTRAN/faster-whisper), a CTranslate2
reimplementation of Whisper that runs roughly 4× faster than the reference version and uses less
VRAM. The model is [`ivrit-ai/whisper-large-v3-turbo-ct2`](https://huggingface.co/ivrit-ai/whisper-large-v3-turbo-ct2),
a Hebrew finetune from [ivrit.ai](https://www.ivrit.ai/). Its language *detection* was degraded by
that training, so the language is always passed explicitly rather than autodetected.

**3. Speaker diarization** (diarization notebook only).
[`pyannote/speaker-diarization-community-1`](https://huggingface.co/pyannote/speaker-diarization-community-1)
segments the audio by speaker. It works out the number of speakers by itself and, unlike simpler
approaches, can detect a speaker change in the middle of a sentence.

**4. Matching words to speakers.** Whisper is asked for word-level timestamps. Each word is assigned
to the diarization turn it overlaps most, and consecutive words with the same speaker are then merged
into readable blocks. This is what allows a speaker change mid-sentence to be represented at all — an
approach that assigns one speaker per Whisper segment structurally cannot express it.

**5. Output.** Readable formats (`txt`) use the grouped blocks or paragraphs; subtitle and data
formats (`srt`, `vtt`, `tsv`, `json`) follow Whisper's raw segments, which are the right size for a
subtitle cue or a spreadsheet row.

### Why Kaggle

Free GPU access with a predictable quota (~30 h/week, shown as a counter), 16 GB of VRAM, sessions up
to 12 hours, and no Google Drive mount to authorise. Google Colab's free tier has an opaque, variable
allowance; Hugging Face Spaces' free tier is CPU-only and too slow for a large model.

### Notes for developers

- **`torch` is never pinned or installed.** Kaggle ships a PyTorch build matched to its own CUDA
  driver. Overriding it is exactly what broke the original notebooks: they pinned `torch==1.13.1`,
  whose wheels only exist for Python ≤ 3.10, against an image running 3.11+.
- **Pinning is not free here.** An exact pin can send pip into a long backtracking search across the
  image's hundreds of packages. Repeated identical download sizes in pip's output mean backtracking,
  not progress.
- **The two notebooks duplicate shared logic** — ffmpeg prep, file discovery, timestamp formatting,
  output writing — rather than importing a common module. A Kaggle notebook has to be self-contained;
  a module would force users to clone a repository before running anything. The cost is that a fix to
  shared logic must be applied to both files.

---

<div dir="rtl">

## קרדיטים

- המחברות המקוריות: [הספרייה המרכזית ע"ש סוראסקי](https://cenlib.tau.ac.il/), אוניברסיטת תל אביב — עודד זרחיה
- מודל התמלול בעברית: [ivrit.ai](https://www.ivrit.ai/)
- מנוע התמלול: [OpenAI Whisper](https://github.com/openai/whisper) ו-[faster-whisper](https://github.com/SYSTRAN/faster-whisper)
- זיהוי דוברים: [pyannote.audio](https://github.com/pyannote/pyannote-audio)

הגרסה הקודמת של המאגר, כולל מחברות ה-Colab המקוריות, שמורה בענף
[`v1-legacy`](https://github.com/ablolarof/hebrewWhisper/tree/v1-legacy).

</div>
