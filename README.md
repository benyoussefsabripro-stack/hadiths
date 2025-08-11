import React, { useEffect, useMemo, useRef, useState } from "react";

// --- Hadith Library — Single-file React App (GitHub-ready) ---
// Déploiement rapide:
// 1) Vite: `npm create vite@latest hadith-app -- --template react` puis `cd hadith-app`
// 2) Installe Tailwind selon la doc officielle (postcss + tailwind.config)
// 3) Remplace `src/App.jsx` par CE fichier.
// 4) `npm i` puis `npm run dev` (local) / `npm run build` (prod) / déploiement GitHub Pages ou Vercel.
//
// Fonctionnalités:
// - Recherche full‑text (FR/AR/numéro/narrateur/collection)
// - Filtres (collection, langue) + filtre par tags
// - Hadith du jour, favoris (localStorage)
// - Mode clair/sombre
// - Import/Export JSON local
// - Option GROS VOLUME: charger automatiquement un pack JSON distant (URL brute GitHub/S3/CDN)
// - Compteur total + limite d'affichage (1000) pour rester fluide
//
// Schéma attendu d'un hadith:
// { id, collection, book?, number?, narrator?, arabic?, translation?, language?, tags?: string[] }

// ------------------------
// Jeux de données par défaut (42 hadiths de l'Arba'in an‑Nawawi)
// ------------------------
const SAMPLE_HADITHS = [
  { id: "nawawi-1", collection: "Arba'in an-Nawawi", book: "Intentions", number: 1, narrator: "ʿUmar ibn al-Khaṭṭāb", arabic: "إِنَّمَا الأَعْمَالُ بِالنِّيَّاتِ ...", translation: "Les actes ne valent que par les intentions ; à chacun selon ce qu’il a eu l’intention de faire.", language: "fr", tags: ["intentions","sincérité","ikhlās"] },
  { id: "nawawi-2", collection: "Arba'in an-Nawawi", book: "Fondements de la foi", number: 2, narrator: "ʿUmar ibn al-Khaṭṭāb", arabic: "جَاءَ جِبْرِيلُ عَلَيْهِ السَّلَامُ ...", translation: "L’ange Jibrîl questionna le Prophète ﷺ sur l’islam, la foi, l’excellence et l’Heure.", language: "fr", tags: ["islam","îmân","iḥsān","jibril"] },
  { id: "nawawi-3", collection: "Arba'in an-Nawawi", book: "Fondements", number: 3, narrator: "ʿAbd Allāh ibn ʿUmar", arabic: "بُنِيَ الإِسْلَامُ عَلَى خَمْسٍ ...", translation: "L’islam est bâti sur cinq piliers…", language: "fr", tags: ["piliers","zakât","ṣalāt","ṣawm","ḥajj"] },
  { id: "nawawi-4", collection: "Arba'in an-Nawawi", book: "Décrets", number: 4, narrator: "ʿAbd Allāh ibn Masʿūd", arabic: "إِنَّ أَحَدَكُمْ يُجْمَعُ خَلْقُهُ ...", translation: "Création in utero et écriture des quatre choses : subsistance, vie, actions, destin.", language: "fr", tags: ["destin","qadar","création"] },
  { id: "nawawi-5", collection: "Arba'in an-Nawawi", book: "Innovation et limites", number: 5, narrator: "ʿĀʾisha", arabic: "مَنْ أَحْدَثَ فِي أَمْرِنَا هَذَا ...", translation: "Toute innovation dans la religion est rejetée.", language: "fr", tags: ["innovation","bidʿa"] },
  { id: "nawawi-6", collection: "Arba'in an-Nawawi", book: "Licite/illicite", number: 6, narrator: "An-Nuʿmān ibn Bashīr", arabic: "إِنَّ الْحَلَالَ بَيِّنٌ ...", translation: "Le licite est clair et l’illicite est clair ; entre les deux, des choses douteuses.", language: "fr", tags: ["halal","haram","doute"] },
  { id: "nawawi-7", collection: "Arba'in an-Nawawi", book: "Religion", number: 7, narrator: "Tamīm ad-Dārī", arabic: "الدِّينُ النَّصِيحَةُ", translation: "La religion, c’est le conseil sincère.", language: "fr", tags: ["conseil","sincérité"] },
  { id: "nawawi-8", collection: "Arba'in an-Nawawi", book: "Sacré du sang", number: 8, narrator: "Ibn ʿUmar", arabic: "أُمِرْتُ أَنْ أُقَاتِلَ النَّاسَ ...", translation: "Piliers visibles: attestation, prière, zakât.", language: "fr", tags: ["attestation","prière","zakât"] },
  { id: "nawawi-9", collection: "Arba'in an-Nawawi", book: "Commandements", number: 9, narrator: "Abū Hurayra", arabic: "مَا نَهَيْتُكُمْ عَنْهُ ...", translation: "Évitez l’interdit, accomplissez l’ordre selon vos capacités.", language: "fr", tags: ["obéissance","capacités"] },
  { id: "nawawi-10", collection: "Arba'in an-Nawawi", book: "Nourriture licite", number: 10, narrator: "Abū Hurayra", arabic: "إِنَّ اللَّهَ طَيِّبٌ ...", translation: "Allah est bon et n’accepte que le bon ; l’invocation et la subsistance licite.", language: "fr", tags: ["licite","invocation"] },
  { id: "nawawi-11", collection: "Arba'in an-Nawawi", book: "Abolition du tort", number: 11, narrator: "Jābir", arabic: "دَعْ مَا يَرِيبُكَ ...", translation: "Délaisse le doute pour ce qui ne fait aucun doute.", language: "fr", tags: ["scrupule","doute"] },
  { id: "nawawi-12", collection: "Arba'in an-Nawawi", book: "Comportement", number: 12, narrator: "Abū Hurayra", arabic: "مِنْ حُسْنِ إِسْلَامِ الْمَرْءِ ...", translation: "Faire partie du bel islam : délaisser ce qui ne te regarde pas.", language: "fr", tags: ["éthique"] },
  { id: "nawawi-13", collection: "Arba'in an-Nawawi", book: "Foi", number: 13, narrator: "Anas", arabic: "لَا يُؤْمِنُ أَحَدُكُمْ ...", translation: "Aimer pour son frère ce que l’on aime pour soi.", language: "fr", tags: ["fraternité"] },
  { id: "nawawi-14", collection: "Arba'in an-Nawawi", book: "Justice", number: 14, narrator: "Ibn Masʿūd", arabic: "لَا يَحِلُّ دَمُ امْرِئٍ ...", translation: "Cas limités où le sang est licite.", language: "fr", tags: ["justice"] },
  { id: "nawawi-15", collection: "Arba'in an-Nawawi", book: "Langue", number: 15, narrator: "Abū Hurayra", arabic: "مَنْ كَانَ يُؤْمِنُ بِاللَّهِ ...", translation: "Dire du bien ou se taire ; honorer le voisin et l’hôte.", language: "fr", tags: ["langue","voisinage","hospitalité"] },
  { id: "nawawi-16", collection: "Arba'in an-Nawawi", book: "Caractère", number: 16, narrator: "Abū Hurayra", arabic: "لَا تَغْضَبْ", translation: "Ne te mets pas en colère.", language: "fr", tags: ["colère","maîtrise"] },
  { id: "nawawi-17", collection: "Arba'in an-Nawawi", book: "Excellence", number: 17, narrator: "Abū Yaʿlā Shaddād", arabic: "إِنَّ اللَّهَ كَتَبَ الإِحْسَانَ ...", translation: "Allah a prescrit l’excellence en toute chose.", language: "fr", tags: ["iḥsān"] },
  { id: "nawawi-18", collection: "Arba'in an-Nawawi", book: "Piété", number: 18, narrator: "Abū Dharr", arabic: "اتَّقِ اللَّهَ حَيْثُمَا كُنْتَ ...", translation: "Crains Allah, fais suivre une mauvaise action d’une bonne, et sois bon avec les gens.", language: "fr", tags: ["taqwā","repentir"] },
  { id: "nawawi-19", collection: "Arba'in an-Nawawi", book: "Proximité divine", number: 19, narrator: "Ibn ʿAbbās", arabic: "احْفَظِ اللَّهَ يَحْفَظْكَ ...", translation: "Sois attentif à Allah, Il sera attentif à toi…", language: "fr", tags: ["providence","tawakkul"] },
  { id: "nawawi-20", collection: "Arba'in an-Nawawi", book: "Pudeur", number: 20, narrator: "Abū Masʿūd", arabic: "اسْتَحْيُوا مِنَ اللَّهِ ...", translation: "Ayez vraiment de la pudeur envers Allah.", language: "fr", tags: ["pudeur","hayā’"] },
  { id: "nawawi-21", collection: "Arba'in an-Nawawi", book: "Foi", number: 21, narrator: "Sufyān ibn ʿAbd Allāh", arabic: "قُلْ آمَنْتُ بِاللَّهِ ثُمَّ اسْتَقِمْ", translation: "Dis: je crois en Allah, puis sois droit.", language: "fr", tags: ["droiture","istiqāma"] },
  { id: "nawawi-22", collection: "Arba'in an-Nawawi", book: "Adoration", number: 22, narrator: "Jābir", arabic: "افْعَلُوا مِنَ الأَعْمَالِ ...", translation: "Faites des actions selon vos capacités.", language: "fr", tags: ["modération"] },
  { id: "nawawi-23", collection: "Arba'in an-Nawawi", book: "Rites", number: 23, narrator: "Abū Mālik al-Ashʿarī", arabic: "الطُّهُورُ شَطْرُ الإِيمَانِ ...", translation: "La pureté est la moitié de la foi ; la prière est lumière.", language: "fr", tags: ["pureté","dhikr","prière"] },
  { id: "nawawi-24", collection: "Arba'in an-Nawawi", book: "Commandements", number: 24, narrator: "Abū Dharr", arabic: "يَقُولُ اللَّهُ تَعَالَى ...", translation: "Allah a interdit l’injustice à Lui-même et entre vous.", language: "fr", tags: ["justice","fraternité"] },
  { id: "nawawi-25", collection: "Arba'in an-Nawawi", book: "Charité", number: 25, narrator: "Abū Hurayra", arabic: "كُلُّ سُلَامَى ...", translation: "Chaque jour, une aumône due pour chaque articulation.", language: "fr", tags: ["charité","bienfaisance"] },
  { id: "nawawi-26", collection: "Arba'in an-Nawawi", book: "Constance", number: 26, narrator: "Abū Hurayra", arabic: "كُلُّ سُلَامَى ...", translation: "La meilleure action est la plus régulière, même petite.", language: "fr", tags: ["constance"] },
  { id: "nawawi-27", collection: "Arba'in an-Nawawi", book: "Cœur", number: 27, narrator: "Wābisah", arabic: "اسْتَفْتِ قَلْبَكَ", translation: "Demande conseil à ton cœur.", language: "fr", tags: ["coeur","scrupule"] },
  { id: "nawawi-28", collection: "Arba'in an-Nawawi", book: "Morale", number: 28, narrator: "Abū Saʿīd al-Khudrī", arabic: "مَنْ رَأَى مِنْكُمْ مُنْكَرًا ...", translation: "Changer le blâmable par main/langue/cœur.", language: "fr", tags: ["ordre du bien","interdiction du mal"] },
  { id: "nawawi-29", collection: "Arba'in an-Nawawi", book: "Adoration", number: 29, narrator: "ʿIyāḍ ibn Ḥimār", arabic: "إِنَّ اللَّهَ يَقُولُ ...", translation: "Allah dit: J’ai interdit l’injustice…", language: "fr", tags: ["justice","qadar"] },
  { id: "nawawi-30", collection: "Arba'in an-Nawawi", book: "Détachement", number: 30, narrator: "Sahl ibn Saʿd", arabic: "ازْهَدْ فِي الدُّنْيَا ...", translation: "Détache-toi, les gens t’aimeront ; et Allah t’aimera.", language: "fr", tags: ["zuhd","relations"] },
  { id: "nawawi-31", collection: "Arba'in an-Nawawi", book: "Droit", number: 31, narrator: "ʿUbāda ibn aṣ-Ṣāmit", arabic: "لَا ضَرَرَ وَلَا ضِرَارَ", translation: "Pas de nuisance ni riposte nuisible.", language: "fr", tags: ["droit","tort"] },
  { id: "nawawi-32", collection: "Arba'in an-Nawawi", book: "Justice", number: 32, narrator: "Ibn ʿAbbās", arabic: "لَوْ يُعْطَى النَّاسُ ...", translation: "La preuve incombe au demandeur.", language: "fr", tags: ["preuve","justice"] },
  { id: "nawawi-33", collection: "Arba'in an-Nawawi", book: "Fiqh", number: 33, narrator: "Abū Hurayra", arabic: "إِذَا شَكَّ أَحَدُكُمْ ...", translation: "Délaisser le doute pour la certitude en prière.", language: "fr", tags: ["certitude","prière"] },
  { id: "nawawi-34", collection: "Arba'in an-Nawawi", book: "Fraternité", number: 34, narrator: "Abū Saʿīd", arabic: "الْمُسْلِمُ أَخُو الْمُسْلِمِ ...", translation: "Tout du musulman est sacré pour le musulman.", language: "fr", tags: ["droits","fraternité"] },
  { id: "nawawi-35", collection: "Arba'in an-Nawawi", book: "Aide", number: 35, narrator: "Abū Hurayra", arabic: "مَنْ نَفَّسَ عَنْ مُؤْمِنٍ ...", translation: "Allah soulage celui qui soulage un croyant…", language: "fr", tags: ["entraide","miséricorde"] },
  { id: "nawawi-36", collection: "Arba'in an-Nawawi", book: "Solidarité", number: 36, narrator: "Anas", arabic: "انْصُرْ أَخَاكَ ...", translation: "Soutiens ton frère : empêche-le s’il est injuste.", language: "fr", tags: ["justice","entraide"] },
  { id: "nawawi-37", collection: "Arba'in an-Nawawi", book: "Destin", number: 37, narrator: "Ibn ʿAbbās", arabic: "إِنَّ اللَّهَ كَتَبَ الْحَسَنَاتِ ...", translation: "Mesure des bonnes et mauvaises actions.", language: "fr", tags: ["intention","rétribution"] },
  { id: "nawawi-38", collection: "Arba'in an-Nawawi", book: "Compagnie", number: 38, narrator: "Abū Hurayra", arabic: "إِنَّ اللَّهَ قَالَ ...", translation: "Je suis comme Mon serviteur s’attend à Moi.", language: "fr", tags: ["dhikr","espérance"] },
  { id: "nawawi-39", collection: "Arba'in an-Nawawi", book: "Éducation", number: 39, narrator: "Ibn ʿAbbās", arabic: "يَا غُلَامُ ...", translation: "Ô jeune ! Préserve Allah, Il te préservera.", language: "fr", tags: ["éducation","tawakkul"] },
  { id: "nawawi-40", collection: "Arba'in an-Nawawi", book: "Ascèse", number: 40, narrator: "Ibn ʿUmar", arabic: "كُنْ فِي الدُّنْيَا كَأَنَّكَ غَرِيبٌ ...", translation: "Sois en ce monde comme un étranger.", language: "fr", tags: ["ascèse","priorités"] },
  { id: "nawawi-41", collection: "Arba'in an-Nawawi", book: "Obéissance", number: 41, narrator: "Abū Muḥammad", arabic: "اتَّبِعُوا وَلَا تَبْتَدِعُوا", translation: "Suivez, n’innovez pas.", language: "fr", tags: ["sunna","innovation"] },
  { id: "nawawi-42", collection: "Arba'in an-Nawawi", book: "Miséricorde", number: 42, narrator: "Anas", arabic: "يَقُولُ اللَّهُ ...", translation: "Ô fils d’Adam, si tes péchés atteignaient les cieux puis tu demandais pardon, Je te pardonnerais.", language: "fr", tags: ["pardon","rahma"] },
];

