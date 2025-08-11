<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
<title>Répertoire de hadiths</title>
<style>
  :root{
    --bg:#0b1220; --text:#e5e7eb; --muted:#9aa3b2;
    --card:#0f172a; --bd:#1f2a44; --accent:#2b5cff; --ok:#10b981; --warn:#f59e0b;
  }
  *{box-sizing:border-box}
  html,body{margin:0;background:var(--bg);color:var(--text);font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial}
  .app{display:grid;grid-template-columns:280px 1fr;min-height:100vh}
  aside{border-right:1px solid var(--bd);padding:14px;background:#0e172c}
  .brand{font-weight:900;margin-bottom:10px}
  .muted{color:var(--muted)}
  .ctrl{display:flex;gap:8px;flex-wrap:wrap;margin:10px 0}
  input,select,button{border:1px solid var(--bd);background:#0c1529;color:var(--text);padding:10px;border-radius:10px}
  button{cursor:pointer;font-weight:700}
  button.primary{background:var(--accent);border-color:var(--accent)}
  .tags{display:flex;flex-wrap:wrap;gap:6px}
  .tag{font-size:12px;background:#0c1529;border:1px solid var(--bd);padding:4px 8px;border-radius:999px}
  main{padding:16px}
  .stats{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:10px}
  .box{background:#0f172a;border:1px solid var(--bd);padding:10px 12px;border-radius:12px}
  .grid{display:grid;grid-template-columns:repeat(2,1fr);gap:10px}
  @media(max-width:900px){.app{grid-template-columns:1fr} aside{order:2} .grid{grid-template-columns:1fr}}
  .card{background:#0f172a;border:1px solid var(--bd);padding:12px;border-radius:14px}
  .meta{display:flex;gap:8px;flex-wrap:wrap;margin:6px 0}
  .hl{background:#1b2d55}
  .fav{float:right;background:#102044;border-color:#28408a}
  .fav.active{background:#134e4a;border-color:#0c7c6e}
  .empty{opacity:.6;text-align:center;padding:24px}
  .footer{margin-top:18px;color:var(--muted);font-size:12px}
  .row{display:grid;grid-template-columns:1fr 1fr;gap:8px}
</style>
</head>
<body>
<div class="app">
  <aside>
    <div class="brand">📚 Répertoire de hadiths</div>
    <div class="ctrl">
      <input id="q" placeholder="Rechercher un mot, ex: miséricorde…" style="flex:1" />
      <button id="btnClear">✕</button>
    </div>
    <div class="ctrl">
      <select id="fBook" style="flex:1">
        <option value="">— Tous recueils —</option>
      </select>
    </div>
    <div class="ctrl">
      <select id="fTopic" style="flex:1">
        <option value="">— Tous thèmes —</option>
      </select>
    </div>
    <div class="ctrl">
      <select id="fNarr" style="flex:1">
        <option value="">— Tous rapporteurs —</option>
      </select>
    </div>
    <div class="ctrl">
      <button id="btnFavs">⭐ Favoris</button>
      <button id="btnAll">Tous</button>
    </div>

    <hr style="border-color:var(--bd);opacity:.4;margin:12px 0">

    <div class="ctrl">
      <button id="btnExport">Exporter JSON</button>
      <label class="muted" style="font-size:12px">
        <input type="file" id="fileImport" accept="application/json" hidden>
        <button id="btnImport">Importer JSON</button>
      </label>
    </div>

    <div class="footer">
      Données stockées en local (navigateur).  
      Tu peux ensuite les héberger en ligne (Firebase) si tu veux.
    </div>
  </aside>

  <main>
    <div class="stats">
      <div class="box">Total : <b id="statTotal">0</b></div>
      <div class="box">Résultats : <b id="statCount">0</b></div>
      <div class="box">Favoris : <b id="statFavs">0</b></div>
      <button id="btnPrint">Imprimer / PDF</button>
    </div>

    <div id="list" class="grid"></div>
    <div id="empty" class="empty" style="display:none">Aucun résultat…</div>
  </main>
</div>

<script>
/* ========= Données d’exemple (REMPLACE par tes vraies entrées) =========
   Schéma d’un hadith :
   {
     id: "bukhari-1",
     book: "Sahih al-Bukhari",
     number: "1",
     arabic: "…",
     translation: "… (français)",
     narrator: "Umar ibn al-Khattab",
     topic: "Intention",
     refs: ["Kitab al-Wahy 1", "…"]
   }
*/
const SAMPLE = [
  {
    id:"bukhari-1",
    book:"Sahih al-Bukhari",
    number:"1",
    arabic:"نَمُوذَج …", // placeholder
    translation:"Les actes ne valent que par leurs intentions… (exemple)",
    narrator:"Umar ibn al-Khattab",
    topic:"Intention",
    refs:["Bukhari, Livre 1, Hadith 1"]
  },
  {
    id:"muslim-1",
    book:"Sahih Muslim",
    number:"1",
    arabic:"نَمُوذَج …",
    translation:"La religion est conseil… (exemple)",
    narrator:"Tamim ad-Dari",
    topic:"Conseil",
    refs:["Muslim, Livre 1, Hadith 1"]
  },
  {
    id:"tirmidhi-1",
    book:"Jami' at-Tirmidhi",
    number:"2616",
    arabic:"نَمُوذَج …",
    translation:"Le miséricordieux, le Miséricordieux leur fera miséricorde… (exemple)",
    narrator:"Abdullah ibn Amr",
    topic:"Miséricorde",
    refs:["Tirmidhi 1924"]
  }
];

// ————— Persistence locale —————
const KEY="hadith_repo_v1";
let DB = load() || { items: SAMPLE, favs: {} };

function save(){ localStorage.setItem(KEY, JSON.stringify(DB)); }
function load(){ try { return JSON.parse(localStorage.getItem(KEY)||"null"); } catch { return null; } }

// ————— UI Elements —————
const E = s => document.querySelector(s);
const listEl = E('#list');
const statTotal=E('#statTotal'), statCount=E('#statCount'), statFavs=E('#statFavs');
const q=E('#q'), fBook=E('#fBook'), fTopic=E('#fTopic'), fNarr=E('#fNarr');
const empty=E('#empty');

// ————— Init filtres —————
function fillSelect(sel, arr){
  const uniq=[...new Set(arr.filter(Boolean))].sort();
  uniq.forEach(v=>{ const o=document.createElement('option'); o.value=v; o.textContent=v; sel.appendChild(o); });
}
function refreshFilters(){
  const items=DB.items;
  fBook.length=1; fTopic.length=1; fNarr.length=1;
  fillSelect(fBook, items.map(x=>x.book));
  fillSelect(fTopic, items.map(x=>x.topic));
  fillSelect(fNarr, items.map(x=>x.narrator));
}

// ————— Rendu —————
let showingFavs=false;
function render(){
  const term=(q.value||"").trim().toLowerCase();
  const book=fBook.value, topic=fTopic.value, narr=fNarr.value;

  let items = DB.items.filter(h=>{
    if(showingFavs && !DB.favs[h.id]) return false;
    if(book && h.book!==book) return false;
    if(topic && h.topic!==topic) return false;
    if(narr && h.narrator!==narr) return false;
    if(term){
      const hay=(h.translation+" "+(h.arabic||"")+" "+h.narrator+" "+h.book+" "+(h.refs||[]).join(" ")).toLowerCase();
      return hay.includes(term);
    }
    return true;
  });

  listEl.innerHTML='';
  items.forEach(h=>{
    const card=document.createElement('div'); card.className='card';
    const favBtn=document.createElement('button'); favBtn.className='fav'; favBtn.textContent = DB.favs[h.id] ? "⭐" : "☆";
    if(DB.favs[h.id]) favBtn.classList.add('active');
    favBtn.onclick=()=>{ DB.favs[h.id]=!DB.favs[h.id]; save(); render(); };

    const meta = `
      <div class="meta">
        <span class="tag">${h.book}${h.number?(" #"+h.number):""}</span>
        <span class="tag">${h.topic||"—"}</span>
        <span class="tag">${h.narrator||"—"}</span>
      </div>`;

    const refs = (h.refs||[]).map(r=>`<span class="tag">${r}</span>`).join(' ');

    card.innerHTML = `
      <div style="display:flex;justify-content:space-between;gap:8px;align-items:start">
        <div><b>${h.id}</b></div>
      </div>
      ${meta}
      ${h.arabic?`<div dir="rtl" style="font-size:18px;line-height:1.6;margin:6px 0">${h.arabic}</div>`:''}
      <div class="hl" style="padding:10px;border-radius:10px;margin:6px 0">${h.translation||''}</div>
      <div class="tags">${refs}</div>
    `;
    card.querySelector('div').appendChild(favBtn);
    listEl.appendChild(card);
  });

  statTotal.textContent = DB.items.length;
  statCount.textContent = items.length;
  statFavs.textContent = Object.values(DB.favs).filter(Boolean).length;
  empty.style.display = items.length ? 'none' : 'block';
}

// ————— Actions —————
E('#btnClear').onclick = ()=>{ q.value=''; render(); };
[q,fBook,fTopic,fNarr].forEach(el=> el.addEventListener('input', render));
E('#btnFavs').onclick=()=>{ showingFavs=true; render(); };
E('#btnAll').onclick=()=>{ showingFavs=false; render(); };

E('#btnExport').onclick=()=>{
  const blob = new Blob([JSON.stringify(DB,null,2)],{type:'application/json'});
  const a=document.createElement('a');
  a.href=URL.createObjectURL(blob);
  a.download='hadiths.json';
  a.click();
  URL.revokeObjectURL(a.href);
};
E('#btnImport').onclick=()=> E('#fileImport').click();
E('#fileImport').onchange = async e=>{
  const f=e.target.files[0]; if(!f) return;
  try{
    const text=await f.text();
    const data=JSON.parse(text);
    if(!Array.isArray(data.items||data)) throw new Error("JSON invalide: attendu {items:[…]} ou un tableau");
    // Accepte {items:…} ou directement un tableau
    DB.items = Array.isArray(data) ? data : data.items;
    DB.favs = data.favs || {};
    save(); refreshFilters(); render();
    alert("Import réussi ✅");
  }catch(err){ alert("Erreur import: "+err.message); }
};

E('#btnPrint').onclick=()=>window.print();

// ————— Boot —————
refreshFilters();
render();
</script>

</body>
</html>
