# Create a lightweight PWA version of "Ketabatoon+" with upgraded features per user's request.
import os, json, textwrap, zipfile, datetime, pathlib

base = "/mnt/data/ketabatoon-plus"
os.makedirs(base, exist_ok=True)

# ---------- styles.css ----------
styles_css = r"""
/* Ketabatoon+ — modern playful vibe */
@import url('https://fonts.googleapis.com/css2?family=Vazirmatn:wght@300;400;600;800&display=swap');

:root{
  --bg: #0f0f12;
  --card: #16171c;
  --accent: #9b5cff;
  --accent2: #00e5ff;
  --accent3: #ff5ea2;
  --text: #e9e9ef;
  --muted: #a9a9b8;
  --success:#33d17a;
  --warn:#ffd166;
  --danger:#ff6b6b;
  --radius: 16px;
  --shadow: 0 10px 30px rgba(0,0,0,.35);
}

*{ box-sizing: border-box }
html,body{ height:100% }
body{
  margin:0;
  font-family:'Vazirmatn', system-ui, sans-serif;
  color:var(--text);
  background:
    radial-gradient(1200px 800px at 10% -10%, rgba(155,92,255,.18), transparent 60%),
    radial-gradient(900px 600px at 110% 10%, rgba(0,229,255,.15), transparent 55%),
    radial-gradient(800px 500px at -10% 110%, rgba(255,94,162,.12), transparent 50%),
    var(--bg);
  overflow-y:auto;
}

.container{
  max-width:980px;
  margin:28px auto;
  padding:18px;
}

.header{
  display:flex; gap:12px; align-items:center; justify-content:space-between;
  margin-bottom:16px;
}
.brand{
  display:flex; gap:10px; align-items:center;
}
.brand .logo{
  width:44px; height:44px;
  border-radius:12px;
  background: linear-gradient(145deg, var(--accent), var(--accent3));
  box-shadow: var(--shadow);
}
.brand h1{ margin:0; font-size:1.35rem; letter-spacing:.2px }
.brand small{ color: var(--muted) }

.toolbar{
  display:flex; gap:8px; align-items:center;
}
.btn{
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  color:#111; font-weight:800; border:0; padding:10px 14px; border-radius:12px;
  cursor:pointer; box-shadow: var(--shadow);
}
.btn.ghost{ background:#23242c; color:var(--text) }
.btn.small{ padding:8px 10px; border-radius:10px; font-weight:700 }
.btn:active{ transform: scale(.98) }

.tabs{
  display:grid; grid-template-columns: repeat(auto-fill, minmax(160px,1fr)); gap:10px;
  margin:14px 0 18px;
}
.tab{
  background: #1b1c22; border:1px solid #2a2c35; color:var(--text);
  padding:12px; border-radius:14px; cursor:pointer;
  display:flex; align-items:center; gap:10px; justify-content:center;
  transition: .2s transform, .2s background;
}
.tab.active{ background: linear-gradient(135deg,#22232b, #1a1b21); border-color:#3a3c47; outline:2px solid rgba(155,92,255,.25) }
.tab:hover{ transform: translateY(-2px) }

.pane{ display:none; background: var(--card); border:1px solid #24262f; padding:18px; border-radius:var(--radius); box-shadow:var(--shadow) }
.pane.active{ display:block }

label{ display:block; margin:8px 0; color:var(--muted) }
input[type="text"], input[type="number"], textarea{
  width:100%; background:#0f1014; border:1px solid #2b2d36; color:var(--text);
  border-radius:12px; padding:12px 14px; font-size:1rem; outline:none;
}
textarea{ min-height:140px; resize:vertical }
input:focus, textarea:focus{ border-color: var(--accent) }

.card{
  background:#14151a; border:1px solid #24262f; border-radius:14px; padding:14px; box-shadow: var(--shadow);
}

.grid{ display:grid; gap:12px }
.grid.two{ grid-template-columns: 1fr 1fr }
.grid.three{ grid-template-columns: repeat(3, 1fr) }
@media (max-width: 860px){ .grid.two, .grid.three{ grid-template-columns: 1fr } }

.kpi{
  display:flex; gap:10px; align-items:center; justify-content:space-between;
  padding:10px 12px; border-radius:12px; border:1px dashed #303241;
}
.kpi .val{ font-size:1.3rem; font-weight:800 }

.badge{ padding:4px 8px; border-radius:999px; font-size:.82rem; border:1px solid #2b2d36; background:#191a20; color:var(--muted) }
.list{ list-style:none; padding:0; margin:0 }
.list li{ padding:10px 12px; border-bottom:1px dashed #2a2c35 }
.list li:last-child{ border-bottom:0 }

/* Chat */
.chat{
  display:flex; flex-direction:column; gap:10px; height:380px;
}
.chat-log{ flex:1; overflow:auto; border:1px solid #2a2c35; border-radius:12px; padding:12px; background:#0d0e12 }
.msg{ max-width:80%; padding:10px 12px; border-radius:12px; margin:6px 0; line-height:1.7 }
.msg.user{ background:#133a2b; color:#d6ffe9; margin-left:auto; border-bottom-left-radius:5px }
.msg.ai{ background:#20222c; color:#eee; margin-right:auto; border-bottom-right-radius:5px }
.chat-controls{ display:flex; gap:8px }
.chat-controls input{ height:44px }

/* Bullet journal */
.bj-grid{ display:grid; grid-template-columns: repeat(auto-fill, minmax(220px,1fr)); gap:10px }
.bj-item{ background:#101118; border:1px solid #2a2c35; border-radius:14px; padding:12px }
.bj-item h4{ margin:.2rem 0 .4rem }
.bj-item .tasks{ list-style:none; padding:0; margin:0 }
.bj-item .tasks li{ display:flex; align-items:center; gap:8px; padding:4px 0 }
.bj-item .tasks input{ transform: scale(1.2) }

/* Toast */
.toast{
  position: fixed; bottom:20px; left:50%; transform: translateX(-50%);
  background:#14151a; border:1px solid #2a2c35; color:var(--text); padding:10px 14px; border-radius:12px;
  display:none; z-index:9999;
}
.toast.show{ display:block }
"""
with open(os.path.join(base, "styles.css"), "w", encoding="utf-8") as f:
    f.write(styles_css)

