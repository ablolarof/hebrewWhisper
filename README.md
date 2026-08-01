<div dir="rtl">
<p align="center"><img style="display: block; margin-left: auto; margin-right: auto;" src="https://github.com/Sourasky-DHLAB/Whisper/blob/main/Resources/Whisper4.png" /></p>
<h1 id="תמלול-אוטומטי-של-וידאו-ואודיו-באמצעות-whisper">תמלול אוטומטי של וידאו ואודיו באמצעות Whisper</h1>
<p><a href="https://openai.com/blog/whisper">וויספר</a> (Whisper) היא מערכת לזיהוי דיבור (ASR: Automatic Speech Recognition) מבית <a href="https://openai.com">OpenAI</a> הזמינה לציבור הרחב בקוד פתוח. מערכת זו אומנה על יותר מ-680 אלף שעות של אודיו באנגלית ובשפות רבות אחרות &ndash; בהן גם עברית וערבית. מטרת מחברות אלו היא להנגיש את יכולות התמלול של המערכת לציבור הרחב בצורה אינטואיטיבית ונוחה.</p>
<p>על אף שוויספר&nbsp;מיועדת בעיקר לתמלול קבצי אודיו, המערכת יכולה לעבוד גם עם סוגים אחרים של קלט דיבור, כגון נתוני וידאו המכילים דיבור.&nbsp;באופן כללי, המערכת יכולה לקבל כל סוג של קלט אודיו או דיבור בפורמט דיגיטלי שנתמך על ידי ספריית <a href="https://ffmpeg.org/about.html">ffmpeg4</a>, ובכלל זה קבצים בפורמט&nbsp;WAV, MP3, MP4 ו-MOV.</p>
<blockquote>
<h3>עדכון 2026 — מחברות Kaggle חדשות</h3>
<p>מחברות ה-Colab שבמאגר זה <b>אינן פועלות עוד</b>. תא ההתקנה שלהן מקבע את <code>torch==1.13.1</code> יחד עם <code>torchvision==0.14.1</code> ו-<code>torchaudio==0.13.1</code>, וגרסאות אלו קיימות רק עבור Python 3.10 ומטה — בעוד ש-Colab ו-Kaggle עברו מאז ל-Python 3.11 ומעלה. כתוצאה מכך <code>pip</code> נכשל עוד לפני שקוד התמלול מתחיל לרוץ.</p>
<p>נוספו שתי מחברות חלופיות הרצות על <a href="https://www.kaggle.com/">Kaggle</a> (כ-30 שעות GPU חינם בשבוע):</p>
<ol style="float:right;">
<li><a href="https://github.com/Sourasky-DHLAB/Whisper/blob/main/Kaggle/hebrew-transcription.ipynb">Kaggle/hebrew-transcription.ipynb</a> — תמלול בלבד. <b>אינה דורשת חשבון כלשהו מלבד Kaggle.</b></li>
<li><a href="https://github.com/Sourasky-DHLAB/Whisper/blob/main/Kaggle/hebrew-diarization.ipynb">Kaggle/hebrew-diarization.ipynb</a> — תמלול וזיהוי דוברים. דורשת גם חשבון Hugging Face חינמי.</li>
</ol>
<p>עיקרי השינויים לעומת המחברות הישנות:</p>
<ul style="float:right;">
<li><b>אינן מקבעות את <code>torch</code></b> — משתמשות בגרסה שהפלטפורמה מספקת, וזו הסיבה שהן ימשיכו לעבוד</li>
<li>מודל <a href="https://huggingface.co/ivrit-ai/whisper-large-v3-turbo-ct2">ivrit-ai/whisper-large-v3-turbo-ct2</a> שכוונן לעברית, על גבי <a href="https://github.com/SYSTRAN/faster-whisper">faster-whisper</a> (מהיר פי ~4)</li>
<li>זיהוי דוברים ב-<a href="https://huggingface.co/pyannote/speaker-diarization-community-1">pyannote community-1</a>, המזהה בעצמו כמה דוברים יש ותומך בהחלפת דובר באמצע משפט</li>
<li>מקבלות כל פורמט אודיו/וידאו — אין צורך בהמרה ידנית ל-WAV</li>
<li>מאתרות את קבצי הקלט בעצמן, ללא צורך בהקלדת נתיב</li>
<li>חמישה פורמטי פלט: <code>txt</code>, <code>srt</code>, <code>vtt</code>, <code>tsv</code>, <code>json</code></li>
</ul>
<p>נבדקו על Kaggle: כ-4 דקות לכל שעת אודיו בתמלול בלבד, כ-6 דקות עם זיהוי דוברים.</p>
<p>מחברות ה-Colab נותרו במאגר כפי שהן ולא שונו.</p>
</blockquote>

