<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>MoodLens: an AI that reads feelings</title>
<meta name="description" content="A small, free AI that reads the mood of a sentence and learns from you.">

<style>
/* =====================================================
   CSS: the look of the website
   ===================================================== */

/* 1. COLORS: change a color here and it changes everywhere */
:root {
  --bg: #f8f5f0;
  --card: #ffffff;
  --text: #1e1a2b;
  --muted: #6a6578;
  --line: #e4ded3;
  --accent: #5a49d6;
  --accent-text: #ffffff;
  --shadow: 0 1px 2px rgba(30, 26, 43, .06), 0 8px 24px rgba(30, 26, 43, .06);

  /* one color for each mood */
  --joy: #b97800;
  --love: #d1337a;
  --surprise: #0c8ca0;
  --sadness: #3a6bd0;
  --anger: #cf3a2a;
  --fear: #7a4cd0;
  --mixed: #6a6578;
}
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #14111d; --card: #1e1a2b; --text: #f1eef8; --muted: #a49fb5;
    --line: #342f45; --accent: #9d8fff; --accent-text: #15103a;
    --shadow: 0 1px 2px rgba(0, 0, 0, .4), 0 8px 24px rgba(0, 0, 0, .3);
    --joy: #ffc247; --love: #ff77ad; --surprise: #4fd3e6;
    --sadness: #7aa2ff; --anger: #ff7d6b; --fear: #b79bff; --mixed: #a49fb5;
  }
}

* { box-sizing: border-box; }
html { -webkit-text-size-adjust: 100%; }
body {
  margin: 0;
  background:
    radial-gradient(900px 400px at 85% -10%, color-mix(in srgb, var(--accent) 16%, transparent), transparent),
    radial-gradient(700px 400px at -10% 0%, color-mix(in srgb, var(--love) 12%, transparent), transparent),
    var(--bg);
  background-attachment: fixed;
  color: var(--text);
  font-family: "Avenir Next", "Segoe UI", system-ui, -apple-system, Roboto, sans-serif;
  line-height: 1.5;
  padding: env(safe-area-inset-top, 0px) env(safe-area-inset-right, 0px)
           env(safe-area-inset-bottom, 0px) env(safe-area-inset-left, 0px);
}

/* 2. LAYOUT */
.wrap { max-width: 640px; margin: 0 auto; padding: 24px 16px 56px; }
.card {
  background: var(--card); border: 1px solid var(--line); border-radius: 20px;
  padding: 20px; margin-bottom: 16px; box-shadow: var(--shadow);
}
.card[hidden] { display: none; }

/* 3. COMPONENTS */
.brand { display: flex; align-items: center; gap: 12px; margin-bottom: 6px; }
.logo {
  width: 46px; height: 46px; border-radius: 14px; display: grid; place-items: center;
  font-size: 1.6rem; background: linear-gradient(135deg, var(--accent), var(--love));
}
h1 { font: 700 1.9rem/1.1 Georgia, "Times New Roman", serif; margin: 0; letter-spacing: -.01em; }
.tagline { color: var(--muted); margin: 0 0 18px; }
h2 { font-size: 1.05rem; margin: 0 0 4px; }
.small { font-size: .88rem; color: var(--muted); margin: 0; }

.legend { display: flex; flex-wrap: wrap; gap: 6px; margin: 0 0 14px; }
.tag {
  font-size: .8rem; padding: 3px 10px; border-radius: 999px;
  color: var(--c); background: color-mix(in srgb, var(--c) 12%, transparent);
  border: 1px solid color-mix(in srgb, var(--c) 35%, transparent);
}

label { display: block; font-weight: 600; margin-bottom: 8px; }
textarea {
  width: 100%; min-height: 112px; padding: 14px; resize: vertical;
  font: inherit; font-size: 16px;               /* 16px stops iPhone zoom-in */
  color: var(--text); background: var(--bg);
  border: 1.5px solid var(--line); border-radius: 14px;
}
textarea:focus { border-color: var(--accent); outline: none; box-shadow: 0 0 0 4px color-mix(in srgb, var(--accent) 22%, transparent); }
.meta { display: flex; justify-content: space-between; margin: 6px 2px 14px; }

.row { display: flex; gap: 10px; flex-wrap: wrap; }
button {
  font: inherit; font-weight: 600; min-height: 48px; padding: 0 20px;
  border-radius: 14px; border: 1.5px solid transparent; cursor: pointer;
  transition: transform .12s, background .2s;
}
button:active { transform: scale(.97); }
button:focus-visible { outline: 3px solid var(--accent); outline-offset: 2px; }
.primary { flex: 1 1 160px; background: var(--accent); color: var(--accent-text); }
.ghost { background: transparent; color: var(--text); border-color: var(--line); }
.link { min-height: 36px; padding: 0 4px; background: none; color: var(--muted); text-decoration: underline; font-weight: 500; }

/* result */
#result { animation: rise .35s ease both; border-top: 5px solid var(--c); }
@keyframes rise { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: none; } }
.verdict { display: flex; align-items: center; gap: 14px; margin-bottom: 16px; }
.face {
  width: 68px; height: 68px; border-radius: 50%; display: grid; place-items: center; font-size: 2.2rem;
  background: color-mix(in srgb, var(--c) 16%, transparent);
  border: 2px solid color-mix(in srgb, var(--c) 40%, transparent);
}
.verdict h2 { font: 700 1.6rem/1.15 Georgia, serif; color: var(--c); margin: 0; }