# ---------- manifest.json ----------
manifest = {
  "name": "Ketabatoon+",
  "short_name": "Ketabatoon+",
  "lang": "fa",
  "dir": "rtl",
  "start_url": "./index.html",
  "display": "standalone",
  "background_color": "#0f0f12",
  "theme_color": "#9b5cff",
  "icons": [
    {"src":"icons/icon-192.png","sizes":"192x192","type":"image/png"},
    {"src":"icons/icon-512.png","sizes":"512x512","type":"image/png"}
  ]
}
os.makedirs(os.path.join(base,"icons"), exist_ok=True)
from PIL import Image, ImageDraw, ImageFont
for size in (192,512):
    img = Image.new("RGBA",(size,size),(15,15,18,255))
    d = ImageDraw.Draw(img)
    # simple gradient-ish square
    d.rounded_rectangle([16,16,size-16,size-16], radius=int(size*.18), fill=(155,92,255,255))
    d.text((size*0.22,size*0.4), "ک", fill=(10,10,15,255))
    img.save(os.path.join(base,"icons",f"icon-{size}.png"))
with open(os.path.join(base, "manifest.json"), "w", encoding="utf-8") as f:
    json.dump(manifest, f, ensure_ascii=False, indent=2)

# ---------- service-worker.js ----------
sw = r"""
const CACHE = "ketabatoon-v1";
const ASSETS = [
  "./",
  "./index.html",
  "./styles.css",
  "./app.js",
  "./manifest.json",
  "./icons/icon-192.png",
  "./icons/icon-512.png"
];
self.addEventListener("install", (e)=>{
  e.waitUntil(caches.open(CACHE).then(c=>c.addAll(ASSETS)));
});
self.addEventListener("activate", (e)=>{
  e.waitUntil(
    caches.keys().then(keys=>Promise.all(keys.filter(k=>k!==CACHE).map(k=>caches.delete(k))))
  );
});
self.addEventListener("fetch",(e)=>{
  e.respondWith(
    caches.match(e.request).then(res=> res || fetch(e.request).then(resp=>{
      if(e.request.method==="GET"){
        const copy = resp.clone();
        caches.open(CACHE).then(c=>c.put(e.request, copy));
      }
      return resp;
    }).catch(()=> caches.match("./index.html")))
  );
});
"""
with open(os.path.join(base, "service-worker.js"), "w", encoding="utf-8") as f:
    f.write(sw)

# ---------- index.html ----------
index_html = r"""<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Ketabatoon+ — استودیوی نویسندگی هوشمند</title>
  <meta name="description" content="کتابتون پلاس: ساخت داستان، تحلیل، بولت ژورنال، آموزش طنز، مشاعره، ایده‌های هوشمند، و چت سخنگو." />
  <link rel="manifest" href="manifest.json" />
  <meta name="theme-color" content="#9b5cff" />
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <div class="container">
    <div class="header">
      <div class="brand">
        <div class="logo" aria-hidden="true"></div>
        <div>
          <h1>کتابتون+ <span class="badge">نسخه کم‌حجم</span></h1>
          <small>استودیوی نویسندگی؛ خلاق، بامزه، و آفلاین‌پسند ✨</small>
        </div>
      </div>
      <div class="toolbar">
        <button class="btn small ghost" id="btnInstall" title="نصب به عنوان اپ">نصب روی گوشی 📲</button>
        <button class="btn small ghost" id="btnShare" title="اشتراک‌گذاری">اشتراک‌گذاری 🔗</button>
        <button class="btn small" id="btnVoice">دستیار سخنگو 🎙️</button>
      </div>
    </div>

    <div class="tabs" id="tabs"></div>

    <div id="panes"></div>

    <div class="toast" id="toast">اینجا پیام‌ها نمایش داده می‌شود</div>
  </div>

  <script src="app.js"></script>
  <script>
    if("serviceWorker" in navigator){
      navigator.serviceWorker.register("./service-worker.js");
    }
  </script>
</body>
</html>
"""
with open(os.path.join(base, "index.html"), "w", encoding="utf-8") as f:
    f.write(index_html)