<h2 id="שימוש-נכון-במחברות">שימוש נכון במחברות</h2>
<p>כדי להשתמש במחברות יש להיעזר ב-Google Colab, כלי שמאפשר לנו לצפות ולהריץ את המחברות שהכנו עבורכם מראש. כדי לפתוח מחברת בסביבת Google Colab יש ללחוץ על הכפתור הבא שנמצא בראשית כל מחברת:</p>
<p align="center"><img src="https://github.com/Sourasky-DHLAB/Whisper/blob/main/Resources/colab.png" /></p>
<h2 id="סוגי-מחברות-במאגר">סוגי מחברות במאגר</h2>
<p>מאגר (Repository) זה מכיל מחברות לשימושים שונים:</p>
<div dir="rtl">1. <a href="https://github.com/Sourasky-DHLAB/Whisper/blob/main/Colab/Whisper_Audio.ipynb">Whisper_Audio.ipynb</a>: מחברת לתמלול קבצי אודיו או וידאו (ישירות - ללא צורך בחילוץ שכבת האודיו). למתחילים מומלץ להתחיל עם מחברת זו.<br />2. <a href="https://github.com/Sourasky-DHLAB/Whisper/blob/main/Colab/Whisper_Video.ipynb">Whisper_Video.ipynb</a>: מחברת זו מאפשרת לתמלל קבצי וידאו תוך חילוץ שכבת האודיו. לאחר מכן ניתן להשוות את איכות הפלט אל מול המקור.<br />3. <a href="https://github.com/Sourasky-DHLAB/Whisper/blob/main/Colab/Whisper_from_Youtube.ipynb">Whisper_from_Youtube.ipynb</a>: מחברת זו מאפשרת להוריד ולתמלל סרטונים מ-Yotube.<br />4. <a href="https://github.com/Sourasky-DHLAB/Whisper/blob/main/Colab/Whisper_Speaker_Diarization.ipynb">Whisper_Speaker_Diarization.ipynb</a>: מחברת לתמלול ראיונות וזיהוי דוברים.</div>
<div dir="rtl">&nbsp;</div>
<div dir="rtl">בתוך תיקיית <a href="https://github.com/Sourasky-DHLAB/Whisper/tree/main/OtherASRs">OtherASRs</a> ניתן למצוא 2 מחברות נוספות לתמלול:</div>
<div dir="rtl">1. <a href="https://github.com/Sourasky-DHLAB/Whisper/blob/main/OtherASRs/ASR_SpeechBrain_Amharic.ipynb">ASR_SpeechBrain_Amharic.ipynb</a>: מחברת לתמלול מהשפה האמהרית. מחברת זו, העושה שימוש ב-<a href="https://speechbrain.github.io/">SpeechBrain</a>, <a href="https://huggingface.co/docs/transformers/model_doc/wav2vec2">Wav2Vec2</a> ו-<a href="https://github.com/jiaaro/pydub">Pydub</a>, מאפשרת תמלול של קבצי WAV בלבד ונועדה לשימוש מתקדם.</div>
<div dir="rtl">2. <a href="https://github.com/Sourasky-DHLAB/Whisper/blob/main/OtherASRs/fairseq_meta_mms.ipynb">fairseq_meta_mms.ipynb</a>: מחברת לתמלול המבוססת על פרויקט ה-<a href="https://github.com/facebookresearch/fairseq/blob/main/examples/mms/README.md">MMS</a> של חברת Meta שתומך ב-1,000 שפות. מחברת זו מאפשרת תמלול של קבצי WAV בלבד ונועדה לשימוש מתקדם.<br />
<h2 id="דוגמה-מתוך-המחברות">דוגמה מתוך המחברות</h2>
<p align="center"><img src="https://github.com/Sourasky-DHLAB/Whisper/blob/main/Resources/screenshot.png" /></p>
</div>
</div>