.bars { display: grid; gap: 9px; margin: 0 0 16px; }
.bar { display: grid; grid-template-columns: 92px 1fr 40px; align-items: center; gap: 10px; font-size: .9rem; }
.track { height: 10px; background: var(--line); border-radius: 999px; overflow: hidden; }
.fill { height: 100%; width: 0; border-radius: 999px; background: var(--c); transition: width .6s cubic-bezier(.2, .8, .2, 1); }
.bar b { text-align: right; font-variant-numeric: tabular-nums; }

.chips { display: flex; flex-wrap: wrap; gap: 6px; margin-top: 8px; }
.chip { font-size: .9rem; padding: 3px 11px; border-radius: 999px; color: var(--c);
  background: color-mix(in srgb, var(--c) 13%, transparent); border: 1px solid color-mix(in srgb, var(--c) 40%, transparent); }
.chip.unk { --c: var(--muted); border-style: dashed; }
.block { margin-bottom: 14px; }
.block[hidden] { display: none; }

hr { border: 0; border-top: 1px solid var(--line); margin: 16px 0; }
.picker { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; margin-top: 10px; }
.pick {
  min-height: 52px; padding: 6px; color: var(--text);
  background: color-mix(in srgb, var(--c) 10%, var(--card));
  border-color: color-mix(in srgb, var(--c) 40%, transparent);
}
.pick:hover { background: color-mix(in srgb, var(--c) 24%, var(--card)); }
.note { color: var(--accent); font-weight: 600; min-height: 1.4em; margin: 10px 0 0; }

/* history */
.hist { list-style: none; margin: 10px 0 0; padding: 0; display: grid; gap: 8px; }
.hist button {
  width: 100%; text-align: left; min-height: 48px; display: flex; gap: 10px; align-items: center;
  padding: 8px 12px; font-weight: 500; background: var(--bg); color: var(--text); border-color: var(--line);
}
.hist span.t { flex: 1; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.hist span.p { color: var(--c); font-size: .85rem; font-weight: 700; }

.footer { text-align: center; margin-top: 8px; }
.footer .row { justify-content: center; }

@media (max-width: 420px) {
  .picker { grid-template-columns: repeat(2, 1fr); }
  .bar { grid-template-columns: 80px 1fr 38px; }
  h1 { font-size: 1.6rem; }
}
@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}
</style>
</head>

<body>
<!-- =====================================================
     HTML: what is on the page
     ===================================================== -->
<main class="wrap">

  <header>
    <div class="brand">
      <div class="logo" aria-hidden="true">🔍</div>
      <h1>MoodLens</h1>
    </div>
    <p class="tagline">A small, free AI that reads the feeling behind your words, and learns from you.</p>
  </header>

  <!-- Input card -->
  <section class="card">
    <div class="legend" id="legend" aria-label="Moods the AI knows"></div>
    <label for="text">How are you feeling? Write a sentence.</label>
    <textarea id="text" maxlength="280" placeholder="Example: I just got great news and I cannot stop smiling"></textarea>
    <div class="meta">
      <span class="small">Tip: Ctrl + Enter to analyze</span>
      <span class="small" id="count">0 / 280</span>
    </div>
    <div class="row">
      <button class="primary" id="go">Analyze mood</button>
      <button class="ghost" id="sample">Try an example</button>
    </div>
  </section>

  <!-- Result card (hidden until you analyze) -->
  <section class="card" id="result" hidden aria-live="polite">
    <div class="verdict">
      <div class="face" id="face"></div>
      <div>
        <h2 id="mood"></h2>
        <p class="small" id="msg"></p>
      </div>
    </div>

    <div class="bars" id="bars"></div>

    <div class="block">
      <p class="small">Words that shaped the answer</p>
      <div class="chips" id="chips"></div>
    </div>
    <div class="block" id="unkBox" hidden>
      <p class="small">Words I do not know yet (ignored)</p>
      <div class="chips" id="unk"></div>
    </div>

    <hr>
    <h2>Did I get it wrong? Teach me.</h2>
    <p class="small">Tap the right mood. I will remember it, even after you close this page.</p>
    <div class="picker" id="picker"></div>
    <p class="note" id="note" role="status"></p>
  </section>

  <!-- History card -->
  <section class="card" id="histCard" hidden>
    <h2>Recent sentences</h2>
    <p class="small">Tap one to try it again.</p>
    <ul class="hist" id="hist"></ul>
  </section>

  <footer class="footer">
    <p class="small" id="stats"></p>
    <div class="row">
      <button class="link" id="clearHist">Clear history</button>
      <button class="link" id="reset">Forget what I learned</button>
    </div>
    <p class="small" style="margin-top:10px">Runs fully in your browser. Nothing is sent anywhere.</p>
  </footer>
</main>

<script>
/* =====================================================
   JavaScript: how the AI thinks
   ===================================================== */

// STEP 1: the moods the AI knows
const MOODS = [
  { id: "joy",      name: "Joy",      emoji: "😄", msg: "Bright, happy energy." },
  { id: "love",     name: "Love",     emoji: "🥰", msg: "Warmth and affection." },
  { id: "surprise", name: "Surprise", emoji: "😲", msg: "Something unexpected happened." },
  { id: "sadness",  name: "Sadness",  emoji: "😢", msg: "A heavy, low feeling." },
  { id: "anger",    name: "Anger",    emoji: "😠", msg: "Frustration or irritation." },
  { id: "fear",     name: "Fear",     emoji: "😨", msg: "Worry, anxiety or fright." }
];
const MIXED = { id: "mixed", name: "Mixed