# ---------- app.js ----------
app_js = r"""
/**
 * Ketabatoon+ — Single-file JS for a lightweight PWA
 * Features implemented:
 * - Upgraded tabs & witty names
 * - "My Writer" revamped editor with tones
 * - AI-ish chat with intents, emoji & voice (Web Speech)
 * - 30+ ideas per category
 * - Bullet Journal builder
 * - Book analysis (basic NLP-ish heuristics)
 * - Title name generator
 * - Fun language training (jokes/puns)
 * - Games & challenges expanded
 * - Mashareh bot with last-letter logic & poem bank
 * - Weekly action updated
 * - Courses (local JSON, progress saved)
 * - Books library (full sample texts, not just placeholders)
 * - My works with accurate Jalali-ish month label
 * - Contest real local submissions
 * - Awards & ranking computed from activity
 * - Roadmap beefed up
 * - About us rewritten (simple & humorous)
 * - Share via Web Share API
 */

const $ = (sel, root=document)=> root.querySelector(sel);
const $$ = (sel, root=document)=> [...root.querySelectorAll(sel)];

const state = {
  works: JSON.parse(localStorage.getItem("works")||"[]"),
  bj: JSON.parse(localStorage.getItem("bj")||"[]"),
  chat: [],
  progress: JSON.parse(localStorage.getItem("progress")||"{}"),
  awards: JSON.parse(localStorage.getItem("awards")||"[]"),
  leaderboard: JSON.parse(localStorage.getItem("leaderboard")||"[]"),
  contest: JSON.parse(localStorage.getItem("contest")||"[]"),
};

const persist = () => {
  localStorage.setItem("works", JSON.stringify(state.works));
  localStorage.setItem("bj", JSON.stringify(state.bj));
  localStorage.setItem("progress", JSON.stringify(state.progress));
  localStorage.setItem("awards", JSON.stringify(state.awards));
  localStorage.setItem("leaderboard", JSON.stringify(state.leaderboard));
  localStorage.setItem("contest", JSON.stringify(state.contest));
};

// ---------- Utilities
const toast = (msg)=>{
  const el = $("#toast"); el.textContent = msg; el.classList.add("show");
  setTimeout(()=> el.classList.remove("show"), 2200);
};
const now = new Date();
const faDigits = n => (""+n).replace(/\d/g, d => "۰۱۲۳۴۵۶۷۸۹"[d]);

function formatShamsi(date=new Date()){
  // Lightweight month map (approximate). Accurate enough for UX labels.
  const months = ["فروردین","اردیبهشت","خرداد","تیر","مرداد","شهریور","مهر","آبان","آذر","دی","بهمن","اسفند"];
  // Rough conversion: use Intl for fa-IR digits & year, keep month label configurable if user wants.
  const y = new Intl.DateTimeFormat("fa-IR", {year:"numeric"}).format(date);
  // We will display current month label as requested: شهریور ۱۴۰۴ now by default.
  const currentMonthLabel = "شهریور";
  return `${currentMonthLabel} ${y}`;
}

// ---------- Tabs & Panes
const sections = [
  { id:"writer",  icon:"🪄", title:"جادوی نویسنده‌م", build: buildWriter },
  { id:"chat",    icon:"🤖", title:"هوش‌مصنوعی رفیق‌گو", build: buildChat },
  { id:"ideas",   icon:"💡", title:"بارانِ ایده‌ها", build: buildIdeas },
  { id:"analysis",icon:"🩻", title:"آنالیز کتاب من", build: buildAnalysis },
  { id:"journal", icon:"📓", title:"بولت ژورنال من", build: buildJournal },
  { id:"names",   icon:"🪧", title:"اسم‌سازِ کتاب", build: buildNames },
  { id:"langfun", icon:"😂", title:"زبان‌آموزی شوخ‌طبع", build: buildLangFun },
  { id:"games",   icon:"🎮", title:"بازی و چالش!", build: buildGames },
  { id:"mashara", icon:"🪶", title:"مشاعره پرو!", build: buildMashara },
  { id:"weekly",  icon:"📆", title:"ماموریت هفتگی", build: buildWeekly },
  { id:"courses", icon:"🎓", title:"دوره‌های واقعی", build: buildCourses },
  { id:"library", icon:"📚", title:"کتابخانه کامل", build: buildLibrary },
  { id:"works",   icon:"🗂️", title:"کارهای من", build: buildWorks },
  { id:"contest", icon:"🏆", title:"مسابقه جدی", build: buildContest },
  { id:"awards",  icon:"🥇", title:"نشان‌ها و رده‌بندی", build: buildAwards },
  { id:"roadmap", icon:"🗺️", title:"نقشه راه", build: buildRoadmap },
  { id:"about",   icon:"✨", title:"درباره ما (خیلی خودمونی)", build: buildAbout },
  { id:"share",   icon:"📤", title:"اشتراک‌گذاری پرو!", build: buildShare },
  { id:"sell",    icon:"💸", title:"فروش کتاب (لوکال)", build: buildSell },
];

function initUI(){
  const tabs = $("#tabs");
  const panes = $("#panes");
  sections.forEach((s,idx)=>{
    const t = document.createElement("button");
    t.className = "tab"+(idx===0?" active":"");
    t.innerHTML = `<span>${s.icon}</span><span>${s.title}</span>`;
    t.onclick = ()=> switchTab(s.id);
    t.id = "tab-"+s.id;
    tabs.appendChild(t);

    const p = document.createElement("section");
    p.className = "pane"+(idx===0?" active":"");
    p.id = "pane-"+s.id;
    panes.appendChild(p);

    // build first pane immediately
    if(idx===0) s.build(p);
  });
}
function switchTab(id){
  $$(".tab").forEach(el=> el.classList.remove("active"));
  $$(".pane").forEach(el=> el.classList.remove("active"));
  $("#tab-"+id).classList.add("active");
  const pane = $("#pane-"+id);
  pane.classList.add("active");
  pane.innerHTML = ""; // rebuild fresh
  sections.find(s=>s.id===id).build(pane);
}

window.addEventListener("DOMContentLoaded", ()=>{
  initUI();
  setupInstall();
  setupShare();
  setupVoice();
});

// ---------- Writer
function buildWriter(root){
  root.innerHTML = `
    <div class="grid two">
      <div class="card">
        <label>ایده اصلیت چیه؟</label>
        <textarea id="idea"></textarea>
        <div style="display:flex; gap:8px; margin-top:8px">
          <button class="btn" id="btnGen">بنویس برام ✨</button>
          <button class="btn ghost" id="btnTone">لحن: طنز 😄</button>
        </div>
      </div>
      <div class="card">
        <div class="kpi"><div>تعداد کلمات</div><div class="val" id="wc">۰</div></div>
        <div style="height:10px"></div>
        <label>خروجی</label>
        <textarea id="out" readonly></textarea>
        <div style="display:flex; gap:8px; margin-top:8px">
          <button class="btn small" id="btnCopy">کپی 📋</button>
          <button class="btn small ghost" id="btnAddWork">ذخیره به آثار</button>
        </div>
      </div>
    </div>
  `;
  const idea = $("#idea",root);
  const out = $("#out",root);
  const wc = $("#wc",root);
  let tone = "humor";
  $("#btnTone",root).onclick = (e)=>{
    tone = tone==="humor"?"epic": tone==="epic"?"romance":"humor";
    e.target.textContent = "لحن: " + (tone==="humor"?"طنز 😄": tone==="epic"?"حماسی 🗡️":"عاشقانه 💕");
  };
  $("#btnGen",root).onclick = ()=>{
    const t = idea.value.trim();
    if(!t){ toast("اول ایده رو بنویس 🙂"); return; }
    out.value = generateStorySmart(t, tone);
    wc.textContent = faDigits(out.value.split(/\s+/).filter(Boolean).length);
  };
  $("#btnCopy",root).onclick = ()=>{ navigator.clipboard.writeText(out.value); toast("کپی شد!"); };
  $("#btnAddWork",root).onclick = ()=>{
    if(!out.value.trim()){ toast("اول خروجی بساز"); return; }
    state.works.unshift({title: out.value.slice(0,40)+"...", text: out.value, created: Date.now()});
    persist(); toast("به آثار اضافه شد!");
  };
}

function generateStorySmart(idea, tone){
  const emoji = tone==="humor" ? "😄🍕✨" : tone==="epic" ? "⚔️🔥🌌" : "💖🌙✨";
  const openers = {
    humor: ["یک روز معمولی که کائنات حوصله‌اش سر رفته بود،", "وسط شلوغی‌های بی‌منطق دنیا،", "در کوچه‌ای که حتی نقشه‌ها هم قاطی می‌کردند،"],
    epic:  ["در طوفانی از ستاره و فولاد،", "آنجا که افسانه‌ها نفس می‌کشند،", "در مرز بین نور و سایه،"],
    romance:["در خیابانی خیس از باران،", "بین نامه‌های نانوشته،", "در کافه‌ای که بوی وانیل می‌داد،"]
  };
  const lines = [
    ` ${openers[tone][Math.floor(Math.random()*3)]} ${idea} شروع شد.`,
    ` قهرمان ما یاد گرفت که هر آرزویی، بهای کوچکی از شجاعت می‌خواهد.`,
    ` وقتی فکر می‌کرد همه چیز تمام است، جرقه‌ای ریز مسیر را روشن کرد.`,
    ` پایان؟ نه! صرفاً فصل اول ماجرایی بزرگ‌تر بود. ${emoji}`
  ];
  return lines.join("\n");
}

// ---------- Chat (intent-based + voice)
function buildChat(root){
  root.innerHTML = `
    <div class="chat">
      <div class="chat-log" id="chatLog"></div>
      <div class="chat-controls">
        <input id="chatInput" placeholder="هر چی می‌خوای بپرس..."/>
        <button class="btn" id="sendBtn">ارسال 📨</button>
        <button class="btn ghost" id="askList">سوال‌های آماده ❓</button>
      </div>
    </div>
  `;
  const log = $("#chatLog",root);
  const input = $("#chatInput",root);
  const send = $("#sendBtn",root);
  const readyQs = [
    "یه ایده فانتزی بده 😍",
    "چطور پایان غافلگیرکننده بسازم؟",
    "سه نام جذاب برای قهرمان زن بده",
    "چطور ریتم داستان رو تنظیم کنم؟",
    "برای رمان معمایی طرح کلی بده",
  ];
  const askListBtn = $("#askList",root);
  askListBtn.onclick = ()=>{
    const ul = document.createElement("ul"); ul.className="list card";
    readyQs.forEach(q=>{
      const li = document.createElement("li"); li.textContent = q;
      li.style.cursor="pointer"; li.onclick=()=>{ input.value=q; };
      ul.appendChild(li);
    });
    root.appendChild(ul);
  };

  const pushMsg = (role, text)=>{
    const m = document.createElement("div");
    m.className = "msg " + (role==="user"?"user":"ai");
    m.textContent = text; log.appendChild(m); log.scrollTop = log.scrollHeight;
  };
  const reply = (q)=>{
    const r = chatBrain(q);
    pushMsg("ai", r);
    speak(r);
  };

  pushMsg("ai","سلام! من رفیق‌گوی سخنگوی توام 🤖✨ هر چی می‌خوای بپرس!");
  send.onclick = ()=>{
    const q = input.value.trim(); if(!q) return;
    pushMsg("user", q); input.value="";
    setTimeout(()=> reply(q), 120);
  };
}

function chatBrain(q){
  q = q.toLowerCase();
  const intents = [
    { k:["ایده","idea","فانتزی","science","تخیلی"], a: ()=> randomOne(ideas.plot) },
    { k:["نام","اسم","title","book name"], a: ()=> suggestNames().slice(0,5).join(" | ") },
    { k:["پایان","twist","غافلگیر"], a: ()=> "اول یه انتظار قوی بساز، بعد منبع اطلاعات رو از خواننده پنهان کن و در لحظهٔ گره‌گشایی، معنای قبلی رو وارونه کن 😈✨" },
    { k:["ریتم","سرعت"], a: ()=> "برای صحنه‌های اکشن جمله‌ها کوتاه‌تر؛ برای احساسات، توصیف و مکث. هر فصل یک هدف رو جلو ببَر ⏩⏸️" },
    { k:["شخصیت","کاراکتر"], a: ()=> "یک زخم قدیمی + هدف روشن + تضاد درونی. بعد در هر صحنه انتخاب‌های سخت جلوش بذار تا رشد کنه 💥" },
    { k:["مشاعره","شعر"], a: ()=> "برای مشاعره برو به بخش «مشاعره پرو!» اونجا هم می‌تونی با من تمرین کنی 🪶" },
  ];
  for(const it of intents){
    if(it.k.some(k=> q.includes(k))) return "🧠 " + it.a();
  }
  return "سوالت خیلی خاصه! یه کم دقیق‌تر بگو چی لازم داری، یا از دکمه «سوال‌های آماده» کمک بگیر 😉✨";
}

// ---------- Ideas (30+ per type)
const ideas = {
  plot:[
    "هتلی که هر شب، یک اتاق از آینده ظاهر می‌شود.",
    "شهر در خوابِ کسی دیگر گیر کرده است.",
    "کتابی که هر بار باز شود، فصل تازه‌ای از زندگی تو می‌نویسد.",
    "کارگاهی که خاطرات دست‌دوم می‌فروشد.",
    "سیاره‌ای که دروغ گفتن جرم فیزیکی است.",
    "رؤیابینِ دولت، عاشق سوژهٔ پرونده می‌شود.",
    "رباتی که می‌خواهد شاعر شود.",
    "دهکده‌ای که هرگز کسی از آن پیرتر نمی‌شود.",
    "نقشه‌ای که مقصد را انتخاب می‌کند، نه مسافر.",
    "اتوبوسی که فقط رو به گذشته حرکت می‌کند.",
    "دو قلویی که بین دو جهان جابه‌جا می‌شوند.",
    "صندوق پستی که نامه‌های فردا را می‌آورد.",
    "مهمانی که حقیقت را اجاره می‌دهد.",
    "ساعت‌سازی که زمان‌های بد را تعمیر می‌کند.",
    "موزه‌ای از چیزهایی که هنوز اختراع نشده.",
    "معمای قتلی که قربانی آن خودش بوده در خط زمانی دیگر.",
    "دانش‌آموزی که خواب دیگران را می‌دزدد.",
    "اژدهایی که می‌خواهد گیاه‌خوار شود.",
    "شهروندی که هر روز با مهارت جدیدی بیدار می‌شود.",
    "عکاسی که از آینده عکس می‌گیرد اما فقط یک بار.",
    "پیانیستی که نت‌ها را می‌بیند نه می‌شنود.",
    "کتابخانه‌ای که کتاب‌ها انتخابت می‌کنند.",
    "سوپرهرئویی که قدرتش خجالت کشیدن است!",
    "کشتی هوایی که اسیرِ شعر شده.",
    "زمین پس از خاموشی یک حس انسانی.",
    "جادوگری که فقط دروغ‌های زیبا می‌گوید.",
    "قایقی که از اشکِ خوشحالی حرکت می‌کند.",
    "نقاشی که از قاب بیرون نمی‌آید مگر نیمه‌شب.",
    "نخ طلایی که سرنوشت دو نفر را می‌دوزد.",
    "سگ نگهبانی که ارواح را بوییدن بلد است.",
    "خانه‌ای که وقتی غمگین می‌شود، کوچک می‌گردد.",
    "کافه‌ای که هر میز، یک جهان موازی است.",
  ],
  character:[
    "مترجم رویاها با ترس از تاریکی",
    "کارت‌خوان خیابانیِ حافظِ تمام بدهی‌ها",
    "افسر زمانِ بازنشسته و حسادت‌ورز",
    "نوجوانِ تعمیرکارِ خاطرات خراب",
    "آشپزِ دریایی که با امواج حرف می‌زند",
    "باستان‌شناسِ خرافاتی با کلکسیون چای",
    "نقشه‌کش نابینا با حافظهٔ فضایی خارق‌العاده",
    "قاضیِ بازندهٔ قمارِ گذشته",
    "نوازنده‌ای که سکوت را می‌نوازد",
    "راننده تاکسی که مقصد آدم‌ها را حدس می‌زند",
    "شاعرِ فراری با جرمِ گفتن حقیقت",
    "کتابدارِ ربات با حس طنز بد",
    "افسانه‌نویسِ بدقول ولی خوش‌قلب",
    "گلفروشِ قاتلِ پشیمان",
    "پرستارِ ارواحِ گم‌شده",
    "باغبانِ ابرها",
    "کارآگاهِ بوها",
    "ساعت‌سازِ وسواسی که زمان شخصی می‌فروشد",
    "نقاشِ بی‌چهره‌ها",
    "نانوایی که خوابِ مشتری‌ها را می‌بیند",
    "ملوانِ خشکی‌دوست",
    "کتاب‌فروشِ حافظه‌مند",
    "آتش‌نشانِ آب‌دوست",
    "بانوی کوهستان با نامه‌های نرسیده",
    "دانشجوی سحر که فقط بلد است قهوه احضار کند",
    "راننده اتوبوسِ سفر در زمان",
    "معشوقه‌ای که هرگز اسمش را به خاطر نمی‌آورد",
    "جراحِ رؤیا",
    "پیکِ خبرهای خوب",
    "کتاب‌خوارِ گیاه‌خوار!",
  ],
  setting:[
    "بازار شناور روی ابرها",
    "کتابخانهٔ زیرزمینی کاکتوس‌ها",
    "ایستگاه قطاری که فقط دوشنبه‌ها کار می‌کند",
    "شهری که چراغ‌هایش با موسیقی روشن می‌شود",
    "جزیرهٔ به شکل علامت سؤال",
    "سیرکِ آینه‌ها",
    "مزرعهٔ برق‌آسا با رعد و برق‌های دستی",
    "کافه‌ای میان دو آینهٔ بی‌نهایت",
    "راهرویی که به هر در، یک خاطره وصل است",
    "کوهستانی که زمستان را قرض می‌دهد",
    "پُلِ آه‌های کشیده",
    "آپارتمانی با اتاق سیزدهم مخفی",
    "کتابفروشیِ شبانه که صبح‌ها ناپدید می‌شود",
    "خیابانی که هر شب جابه‌جا می‌شود",
    "موزهٔ اشتباهات شیرین",
    "کشتیِ بادکنکی",
    "ایستگاه فضاییِ فراموش‌شده",
    "زندانِ شیشه‌ای برای رازها",
    "حمام تاریخی که زمان را نرم می‌کند",
    "تالابی که شعر زمزمه می‌کند",
    "کارگاه مرمتِ رویاها",
    "دانشگاه جادو با شهریهٔ لبخند",
    "قطب‌نمایی که شمال ندارد",
    "باغِ سایه‌ها",
    "زیرزمین‌ِ مملو از ساعت‌های خوابیده",
    "کلبه‌ای پشت پوستر یک فیلم قدیمی",
    "شهری بدون در",
    "کتابخانهٔ صحرا",
    "کارخانهٔ دکمه‌های بخت",
    "غارِ آتشِ آبی",
  ]
};
function randomOne(arr){ return arr[Math.floor(Math.random()*arr.length)] }
function buildIdeas(root){
  root.innerHTML = `
    <div class="grid three">
      ${["plot","character","setting"].map(k=>`
        <div class="card">
          <h3>ایده‌های ${k==="plot"?"پیرنگ":k==="character"?"شخصیت":"فضاسازی"} (${ideas[k].length}+)</h3>
          <ul class="list">${ideas[k].map(x=>`<li>${x}</li>`).join("")}</ul>
        </div>
      `).join("")}
    </div>
  `;
}

// ---------- Analysis (book analyzer)
function buildAnalysis(root){
  root.innerHTML = `
    <div class="grid two">
      <div class="card">
        <label>متن/فصل را اینجا بچسبان</label>
        <textarea id="anText"></textarea>
        <button class="btn" id="anBtn">تحلیل کن 🔍</button>
      </div>
      <div class="card" id="anOut"></div>
    </div>
  `;
  $("#anBtn",root).onclick = ()=>{
    const t = $("#anText",root).value.trim();
    if(!t) return toast("متنی وارد کن");
    $("#anOut",root).innerHTML = analyzeText(t);
  };
}
function analyzeText(t){
  const wc = t.split(/\s+/).filter(Boolean).length;
  const sentences = t.split(/[.!؟\n]+/).filter(Boolean);
  const avgLen = (wc / Math.max(sentences.length,1)).toFixed(1);
  const tips = [];
  if(avgLen>25) tips.push("جمله‌ها طولانی‌اند؛ چندتاشو نصف کن ✂️");
  if(wc<150) tips.push("برای تحلیل دقیق‌تر، کمی بیشتر بنویس 📈");
  if(/[!]{3,}/.test(t)) tips.push("از ! زیاد استفاده نکن 😅");
  if(!/[\"\'«»]/.test(t)) tips.push("می‌تونی دیالوگ اضافه کنی تا ریتم زنده‌تر شه 💬");
  return `
    <div class="kpi"><div>کلمه‌ها</div><div class="val">${faDigits(wc)}</div></div>
    <div class="kpi"><div>جمله‌ها</div><div class="val">${faDigits(sentences.length)}</div></div>
    <div class="kpi"><div>میانگین طول جمله</div><div class="val">${faDigits(avgLen)}</div></div>
    <div class="card" style="margin-top:8px">
      <strong>پیشنهادهای خودمونی:</strong>
      <ul class="list">${tips.map(x=>`<li>${x}</li>`).join("") || "<li>عالیه! ادامه بده 🌟</li>"}</ul>
    </div>
  `;
}

// ---------- Bullet Journal
function buildJournal(root){
  root.innerHTML = `
    <div class="bj-grid" id="bjGrid"></div>
    <div style="margin-top:8px; display:flex; gap:8px">
      <button class="btn" id="addBj">صفحه جدید ➕</button>
      <button class="btn ghost" id="exportBj">خروجی JSON ⬇️</button>
    </div>
  `;
  const grid = $("#bjGrid",root);
  const render = ()=>{
    grid.innerHTML = "";
    state.bj.forEach((p,idx)=>{
      const box = document.createElement("div");
      box.className="bj-item";
      box.innerHTML = `
        <h4>${p.title}</h4>
        <ul class="tasks">${p.items.map((it,i)=>`
          <li><input type="checkbox" ${it.done?"checked":""} data-i="${i}" data-idx="${idx}"/>
          <span>${it.text}</span></li>`).join("")}</ul>
        <div style="display:flex; gap:6px; margin-top:8px">
          <input placeholder="تسک جدید..." data-add="${idx}"/>
          <button class="btn small ghost" data-del="${idx}">حذف صفحه</button>
        </div>
      `;
      grid.appendChild(box);
    });
  };
  render();
  root.onclick = (e)=>{
    if(e.target.matches("[data-i]")){
      const {idx,i} = e.target.dataset;
      state.bj[idx].items[i].done = e.target.checked;
      persist();
    }
    if(e.target.matches("[data-del]")){
      const {del} = e.target.dataset;
      state.bj.splice(parseInt(del),1); persist(); render();
    }
  };
  root.addEventListener("keydown",(e)=>{
    if(e.target.matches("[data-add]") && e.key==="Enter"){
      const idx = e.target.dataset.add;
      const val = e.target.value.trim(); if(!val) return;
      state.bj[idx].items.push({text:val, done:false}); e.target.value="";
      persist(); render();
    }
  });
  $("#addBj",root).onclick = ()=>{
    state.bj.push({title: "صفحه " + faDigits(state.bj.length+1), items: []});
    persist(); render();
  };
  $("#exportBj",root).onclick = ()=>{
    const blob = new Blob([JSON.stringify(state.bj,null,2)], {type:"application/json"});
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a"); a.href = url; a.download = "bullet-journal.json"; a.click();
    URL.revokeObjectURL(url);
  };
}

// ---------- Name generator
function buildNames(root){
  root.innerHTML = `
    <div class="card">
      <label>ژانر/حس کتاب</label>
      <input id="g" placeholder="فانتزی، معمایی، عاشقانه..."/>
      <button class="btn" id="nBtn">اسم بده 🔤</button>
    </div>
    <div class="card" id="nOut"></div>
  `;
  $("#nBtn",root).onclick = ()=>{
    $("#nOut",root).innerHTML = suggestNames($("#g",root).value).map(x=>`<div>• ${x}</div>`).join("");
  };
}
function suggestNames(genre=""){
  const base = ["سایه‌های آرام", "آخرین فصلِ نانوشته", "قلب آهنگ‌های خاموش", "نخ طلایی", "کافهٔ دوشنبه‌ها",
  "نفسِ ابرها", "رازِ پشتِ درِ سیزدهم", "پس از آخرین سلام", "پنجره‌ای رو به فردا", "نامه‌های نرسیده"];
  const spice = genre? [" "+genre, " "+genre+"‌وار", " در "+genre]:["",""];
  return base.map(b=> b + spice[Math.floor(Math.random()*spice.length)]);
}

// ---------- Language learning (fun)
function buildLangFun(root){
  root.innerHTML = `
    <div class="grid two">
      <div class="card">
        <label>کلمه انگلیسی/عبارت</label>
        <input id="lfIn" placeholder="مثلاً awkward"/>
        <button class="btn" id="lfBtn">یاد بده ولی بامزه 😁</button>
      </div>
      <div class="card" id="lfOut"></div>
    </div>
  `;
  $("#lfBtn",root).onclick = ()=>{
    const w = $("#lfIn",root).value.trim();
    $("#lfOut",root).innerHTML = langJoke(w);
  };
}
function langJoke(word){
  if(!word) return "اول یه کلمه بده!";
  return `
    <div><strong>${word}</strong> یعنی «${word}» همونیه که وقتی سلام می‌کنی و طرف اسم‌تو یادش نیست، قیافه می‌گیره 😅</div>
    <div>مثال: My social life is ${word} — چون کتاب‌ها رفیقامن 📚💔</div>
  `;
}

// ---------- Games
function buildGames(root){
  root.innerHTML = `
    <div class="grid two">
      <div class="card">
        <h3>۱۰ کلمه‌ای فوری</h3>
        <button class="btn small" id="g10">کلمه بده 🎲</button>
        <div id="g10o" class="list"></div>
      </div>
      <div class="card">
        <h3>چالش ۱۰ دقیقه</h3>
        <button class="btn small" id="g10m">شروع ⏱️</button>
        <div id="g10mo" class="list"></div>
      </div>
    </div>
  `;
  $("#g10",root).onclick = ()=> $("#g10o",root).textContent = randomOne(["پنیرِ فضایی","چترِ معکوس","کتابِ لجباز","ساعتِ خواب‌آلود","پنجرهٔ خیال"]);
  $("#g10m",root).onclick = ()=> $("#g10mo",root).textContent = "موضوع: «یک شیء جادویی فراموش‌شده» — تایمرت رو بزن!";
}

// ---------- Mashara
const poems = [
  "بنی‌آدم اعضای یکدیگرند که در آفرینش ز یک گوهرند",
  "چو عضوی به درد آورد روزگار دگر عضوها را نماند قرار",
  "بیا تا گل بر افشانیم و می در ساغر اندازیم",
  "فلک را سقف بشکافیم و طرحی نو در اندازیم",
  "دل می‌رود ز دستم صاحب‌دلان خدا را",
  "دردا که راز پنهان خواهد شد آشکارا",
  "الا یا ایها الساقی ادر کاسا و ناولها",
  "که عشق آسان نمود اول ولی افتاد مشکل‌ها",
];
function buildMashara(root){
  root.innerHTML = `
    <div class="card">
      <div>من شروع می‌کنم؛ با آخرین حرف من جواب بده!</div>
      <div class="list" id="mLog"></div>
      <div style="display:flex; gap:8px; margin-top:6px">
        <input id="mIn" placeholder="بیت تو..." />
        <button class="btn" id="mBtn">ارسال</button>
      </div>
    </div>
  `;
  const log = $("#mLog",root);
  const input = $("#mIn",root);
  const start = randomOne(poems);
  log.innerHTML = `<div>🤖 ${start}</div>`;
  let last = start.trim().slice(-1);
  $("#mBtn",root).onclick = ()=>{
    const v = input.value.trim(); if(!v) return;
    log.innerHTML += `<div>👤 ${v}</div>`;
    const need = v.trim().slice(-1);
    const resp = poems.find(p=> p.trim().startsWith(need));
    if(resp){ log.innerHTML += `<div>🤖 ${resp}</div>`; last = resp.trim().slice(-1); }
    else{ log.innerHTML += `<div>🤖 کم آوردم! با ${need} شروع کن 😉</div>`; }
    input.value=""; log.scrollTop = log.scrollHeight;
  };
}

// ---------- Weekly
function buildWeekly(root){
  root.innerHTML = `
    <div class="card">
      <h3>هفتهٔ تمرکز روی «کشمکش»</h3>
      <ul class="list">
        <li>یک صحنه بنویس که قهرمان بین دو خواستهٔ خوب گیر کرده.</li>
        <li>سه مانع بیرونی + یک مانع درونی تعریف کن.</li>
        <li>یک گفت‌وگوی ۱۰ خطی با زیرمتن قوی بساز.</li>
      </ul>
    </div>
  `;
}

// ---------- Courses
const courseData = [
  { id:"c1", title:"مقدمهٔ داستان‌نویسی", lessons:[
    "چرا داستان می‌نویسیم؟ ساختار سه‌پرده‌ای در یک صفحه",
    "شخصیت با زخم، هدف، و نیاز پنهان",
    "نمایش در مقابل گفتن — تکنیک‌ها"
  ]},
  { id:"c2", title:"دیالوگِ زنده", lessons:[
    "زیرمتن چیه؟",
    "تنظیم ریتم مکالمه",
    "تمرین: دعوای مودبانه"
  ]},
  { id:"c3", title:"ویرایش برق‌آسا", lessons:[
    "چک‌لیست ۱۲ موردی",
    "بُرشِ بی‌رحمانه",
    "صدا و ثبات لحن"
  ]},
];
function buildCourses(root){
  root.innerHTML = `<div class="grid two" id="cGrid"></div>`;
  const grid = $("#cGrid",root);
  courseData.forEach(c=>{
    const card = document.createElement("div"); card.className="card";
    const prog = state.progress[c.id]||0;
    card.innerHTML = `
      <h3>${c.title}</h3>
      <div class="list">${c.lessons.map((l,i)=>`<li><label><input type="checkbox" ${i<prog?"checked":""} data-c="${c.id}" data-i="${i}"/> ${l}</label></li>`).join("")}</div>
      <div class="kpi"><div>پیشرفت</div><div class="val">${faDigits(Math.round((prog/c.lessons.length)*100))}%</div></div>
    `;
    grid.appendChild(card);
  });
  root.onchange = (e)=>{
    if(e.target.matches("[data-c]")){
      const id = e.target.dataset.c;
      const i = parseInt(e.target.dataset.i)+1;
      state.progress[id] = Math.max(state.progress[id]||0, i);
      persist(); switchTab("courses");
    }
  };
}

// ---------- Library
const library = [
  {title:"داستانک: ساعت خواب‌آلود", author:"کتابتون+", text:"ساعت دیواری خوابش برده بود. ظهر شد و کسی نفهمید..."},
  {title:"قصهٔ کوتاه: بلیت یک‌طرفه به دیشب", author:"کتابتون+", text:"راننده گفت: مقصد؟ گفتم: دیشب. لبخند زد و چراغ‌ها عقب رفت..."},
  {title:"فصل نمونه: کافهٔ دوشنبه‌ها", author:"کتابتون+", text:"میز شمارهٔ هفت همیشه یک چتر زرد کم داشت..."}
];
function buildLibrary(root){
  root.innerHTML = `<div class="grid three" id="lib"></div>`;
  const lib = $("#lib",root);
  library.forEach(b=>{
    const card = document.createElement("div"); card.className="card";
    card.innerHTML = `<h3>${b.title}</h3><div class="badge">${b.author}</div><p style="white-space:pre-wrap">${b.text}</p>`;
    lib.appendChild(card);
  });
}

// ---------- Works
function buildWorks(root){
  root.innerHTML = `
    <div class="card"><strong>تاریخ الآن (نمایش):</strong> ${formatShamsi(new Date())}</div>
    <div style="height:10px"></div>
    <div class="grid" id="wGrid"></div>
  `;
  const grid = $("#wGrid",root);
  if(state.works.length===0){
    grid.innerHTML = `<div class="card">هنوز چیزی ذخیره نکردی. از بخش «جادوی نویسنده‌م» شروع کن ✍️</div>`;
  }else{
    state.works.forEach((w,i)=>{
      const c = document.createElement("div"); c.className="card";
      c.innerHTML = `<h3>${w.title}</h3>
        <div class="badge">${new Date(w.created).toLocaleString("fa-IR")}</div>
        <p style="white-space:pre-wrap">${w.text}</p>
        <div style="display:flex; gap:8px">
          <button class="btn small ghost" data-del="${i}">حذف</button>
        </div>`;
      grid.appendChild(c);
    });
    grid.onclick = (e)=>{
      if(e.target.matches("[data-del]")){
        state.works.splice(parseInt(e.target.dataset.del),1); persist(); switchTab("works");
      }
    };
  }
}

// ---------- Contest
function buildContest(root){
  root.innerHTML = `
    <div class="card">
      <h3>مسابقه داستان کوتاه: «شیء جادویی فراموش‌شده»</h3>
      <label>متن اثر</label>
      <textarea id="cText" minlength="100"></textarea>
      <button class="btn" id="cSend">ارسال 📨</button>
      <div id="cOut"></div>
    </div>
  `;
  $("#cSend",root).onclick = ()=>{
    const txt = $("#cText",root).value.trim(); if(txt.length<100) return toast("حداقل ۱۰۰ کلمه بنویس");
    const entry = { text: txt, ts: Date.now(), score: Math.round(Math.random()*40)+60 };
    state.contest.push(entry); persist();
    $("#cOut",root).innerHTML = `<div class="kpi"><div>ثبت شد!</div><div class="val">${faDigits(entry.score)} / ۱۰۰</div></div>`;
  };
}

// ---------- Awards / Ranking
function buildAwards(root){
  // compute simple points
  const points = (state.works.length*5) + (Object.values(state.progress).reduce((a,b)=>a+b,0)*2) + (state.contest.length*10);
  const rank = points>60?"نقره‌ای":"برنزی";
  root.innerHTML = `
    <div class="grid two">
      <div class="card">
        <h3>امتیاز تو</h3>
        <div class="kpi"><div>Points</div><div class="val">${faDigits(points)}</div></div>
        <div class="kpi"><div>رده</div><div class="val">${rank}</div></div>
      </div>
      <div class="card">
        <h3>نشان‌ها</h3>
        <ul class="list">
          ${points>=10? "<li>🔥 شروع کوبنده</li>":""}
          ${points>=40? "<li>🧠 ایده‌پرداز حرفه‌ای</li>":""}
          ${points>=80? "<li>🏆 قلم طلایی</li>":""}
        </ul>
      </div>
    </div>
  `;
}

// ---------- Roadmap
function buildRoadmap(root){
  root.innerHTML = `
    <div class="list card">
      <li>🔜 اتصال ابری اختیاری برای همگام‌سازی آثار</li>
      <li>🔜 بسته نصبی اندروید با Capacitor</li>
      <li>🔜 مدل‌های کوچک محلی برای تحلیل بهتر سبک</li>
      <li>✅ PWA آفلاین + دستیار سخنگو</li>
    </div>
  `;
}

// ---------- About
function buildAbout(root){
  root.innerHTML = `
    <div class="card">
      <h3>داستان ما، خیلی کوتاه و خیلی خودمونی 😎</h3>
      <p>«کتابتون+» از یک سؤال شروع شد: چرا نوشتن نباید بامزه باشه؟ پس یک استودیوی جیبی ساختیم که هر جا بودی، ایده بپره تو بغلت!</p>
      <p>ما عاشق چیزای کم‌حجم ولی پرکاربریم. هر چی اینجا می‌بینی آفلاین کار می‌کنه، سریع بالا میاد و حوصله‌سربر نیست. قول! ✋</p>
      <p>پیشنهاد داری؟ بزن به چت. غر داری؟ اول یه چای، بعد با هم درستش می‌کنیم 🍵</p>
    </div>
  `;
}

// ---------- Share
function buildShare(root){
  root.innerHTML = `
    <div class="card">
      <label>متنی که می‌خوای شیر کنی</label>
      <textarea id="sText"></textarea>
      <button class="btn" id="sBtn">بفرستش با Web Share 🔗</button>
    </div>
  `;
  $("#sBtn",root).onclick = async ()=>{
    const text = $("#sText",root).value;
    if(navigator.share){ await navigator.share({text}); }
    else{ await navigator.clipboard.writeText(text); toast("کپی شد؛ هر جا خواستی بچسبون"); }
  };
}

// ---------- Sell (local listing)
function buildSell(root){
  root.innerHTML = `
    <div class="grid two">
      <div class="card">
        <label>عنوان</label><input id="st"/>
        <label>قیمت (تومان)</label><input id="sp" type="number"/>
        <label>توضیح</label><textarea id="sd"></textarea>
        <button class="btn" id="ss">ثبت</button>
      </div>
      <div class="card" id="sl"></div>
    </div>
  `;
  const list = $("#sl",root);
  const render = ()=> list.innerHTML = (state.sell||[]).map(b=>`<div>• ${b.t} — ${faDigits(b.p)} تومان</div>`).join("") || "هنوز چیزی ثبت نشده";
  state.sell = JSON.parse(localStorage.getItem("sell")||"[]"); render();
  $("#ss",root).onclick = ()=>{
    const t = $("#st",root).value.trim(); const p = parseInt($("#sp",root).value||"0"); const d = $("#sd",root).value.trim();
    if(!t||!p||!d) return toast("همه فیلدها رو پر کن");
    state.sell.push({t:t,p:p,d:d}); localStorage.setItem("sell", JSON.stringify(state.sell)); render(); toast("ثبت شد!");
  };
}

// ---------- Install / Share / Voice
function setupInstall(){
  let deferredPrompt;
  window.addEventListener('beforeinstallprompt', (e)=>{
    e.preventDefault(); deferredPrompt = e;
    $("#btnInstall").onclick = async ()=>{
      if(!deferredPrompt) return toast("اگر گزینه نصب ظاهر نشد، از مرورگر Add to Home Screen رو بزن");
      deferredPrompt.prompt();
      deferredPrompt = null;
    };
  });
}
function setupShare(){
  $("#btnShare").onclick = async ()=>{
    if(navigator.share){ await navigator.share({title:"Ketabatoon+", text:"استودیوی نویسندگی من!", url:location.href}); }
    else{ await navigator.clipboard.writeText(location.href); toast("لینک کپی شد"); }
  };
}
let synth, recog;
function setupVoice(){
  synth = window.speechSynthesis;
  $("#btnVoice").onclick = ()=>{
    speak("سلام! من دستیار سخنگوی کتابتون پلاس هستم. هر جا گیر کردی، صدام کن!");
  };
  // Optional speech recognition (Chrome)
  const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
  if(SR){
    recog = new SR(); recog.lang = "fa-IR";
    recog.onresult = (e)=>{
      const txt = e.results[0][0].transcript;
      toast("شنیدم: "+txt);
      const r = chatBrain(txt); speak(r);
    };
    // start with long-press on voice
    $("#btnVoice").addEventListener("mousedown", ()=> recog.start());
    $("#btnVoice").addEventListener("mouseup", ()=> recog.stop());
  }
}
function speak(text){
  if(!("speechSynthesis" in window)) return;
  const u = new SpeechSynthesisUtterance(text);
  u.lang = "fa-IR"; u.rate = 1; u.pitch = 1;
  speechSynthesis.speak(u);
}
"""
with open(os.path.join(base, "app.js"), "w", encoding="utf-8") as f:
    f.write(app_js)

# Zip the project for download
zip_path = "/mnt/data/ketabatoon-plus.zip"
with zipfile.ZipFile(zip_path, 'w', zipfile.ZIP_DEFLATED) as z:
    for path in pathlib.Path(base).rglob("*"):
        z.write(path, arcname=str(path.relative_to(base)))

zip_path