// ------------------------
// Utils
// ------------------------
const useLocalStorage = (key, initialValue) => {
  const [value, setValue] = useState(() => {
    try { const item = localStorage.getItem(key); return item ? JSON.parse(item) : initialValue; } catch { return initialValue; }
  });
  useEffect(() => { try { localStorage.setItem(key, JSON.stringify(value)); } catch {} }, [key, value]);
  return [value, setValue];
};

const classNames = (...xs) => xs.filter(Boolean).join(" ");

function fuzzyIncludes(text, query) {
  if (!query) return true; const t = (text || "").toLowerCase(); const q = query.toLowerCase().trim(); return t.includes(q);
}

function todayIndex(n) {
  if (!n) return 0; const d = new Date(); const seed = d.getFullYear()*10000 + (d.getMonth()+1)*100 + d.getDate(); return seed % n;
}

// ------------------------
// App
// ------------------------
export default function App() {
  // 🔌 Option: URL brute d'un gros JSON (stockée en localStorage pour persister entre sessions)
  const DATA_URL = typeof window !== 'undefined' ? (localStorage.getItem('hadith-data-url') || '') : '';

  const [userData, setUserData] = useLocalStorage("hadith-data", SAMPLE_HADITHS);
  const [loading, setLoading] = useState(false);
  const [loadError, setLoadError] = useState("");

  const [query, setQuery] = useState("");
  const [collectionFilter, setCollectionFilter] = useState("all");
  const [langFilter, setLangFilter] = useState("all");
  const [tag, setTag] = useState("");
  const [favorites, setFavorites] = useLocalStorage("hadith-favs", []);
  const [theme, setTheme] = useLocalStorage("hadith-theme", "light");

  const fileRef = useRef(null);

  // Chargement auto du pack distant
  useEffect(() => {
    let cancelled = false;
    async function loadRemote() {
      if (!DATA_URL) return;
      setLoading(true); setLoadError("");
      try {
        const res = await fetch(DATA_URL, { cache: 'no-store' });
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const json = await res.json();
        if (!Array.isArray(json)) throw new Error('Le JSON distant doit être un tableau d\'objets');
        if (!cancelled) setUserData(json);
      } catch (e) {
        if (!cancelled) setLoadError(e.message || 'Erreur de chargement');
      } finally {
        if (!cancelled) setLoading(false);
      }
    }
    loadRemote();
    return () => { cancelled = true; };
  }, [DATA_URL, setUserData]);

  const collections = useMemo(() => Array.from(new Set(userData.map(h => h.collection))).sort(), [userData]);
  const languages = useMemo(() => Array.from(new Set(userData.map(h => h.language || ""))).filter(Boolean), [userData]);

  const filtered = useMemo(() => userData.filter(h => {
    const matchQ = fuzzyIncludes(h.translation, query) || fuzzyIncludes(h.arabic, query) || fuzzyIncludes(h.narrator, query) || fuzzyIncludes(h.collection, query) || fuzzyIncludes(h.book, query) || fuzzyIncludes(String(h.number || ""), query);
    const matchC = collectionFilter === "all" || h.collection === collectionFilter;
    const matchL = langFilter === "all" || (h.language || "") === langFilter;
    const matchT = !tag || (h.tags || []).some(t => fuzzyIncludes(t, tag));
    return matchQ && matchC && matchL && matchT;
  }), [userData, query, collectionFilter, langFilter, tag]);

  const dailyHadith = useMemo(() => userData[todayIndex(userData.length)] || null, [userData]);

  function toggleFav(id) { setFavorites(prev => prev.includes(id) ? prev.filter(x => x !== id) : [...prev, id]); }
  const isFav = (id) => favorites.includes(id);

  function handleUploadJSON(e) {
    const file = e.target.files?.[0]; if (!file) return; const reader = new FileReader();
    reader.onload = (ev) => { try {
      const json = JSON.parse(ev.target.result);
      if (Array.isArray(json)) {
        const normalized = json.map((h,i) => ({ id: h.id || `hadith-${Date.now()}-${i}`, collection: h.collection || "", book: h.book || "", number: h.number ?? "", narrator: h.narrator || "", arabic: h.arabic || "", translation: h.translation || "", language: h.language || "", tags: Array.isArray(h.tags) ? h.tags : [] }));
        setUserData(normalized); alert("Données importées avec succès ✅");
      } else { alert("Le fichier JSON doit être un tableau d'objets."); }
    } catch(err) { console.error(err); alert("JSON invalide."); }
    if (fileRef.current) fileRef.current.value = ""; };
    reader.readAsText(file);
  }

  function exportJSON() {
    const blob = new Blob([JSON.stringify(userData, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob); const a = document.createElement("a"); a.href = url; a.download = "hadiths.json"; a.click(); URL.revokeObjectURL(url);
  }

  useEffect(() => {
    if (typeof document !== 'undefined') {
      if (theme === 'dark') document.documentElement.classList.add('dark');
      else document.documentElement.classList.remove('dark');
    }
  }, [theme]);

  return (
    <div className={classNames("min-h-screen", theme === "dark" ? "bg-neutral-950 text-neutral-100" : "bg-neutral-50 text-neutral-900")}> 
      {/* Header */}
      <header className="sticky top-0 z-30 backdrop-blur bg-white/70 dark:bg-neutral-900/70 border-b border-neutral-200 dark:border-neutral-800">
        <div className="max-w-6xl mx-auto px-4 py-3 flex items-center gap-3">
          <div className="flex items-center gap-2">
            <span className="inline-flex h-9 w-9 items-center justify-center rounded-2xl bg-emerald-600 text-white font-bold shadow-sm">﷽</span>
            <h1 className="text-lg sm:text-xl font-semibold tracking-tight">Bibliothèque de Hadiths</h1>
          </div>
          <div className="ml-auto flex items-center gap-2">
            <button onClick={() => setTheme(theme === "dark" ? "light" : "dark")} className="px-3 py-1.5 rounded-xl border border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800">
              {theme === "dark" ? "☀️ Clair" : "🌙 Sombre"}
            </button>
            <input ref={fileRef} type="file" accept="application/json" onChange={handleUploadJSON} className="hidden" />
            <button onClick={() => fileRef.current?.click()} className="px-3 py-1.5 rounded-xl border border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800">Importer JSON</button>
            <button onClick={exportJSON} className="px-3 py-1.5 rounded-xl border border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800">Exporter JSON</button>
          </div>
        </div>
        <div className="max-w-6xl mx-auto px-4 pb-3 grid grid-cols-1 sm:grid-cols-[1fr_auto] gap-2">
          <input defaultValue={DATA_URL} onBlur={(e)=>{ localStorage.setItem('hadith-data-url', e.target.value.trim()); }} placeholder="Coller l'URL brute d'un gros JSON (GitHub raw / CDN) puis recharger" className="w-full px-3 py-2 rounded-xl border border-neutral-300 dark:border-neutral-700 bg-white dark:bg-neutral-900" />
          <button onClick={()=>window.location.reload()} className="px-3 py-2 rounded-xl border border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800">Charger URL</button>
        </div>
      </header>

      {/* Daily Hadith */}
      {dailyHadith && (
        <section className="max-w-6xl mx-auto px-4 pt-6">
          <div className="rounded-2xl border border-neutral-200 dark:border-neutral-800 p-5 bg-white dark:bg-neutral-900 shadow-sm">
            <div className="text-xs uppercase tracking-wider text-neutral-500 dark:text-neutral-400 mb-2">Hadith du jour</div>
            <h2 className="text-xl font-semibold mb-2">{dailyHadith.collection}{dailyHadith.number ? ` · n°${dailyHadith.number}` : ""}</h2>
            {dailyHadith.book && <div className="text-sm text-neutral-600 dark:text-neutral-400 mb-1">{dailyHadith.book}</div>}
            {dailyHadith.narrator && <div className="text-sm text-neutral-600 dark:text-neutral-400 mb-4">Narrateur : {dailyHadith.narrator}</div>}
            {dailyHadith.arabic && (<p className="text-lg leading-8 font-[600] mb-3 text-right" dir="rtl">{dailyHadith.arabic}</p>)}
            {dailyHadith.translation && (<p className="text-base leading-7">{dailyHadith.translation}</p>)}
            <div className="mt-4 flex flex-wrap gap-2">
              {(dailyHadith.tags || []).map(t => (<span key={t} className="text-xs px-2 py-1 rounded-full bg-emerald-100 text-emerald-700 dark:bg-emerald-900/40 dark:text-emerald-200">#{t}</span>))}
            </div>
            <div className="mt-4">
              <button onClick={() => toggleFav(dailyHadith.id)} className={classNames("px-3 py-1.5 rounded-xl border", isFav(dailyHadith.id) ? "bg-amber-500 text-white border-amber-600" : "border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800")}>{isFav(dailyHadith.id) ? "★ Favori" : "☆ Ajouter aux favoris"}</button>
            </div>
          </div>
        </section>
      )}

      {/* Controls */}
      <section className="max-w-6xl mx-auto px-4 py-6">
        {loading && (<div className="mb-3 text-sm text-emerald-700 dark:text-emerald-300">Chargement d'un gros pack distant…</div>)}
        {loadError && (<div className="mb-3 text-sm text-red-700 dark:text-red-300">Erreur: {loadError}</div>)}
        <div className="grid grid-cols-1 md:grid-cols-4 gap-3">
          <input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="Rechercher (texte, narrateur, numéro...)" className="md:col-span-2 w-full px-3 py-2 rounded-xl border border-neutral-300 dark:border-neutral-700 bg-white dark:bg-neutral-900" />
          <select value={collectionFilter} onChange={(e) => setCollectionFilter(e.target.value)} className="w-full px-3 py-2 rounded-xl border border-neutral-300 dark:border-neutral-700 bg-white dark:bg-neutral-900">
            <option value="all">Toutes les collections</option>
            {collections.map(c => (<option key={c} value={c}>{c}</option>))}
          </select>
          <select value={langFilter} onChange={(e) => setLangFilter(e.target.value)} className="w-full px-3 py-2 rounded-xl border border-neutral-300 dark:border-neutral-700 bg-white dark:bg-neutral-900">
            <option value="all">Toutes les langues</option>
            {languages.map(l => (<option key={l} value={l}>{l}</option>))}
          </select>
        </div>
        <div className="mt-3 flex items-center gap-2">
          <input value={tag} onChange={(e) => setTag(e.target.value)} placeholder="Filtrer par tag (ex: intentions, foi, halal)" className="w-full px-3 py-2 rounded-xl border border-neutral-300 dark:border-neutral-700 bg-white dark:bg-neutral-900" />
          <button onClick={() => { const v = prompt("URL brute du JSON (ex: https://raw.githubusercontent.com/toncompte/tonrepo/main/hadiths.json)", localStorage.getItem('hadith-data-url') || ''); if (v !== null) { localStorage.setItem('hadith-data-url', v.trim()); window.location.reload(); } }} className="px-3 py-2 rounded-xl border border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800">Charger URL distante</button>
          <button onClick={() => { localStorage.removeItem('hadith-data-url'); window.location.reload(); }} className="px-3 py-2 rounded-xl border border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800">Désactiver URL</button>
        </div>
      </section>

      {/* Results */}
      <section className="max-w-6xl mx-auto px-4 pb-16">
        <div className="flex items-center justify-between mb-3">
          <h3 className="text-sm text-neutral-500">{filtered.length} résultat(s) · {userData.length} hadith(s) chargés</h3>
          <button onClick={() => setUserData(SAMPLE_HADITHS)} className="text-sm underline underline-offset-4 text-neutral-600 dark:text-neutral-300">Réinitialiser (42 par défaut)</button>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          {/* NOTE: affichage limité à 1000 items pour garder l'UI fluide. Utilisez la recherche/filtre. */}
          {filtered.slice(0, 1000).map(h => (
            <article key={h.id} className="rounded-2xl border border-neutral-200 dark:border-neutral-800 p-5 bg-white dark:bg-neutral-900 shadow-sm">
              <header className="flex items-center gap-2 mb-2">
                <div className="text-xs px-2 py-1 rounded-full bg-neutral-100 dark:bg-neutral-800 border border-neutral-200 dark:border-neutral-700">{h.collection}</div>
                {h.number !== undefined && (<div className="text-xs px-2 py-1 rounded-full bg-neutral-100 dark:bg-neutral-800 border border-neutral-200 dark:border-neutral-700">n° {h.number}</div>)}
                {h.book && (<div className="ml-auto text-xs text-neutral-500">{h.book}</div>)}
              </header>
              {h.narrator && (<div className="text-sm text-neutral-600 dark:text-neutral-400 mb-3">Narrateur : {h.narrator}</div>)}
              {h.arabic && (<p className="text-lg leading-8 font-[600] mb-3 text-right" dir="rtl">{h.arabic}</p>)}
              {h.translation && (<p className="text-base leading-7">{h.translation}</p>)}
              <footer className="mt-4 flex items-center justify-between">
                <div className="flex flex-wrap gap-2">
                  {(h.tags || []).map(t => (<span key={t} className="text-xs px-2 py-1 rounded-full bg-emerald-100 text-emerald-700 dark:bg-emerald-900/40 dark:text-emerald-200">#{t}</span>))}
                </div>
                <button onClick={() => toggleFav(h.id)} className={classNames("px-3 py-1.5 rounded-xl border", isFav(h.id) ? "bg-amber-500 text-white border-amber-600" : "border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800")}>{isFav(h.id) ? "★" : "☆"}</button>
              </footer>
            </article>
          ))}
        </div>

        {favorites.length > 0 && (
          <div className="mt-10">
            <h4 className="text-lg font-semibold mb-3">Vos favoris</h4>
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
              {userData.filter(h => favorites.includes(h.id)).map(h => (
                <article key={h.id} className="rounded-2xl border border-neutral-200 dark:border-neutral-800 p-5 bg-white dark:bg-neutral-900">
                  <div className="text-sm text-neutral-600 dark:text-neutral-400 mb-1">{h.collection} {h.number ? `· n°${h.number}` : ""}</div>
                  <div className="font-medium mb-2">{h.narrator}</div>
                  {h.translation && <p className="text-base leading-7">{h.translation}</p>}
                  <div className="mt-3 flex gap-2">
                    <button onClick={() => toggleFav(h.id)} className="px-3 py-1.5 rounded-xl border border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800">Retirer</button>
                  </div>
                </article>
              ))}
            </div>
          </div>
        )}
      </section>

      {/* Footer */}
      <footer className="border-t border-neutral-200 dark:border-neutral-800 py-8">
        <div className="max-w-6xl mx-auto px-4 text-sm text-neutral-500 dark:text-neutral-400">
          <p className="mb-2">⚠️ Remarque juridique : le texte des hadiths est dans le domaine public, mais certaines traductions sont protégées par le droit d'auteur. Utilisez des traductions libres ou les vôtres.</p>
          <p>Fait avec ❤️ pour la recherche et l'étude. Optimisé pour la clarté, la recherche et les favoris.</p>
        </div>
      </footer>
    </div>
  );
}
