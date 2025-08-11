<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
<title>Répertoire de hadiths</title>

<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&family=Amiri:wght@400;700&display=swap" rel="stylesheet">

<style>
:root{
  --bg:#0b1220; --card:#0f172a; --muted:#9aa3b2; --text:#e5e7eb; --bd:#1f2a44;
  --acc:#4f8cff; --ok:#17b26a; --warn:#f59e0b; --chip:#112447; --shadow:0 14px 40px rgba(0,0,0,.35);
}
:root.light{
  --bg:#f5f7fb; --card:#ffffff; --text:#0f172a; --muted:#5b6474; --bd:#e5eaf3; --chip:#eef3ff; --shadow:0 8px 24px rgba(0,0,0,.08);
}
*{box-sizing:border-box}
html,body{margin:0;height:100%}
body{background:var(--bg);color:var(--text);font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,Arial}
a{color:inherit}
button,input,select{font-family:inherit}
.container{max-width:1100px;margin:0 auto;padding:18px}

/* Header */
.header{position:sticky;top:0;z-index:30;background:linear-gradient(180deg,rgba(11,18,32,.85),rgba(11,18,32,.65));backdrop-filter:saturate(1.2) blur(8px);border-bottom:1px solid var(--bd)}
:root.light .header{background:linear-gradient(180deg,rgba(245,247,251,.85),rgba(245,247,251,.70))}
.brand{display:flex;align-items:center;gap:12px;font-weight:800}
.logo{width:36px;height:36px;border-radius:10px;background:linear-gradient(135deg,#6aa3ff,#2b5cff);box-shadow:inset 0 0 0 2px #1e3a8a66}
.header-row{display:flex;align-items:center;justify-content:space-between;gap:12px;padding:10px 18px}
.actions{display:flex;gap:8px;align-items:center}

/* Controls */
.controls{display:grid;grid-template-columns:1fr;gap:10px;padding:12px 18px 18px}
@media(min-width:720px){.controls{grid-template-columns:1.2fr .8fr .8fr .8fr auto auto}}
.inp,.sel,.btn{border:1px solid var(--bd);background:var(--card);color:var(--text);padding:12px 12px;border-radius:12px}
.inp{width:100%}
.sel{width:100%}
.btn{cursor:pointer;font-weight:800}
.btn.primary{background:var(--acc);border-color:var(--acc);color:#fff}
.btn.ghost{background:transparent}
.btn.flat{background:var(--chip);border-color:transparent}
.badge{font-size:12px;padding:5px 8px;border-radius:999px;background:var(--chip);border:1px solid #25407c45}

/* Stats */
.stats{display:flex;gap:10px;flex-wrap:wrap;padding:0 18px 10px}
.stat{background:var(--card);border:1px solid var(--bd);padding:10px 12px;border-radius:12px;box-shadow:var(--shadow);font-weight:700}
.stat small{display:block;color:var(--muted);font-weight:600}

/* Grid */
.grid{display:grid;grid-template-columns:1fr;gap:12px;padding:0 18px 24px}
@media(min-width:760px){.grid{grid-template-columns:1fr 1fr}}
.card{background:var(--card);border:1px solid var(--bd);border-radius:16px;padding:14px;box-shadow:var(--shadow)}
.card h3{margin:0 0 6px;font-size:16px}
.meta{display:flex;gap:6px;flex-wrap:wrap;margin:6px 0}
.tag{font-size:12px;background:var(--chip);border:1px solid var(--bd);padding:4px 8px;border-radius:999px}
.ar{font-family:Amiri,serif;direction:rtl;font-size:20px;line-height:1.7;margin:8px 0}
.tr{background:rgba(68,97,166,.14);border:1px dashed #2b5cff33;padding:10px;border-radius:12px}
.fav{float:right}
.empty{opacity:.7;text-align:center;padding:20px}

/* Pagination */
.pager{display:flex;gap:8px;justify-content:center;padding:0 18px 26px}
.pagebtn{padding:10px 12px;border-radius:10px;border:1px solid var(--bd);background:var(--card);cursor:pointer}
.pagebtn.active{background:var(--acc);border-color:var(--acc);color:#fff}

/* Modal */
.modal-back{position:fixed;inset:0;background:rgba(2,6,23,.55);display:none;align-items:center;justify-content:center;padding:16px;z-index:50}
:root.light .modal-back{background:rgba(0,0,0,.25)}
.modal{width:100%;max-width:760px;background:var(--card);border:1px solid var(--bd);border-radius:16px;box-shadow:var(--shadow);padding:16px}
.modal header{display:flex;justify-content:space-between;align-items:center;gap:12px;margin-bottom:6px}
.close{background:transparent;border:1px solid var(--bd);border-radius:10px;padding:8px;cursor:pointer}

/* Print */
@media print{
  .header,.controls,.pager,.actions,.fav{display:none}
  body{background:#fff;color:#111}
  .card{break-inside:avoid;border:1px solid #ddd}
}

/* Tiny helpers */
.row{display:flex;gap:8px;flex-wrap:wrap}
</style>
</head>
<body>

<!-- Header -->
<div class="header">
  <div class="header-row container">
    <div class="brand"><div class="logo"></div><div>Répertoire de hadiths</div></div>
    <div class="actions">
      <button class="btn flat" id="btnFavs">⭐ Favoris</button>
      <button class="btn" id="btnAll">Tous</button>
      <button class="btn" id="btnTheme">🌙/☀️</button>
      <button class="btn" id="btnPrint" title="Imprimer / PDF">🖨️</button>
    </div>
  </div>

  <!-- Controls -->
  <div class="controls container">
    <input id="q" class="inp" placeholder="Rechercher (français, arabe, livre, rapporteur…)" />
    <select id="fBook" class="sel"><option value="">— Tous recueils —</option></select>
    <select id="fTopic" class="sel"><option value="">— Tous thèmes —</option></select>
    <select id="fNarr" class="sel"><option value="">— Tous rapporteurs —</option></select>
    <button id="btnExport" class="btn">Exporter</button>
    <label class="btn ghost" style="display:flex;align-items:center;gap:8px;cursor:pointer">
      <input type="file" id="fileImport" accept="application/json" hidden> Importer
    </label>
  </div>

  <!-- Stats -->
  <div class="stats container">
    <div class="stat"><small>Total</small><span id="sTotal">0</span></div>
    <div class="stat"><small>Résultats</small><span id="sCount">0</span></div>
    <div class="stat"><small>Favoris</small><span id="sFavs">0</span></div>
  </div>
</div>

<!-- List -->
<div class="container">
  <div id="list" class="grid"></div>
  <div id="empty" class="empty" style="display:none">Aucun résultat…</div>
  <div id="pager" class="pager"></div>
</div>

<!-- Modal -->
<div class="modal-back" id="mb">
  <div class="modal">
    <header>
      <h3 id="mTitle" style="margin:0"></h3>
      <div class="row">
        <button id="mFav" class="btn flat">☆</button>
        <button id="mClose" class="close">✕</button>
      </div>
    </header>
    <div class="meta" id="mMeta"></div>
    <div class="ar" id="mAr"></div>
    <div class="tr" id="mTr"></div>
    <div class="row" id="mRefs" style="margin-top:8px"></div>
  </div>
</div>

<script>
/* ========== Données démo (remplace-les ou importe un JSON) ========== */
const SAMPLE = [
  { id:"bukhari-1", book:"Sahih al-Bukhari", number:"1", arabic:"إِنَّمَا الأَعْمَالُ بِالنِّيَّاتِ …", translation:"Les actes ne valent que par leurs intentions…", narrator:"Umar ibn al-Khattab", topic:"Intention", refs:["Bukhari 1"] },
  { id:"muslim-1",  book:"Sahih Muslim",     number:"1", arabic:"الدِّينُ النَّصِيحَةُ …",            translation:"La religion est le conseil…",                narrator:"Tamim ad-Dari",        topic:"Conseil",   refs:["Muslim 55"] },
  { id:"tirmidhi-1",book:"Jami' at-Tirmidhi", number:"1924", arabic:"الرَّاحِمُونَ يَرْحَمُهُمُ الرَّحْمَٰنُ …", translation:"Les miséricordieux, le Miséricordieux leur fera miséricorde…", narrator:"Abdullah ibn Amr", topic:"Miséricorde", refs:["Tirmidhi 1924"] },
  { id:"nawawi-3",  book:"Arba'in an-Nawawi", number:"3", arabic:"بُنِيَ الإِسْلَامُ عَلَى خَمْسٍ …", translation:"L’islam est bâti sur cinq…", narrator:"Ibn Umar", topic:"Piliers", refs:["Nawawi #3"] }
];

/* ========== Persistence locale ========== */
const KEY="hadith_repo_ui_v2";
let DB = load() || { items:SAMPLE, favs:{}, theme:"dark" };

function save(){ localStorage.setItem(KEY, JSON.stringify(DB)); }
function load(){ try{ return JSON.parse(localStorage.getItem(KEY)||"null"); }catch{ return null; } }

/* ========== Éléments ========== */
const $ = s => document.querySelector(s);
const listEl = $('#list'), emptyEl = $('#empty'), pagerEl = $('#pager');
const sTotal=$('#sTotal'), sCount=$('#sCount'), sFavs=$('#sFavs');
const q=$('#q'), fBook=$('#fBook'), fTopic=$('#fTopic'), fNarr=$('#fNarr');
const btnFavs=$('#btnFavs'), btnAll=$('#btnAll'), btnPrint=$('#btnPrint'), btnTheme=$('#btnTheme');

/* ========== Thème ========== */
applyTheme(DB.theme||'dark');
btnTheme.onclick=()=>{ DB.theme = (DB.theme==='light'?'dark':'light'); applyTheme(DB.theme); save(); };
function applyTheme(mode){ document.documentElement.classList.toggle('light', mode==='light'); }

/* ========== Filtres dynamiques ========== */
function fill(sel, arr){ sel.length=1; [...new Set(arr.filter(Boolean))].sort().forEach(v=>{ const o=document.createElement('option');o.value=v;o.textContent=v;sel.appendChild(o); }); }
function refreshFilters(){
  const items=DB.items;
  fill(fBook, items.map(x=>x.book));
  fill(fTopic, items.map(x=>x.topic));
  fill(fNarr, items.map(x=>x.narrator));
}

/* ========== Recherche + Pagination ========== */
let onlyFavs=false, page=1, perPage=8, currentList=[];
function filterItems(){
  const term=(q.value||"").toLowerCase().trim();
  const book=fBook.value, topic=fTopic.value, narr=fNarr.value;
  let out = DB.items.filter(h=>{
    if(onlyFavs && !DB.favs[h.id]) return false;
    if(book && h.book!==book) return false;
    if(topic && h.topic!==topic) return false;
    if(narr && h.narrator!==narr) return false;
    if(term){
      const hay=(h.translation+" "+(h.arabic||"")+" "+h.book+" "+(h.narrator||"")+" "+(h.refs||[]).join(" ")).toLowerCase();
      return hay.includes(term);
    }
    return true;
  });
  return out;
}
function render(){
  currentList = filterItems();
  sTotal.textContent = DB.items.length;
  sCount.textContent = currentList.length;
  sFavs.textContent = Object.values(DB.favs).filter(Boolean).length;

  // Pagination
  const pages = Math.max(1, Math.ceil(currentList.length / perPage));
  page = Math.min(page, pages);
  const slice = currentList.slice((page-1)*perPage, page*perPage);

  // Cartes
  listEl.innerHTML='';
  slice.forEach(h=>{
    const card=document.createElement('div');card.className='card';
    const fav=document.createElement('button');fav.className='btn flat fav'; fav.textContent = DB.favs[h.id] ? '⭐' : '☆';
    fav.onclick=()=>{ DB.favs[h.id]=!DB.favs[h.id]; save(); render(); };

    const title=document.createElement('h3');
    title.innerHTML = `<span>${h.book}${h.number?(" #"+h.number):""}</span>`;
    title.appendChild(fav);

    const meta=document.createElement('div'); meta.className='meta';
    meta.innerHTML = `
      <span class="tag">${h.topic||'—'}</span>
      <span class="tag">${h.narrator||'—'}</span>
      <span class="tag">${h.id}</span>
    `;

    const ar = h.arabic ? `<div class="ar">${h.arabic}</div>` : '';
    const tr = `<div class="tr">${h.translation||''}</div>`;
    const refs = (h.refs||[]).map(r=>`<span class="badge">${r}</span>`).join(' ');

    card.innerHTML = '';
    card.appendChild(title);
    card.appendChild(meta);
    if(h.arabic){ const d=document.createElement('div'); d.className='ar'; d.innerHTML=h.arabic; card.appendChild(d); }
    const t=document.createElement('div'); t.className='tr'; t.innerHTML=h.translation||''; card.appendChild(t);
    const rr=document.createElement('div'); rr.className='row'; rr.innerHTML = refs; card.appendChild(rr);

    card.style.cursor='pointer';
    card.onclick = (e)=>{ if(e.target===fav) return; openModal(h); };

    listEl.appendChild(card);
  });

  emptyEl.style.display = slice.length ? 'none' : 'block';

  // Pager
  pagerEl.innerHTML='';
  if(pages>1){
    const mk = (p,txt,active=false)=>{ const b=document.createElement('button'); b.className='pagebtn'+(active?' active':''); b.textContent=txt; b.onclick=()=>{page=p; render(); window.scrollTo({top:0,behavior:"smooth"});} ; return b; };
    if(page>1) pagerEl.appendChild(mk(page-1,'←'));
    for(let i=1;i<=pages;i++){ if(i===1||i===pages||Math.abs(i-page)<=1) pagerEl.appendChild(mk(i,String(i),i===page)); else if([...pagerEl.children].at(-1)?.textContent!=='…'){ const s=document.createElement('span'); s.style.padding='10px'; s.textContent='…'; pagerEl.appendChild(s); } }
    if(page<pages) pagerEl.appendChild(mk(page+1,'→'));
  }
}

/* ========== Modal détails ========== */
const mb=$('#mb'), mTitle=$('#mTitle'), mMeta=$('#mMeta'), mAr=$('#mAr'), mTr=$('#mTr'), mRefs=$('#mRefs'), mFav=$('#mFav'), mClose=$('#mClose');
let modalHadith=null;
function openModal(h){
  modalHadith=h;
  mTitle.textContent = `${h.book}${h.number?(" #"+h.number):""}`;
  mMeta.innerHTML = `<span class="tag">${h.topic||'—'}</span> <span class="tag">${h.narrator||'—'}</span> <span class="tag">${h.id}</span>`;
  mAr.textContent = h.arabic||'';
  mAr.style.display = h.arabic?'block':'none';
  mTr.textContent = h.translation||'';
  mRefs.innerHTML = (h.refs||[]).map(r=>`<span class="badge">${r}</span>`).join(' ');
  mFav.textContent = DB.favs[h.id] ? '⭐' : '☆';
  mb.style.display='flex';
}
mClose.onclick=()=> mb.style.display='none';
mb.addEventListener('click',e=>{ if(e.target===mb) mb.style.display='none';});
mFav.onclick=()=>{ if(!modalHadith) return; const id=modalHadith.id; DB.favs[id]=!DB.favs[id]; save(); mFav.textContent = DB.favs[id] ? '⭐' : '☆'; render(); };

/* ========== IO (export/import/print) ========== */
$('#btnExport').onclick=()=>{
  const blob=new Blob([JSON.stringify(DB,null,2)],{type:'application/json'});
  const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='hadiths.json'; a.click(); URL.revokeObjectURL(a.href);
};
$('#fileImport').onchange=async e=>{
  const f=e.target.files[0]; if(!f) return;
  try{
    const t=await f.text(); const data=JSON.parse(t);
    DB.items = Array.isArray(data) ? data : (data.items || []);
    DB.favs = data.favs || {};
    save(); refreshFilters(); page=1; render();
    alert("Import réussi ✅");
  }catch(err){ alert("Erreur import: "+(err?.message||err)); }
};
btnPrint.onclick=()=>window.print();

/* ========== Réactivité ========== */
[q,fBook,fTopic,fNarr].forEach(el=> el.addEventListener('input', ()=>{ page=1; render(); }));
btnFavs.onclick=()=>{ onlyFavs=true; page=1; render(); };
btnAll.onclick=()=>{ onlyFavs=false; page=1; render(); };

/* ========== Boot ========== */
refreshFilters();
render();
</script>

</body>
</html>
