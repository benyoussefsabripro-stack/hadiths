# Projet: hadith-site

Ce dépôt est un **site de hadiths moderne** en **Next.js 14 (App Router) + TypeScript + Tailwind + Prisma (SQLite)**, prêt à pousser sur GitHub et déployer sur Vercel. Il inclut :
- API REST /api/hadiths (recherche/filtrage/pagination) et /api/random
- UI moderne (dark mode, recherche instantanée, filtres, favoris)
- Base SQLite locale + script de seed (3 hadiths d'exemple). Tu pourras importer un dataset complet plus tard.

---

## Arborescence

```
hadith-site/
├─ app/
│  ├─ api/
│  │  ├─ hadiths/route.ts
│  │  └─ random/route.ts
│  ├─ globals.css
│  └─ page.tsx
├─ components/
│  ├─ HadithCard.tsx
│  └─ ThemeToggle.tsx
├─ lib/
│  └─ db.ts
├─ prisma/
│  ├─ schema.prisma
│  └─ seed.ts
├─ public/
│  └─ favicon.ico
├─ scripts/
│  └─ import-json.ts
├─ data/
│  └─ hadiths.sample.json
├─ .env.example
├─ .gitignore
├─ next.config.mjs
├─ package.json
├─ postcss.config.mjs
├─ README.md
├─ tailwind.config.ts
└─ tsconfig.json
```

---

## Fichiers

> Copie-colle tel quel dans ton repo GitHub (même arborescence).

---

// FILE: package.json
{
  "name": "hadith-site",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "postinstall": "prisma generate",
    "db:migrate": "prisma migrate dev --name init",
    "db:reset": "prisma migrate reset --force",
    "db:seed": "node --loader ts-node/esm prisma/seed.ts",
    "import:json": "node --loader ts-node/esm scripts/import-json.ts"
  },
  "dependencies": {
    "@prisma/client": "5.17.0",
    "next": "14.2.5",
    "react": "18.3.1",
    "react-dom": "18.3.1"
  },
  "devDependencies": {
    "autoprefixer": "10.4.20",
    "postcss": "8.4.41",
    "prisma": "5.17.0",
    "tailwindcss": "3.4.10",
    "ts-node": "10.9.2",
    "typescript": "5.5.4"
  }
}

---

// FILE: next.config.mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
};
export default nextConfig;

---

// FILE: tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": false,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "types": ["node"]
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", "prisma/**/*.ts", "scripts/**/*.ts"],
  "exclude": ["node_modules"]
}

---

// FILE: tailwind.config.ts
import type { Config } from 'tailwindcss'

export default {
  darkMode: 'class',
  content: [
    './app/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}'
  ],
  theme: {
    extend: {}
  },
  plugins: []
} satisfies Config

---

// FILE: postcss.config.mjs
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};

---

// FILE: .gitignore
node_modules
.next
.env
*.sqlite

---

// FILE: .env.example
DATABASE_URL="file:./hadith.sqlite"

---

// FILE: prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model Hadith {
  id          Int      @id @default(autoincrement())
  collection  String
  book        String
  number      String
  arabic      String?
  english     String?
  french      String?
  grade       String?
  topic       String?
  narrators   String?   // JSON string (array)
  createdAt   DateTime @default(now())
}

---

// FILE: prisma/seed.ts
import { PrismaClient } from '@prisma/client'
import seed from '../data/hadiths.sample.json' assert { type: 'json' }

const prisma = new PrismaClient()

async function main() {
  for (const h of seed) {
    await prisma.hadith.create({
      data: {
        collection: h.refs?.collection ?? h.book,
        book: h.book,
        number: String(h.number),
        arabic: h.arabic ?? null,
        english: h.english ?? null,
        french: h.french ?? null,
        grade: h.grade ?? null,
        topic: h.topic ?? null,
        narrators: h.narrators ? JSON.stringify(h.narrators) : null,
      }
    })
  }
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })

---

// FILE: lib/db.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = global as unknown as { prisma: PrismaClient | undefined }
export const prisma = globalForPrisma.prisma ?? new PrismaClient()
if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma

---

// FILE: app/globals.css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  color-scheme: light dark;
}

body { @apply bg-white text-neutral-900 dark:bg-neutral-950 dark:text-neutral-100; }

.container { @apply max-w-6xl mx-auto px-4; }

.card { @apply rounded-2xl border border-neutral-200/70 dark:border-neutral-800 bg-white/70 dark:bg-neutral-900/60 backdrop-blur shadow-sm; }

.input { @apply w-full rounded-xl border border-neutral-300 dark:border-neutral-700 bg-transparent px-3 py-2 focus:outline-none focus:ring-2 focus:ring-neutral-400/40 dark:focus:ring-neutral-600/40; }

.btn { @apply inline-flex items-center gap-2 rounded-xl px-3 py-2 border border-neutral-300 dark:border-neutral-700 hover:bg-neutral-100 dark:hover:bg-neutral-800 transition; }

.badge { @apply inline-flex items-center rounded-lg px-2 py-1 text-xs border border-neutral-300 dark:border-neutral-700; }

---

// FILE: components/ThemeToggle.tsx
'use client'
import { useEffect, useState } from 'react'

export default function ThemeToggle() {
  const [dark, setDark] = useState(true)
  useEffect(() => {
    document.documentElement.classList.toggle('dark', dark)
  }, [dark])
  return (
    <button className="btn" onClick={() => setDark(d => !d)}>
      {dark ? '☀️ Thème clair' : '🌙 Thème sombre'}
    </button>
  )
}

---

// FILE: components/HadithCard.tsx
import React from 'react'

export type Hadith = {
  id: number
  collection: string
  book: string
  number: string
  arabic?: string | null
  english?: string | null
  french?: string | null
  grade?: string | null
  topic?: string | null
  narrators?: string | null // JSON array
}

export default function HadithCard({ h, lang = 'french' as 'french'|'english'|'arabic' }) {
  const text = (h as any)[lang] || h.english || h.arabic || ''
  const narrators = h.narrators ? JSON.parse(h.narrators) as string[] : []
  return (
    <article className="card p-4 flex flex-col gap-2">
      <header className="flex items-center justify-between gap-3">
        <div className="text-sm opacity-80">{h.collection} · {h.book} #{h.number}</div>
        {h.grade && <span className="badge">{h.grade}</span>}
      </header>
      <div className={lang === 'arabic' ? 'text-right text-xl leading-relaxed' : 'text-base leading-relaxed'}>
        {text}
      </div>
      <footer className="text-xs opacity-70">
        {h.topic && <span className="mr-2">{h.topic}</span>}
        {narrators.length > 0 && <span>Narrateur(s) : {narrators.join(', ')}</span>}
      </footer>
    </article>
  )}

---

// FILE: app/page.tsx
'use client'
import { useEffect, useMemo, useState } from 'react'
import ThemeToggle from '@/components/ThemeToggle'
import HadithCard, { Hadith } from '@/components/HadithCard'

export default function Page() {
  const [q, setQ] = useState('')
  const [lang, setLang] = useState<'french'|'english'|'arabic'>('french')
  const [collection, setCollection] = useState('All')
  const [grade, setGrade] = useState('All')
  const [data, setData] = useState<Hadith[]>([])
  const [loading, setLoading] = useState(false)
  const [daily, setDaily] = useState<Hadith | null>(null)
  const [bookmarks, setBookmarks] = useState<number[]>(() => {
    try { return JSON.parse(localStorage.getItem('hd_bookmarks') || '[]') } catch { return [] }
  })

  useEffect(() => {
    localStorage.setItem('hd_bookmarks', JSON.stringify(bookmarks))
  }, [bookmarks])

  const fetchData = async () => {
    setLoading(true)
    const params = new URLSearchParams()
    if (q) params.set('query', q)
    if (collection !== 'All') params.set('collection', collection)
    if (grade !== 'All') params.set('grade', grade)
    params.set('lang', lang)
    const res = await fetch('/api/hadiths?'+params.toString())
    const json = await res.json()
    setData(json.items)
    setLoading(false)
  }

  const fetchRandom = async () => {
    const res = await fetch('/api/random?lang='+lang)
    const j = await res.json()
    setDaily(j.item)
  }

  useEffect(() => { fetchData() }, [q, collection, grade, lang])
  useEffect(() => { fetchRandom() }, [lang])

  return (
    <main className="container py-8 space-y-6">
      <header className="flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
        <div>
          <h1 className="text-2xl font-semibold">Hadith — Rappels</h1>
          <p className="text-sm opacity-70">Recherche rapide, filtres intelligents, design épuré.</p>
        </div>
        <div className="flex gap-2">
          <select className="input w-36" value={lang} onChange={e=>setLang(e.target.value as any)}>
            <option value="french">Français</option>
            <option value="english">English</option>
            <option value="arabic">العربية</option>
          </select>
          <ThemeToggle />
        </div>
      </header>

      {daily && (
        <section className="card p-5">
          <div className="flex items-start justify-between">
            <div>
              <div className="text-sm opacity-70">Hadith du jour</div>
              <div className="text-xs opacity-60">{daily.collection} · {daily.book} #{daily.number} · {daily.grade || ''}</div>
            </div>
            <button className="btn" onClick={fetchRandom}>Aléatoire</button>
          </div>
          <div className={lang==='arabic'? 'mt-3 text-xl text-right' : 'mt-3 text-base'}>{(daily as any)[lang] || daily.english}</div>
          <div className="mt-3 flex gap-2">
            <button className="btn" onClick={()=>{
              navigator.clipboard.writeText(`${daily.collection} ${daily.book} ${daily.number} — ${(daily as any)[lang] || daily.english}`)
            }}>Copier</button>
            <button className="btn" onClick={()=>{
              setBookmarks(bm => bm.includes(daily.id) ? bm.filter(x=>x!==daily.id) : [...bm, daily.id])
            }}>{bookmarks.includes(daily.id) ? 'Enregistré' : 'Enregistrer'}</button>
          </div>
        </section>
      )}

      <section className="card p-4">
        <div className="flex flex-col md:flex-row gap-2">
          <input className="input" placeholder="Rechercher un hadith, un thème, un narrateur..." value={q} onChange={e=>setQ(e.target.value)} />
          <select className="input w-48" value={collection} onChange={e=>setCollection(e.target.value)}>
            <option value="All">Tous les recueils</option>
            <option value="Sahih al-Bukhari">Sahih al-Bukhari</option>
            <option value="Sahih Muslim">Sahih Muslim</option>
            <option value="Jami' at-Tirmidhi">Jami' at-Tirmidhi</option>
          </select>
          <select className="input w-40" value={grade} onChange={e=>setGrade(e.target.value)}>
            <option>All</option>
            <option>Sahih</option>
            <option>Hasan</option>
            <option>Da'if</option>
          </select>
        </div>
      </section>

      <section className="grid grid-cols-1 md:grid-cols-2 gap-4">
        {loading && <div className="card p-4">Chargement…</div>}
        {!loading && data.map(h => (
          <div key={h.id}>
            <HadithCard h={h} lang={lang} />
            <div className="mt-2 flex justify-end">
              <button className="btn" onClick={()=>setBookmarks(bm => bm.includes(h.id) ? bm.filter(x=>x!==h.id) : [...bm, h.id])}>
                {bookmarks.includes(h.id) ? 'Retirer des favoris' : 'Ajouter aux favoris'}
              </button>
            </div>
          </div>
        ))}
      </section>

      <footer className="text-xs opacity-60 text-center py-8">
        <p>Sources : textes arabes du domaine public / datasets open. Affiche toujours la source et le degré d'authenticité.
        </p>
      </footer>
    </main>
  )
}

---

// FILE: app/api/hadiths/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { prisma } from '@/lib/db'

function norm(s?: string | null) {
  return (s || '').toLowerCase()
}

export async function GET(req: NextRequest) {
  const { searchParams } = new URL(req.url)
  const query = searchParams.get('query') || ''
  const collection = searchParams.get('collection') || 'All'
  const grade = searchParams.get('grade') || 'All'
  const lang = (searchParams.get('lang') || 'french') as 'arabic'|'english'|'french'

  const all = await prisma.hadith.findMany({ orderBy: { id: 'asc' } })
  const items = all.filter(h => {
    if (collection !== 'All' && h.collection !== collection) return false
    if (grade !== 'All' && (h.grade || '') !== grade) return false
    if (!query) return true
    const q = norm(query)
    const txt = [h.arabic, h.english, h.french, h.topic, h.collection, h.book, h.number, h.grade, h.narrators]
      .filter(Boolean).map(x => norm(String(x)))
    return txt.some(t => t.includes(q))
  }).slice(0, 50)

  return NextResponse.json({ items, lang })
}

---

// FILE: app/api/random/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { prisma } from '@/lib/db'

export async function GET(req: NextRequest) {
  const count = await prisma.hadith.count()
  const skip = Math.floor(Math.random() * Math.max(1, count))
  const item = await prisma.hadith.findFirst({ skip })
  return NextResponse.json({ item })
}

---

// FILE: data/hadiths.sample.json
[
  {
    "id": "bukhari-1",
    "book": "Bukhari",
    "number": "1",
    "topic": "Intention",
    "grade": "Sahih",
    "arabic": "إِنَّمَا الأَعْمَالُ بِالنِّيَّاتِ وَإِنَّمَا لِكُلِّ امْرِئٍ مَا نَوَى",
    "english": "Actions are only by intentions, and every person will have only what they intended.",
    "french": "Les actes ne valent que par les intentions et chacun n'aura que ce qu'il a eu l'intention de faire.",
    "narrators": ["Umar ibn al-Khattab"],
    "refs": { "collection": "Sahih al-Bukhari", "book": 1, "hadith": 1 }
  },
  {
    "id": "muslim-9",
    "book": "Muslim",
    "number": "9",
    "topic": "Islam/Iman/Ihsan",
    "grade": "Sahih",
    "arabic": "الإِحْسَانُ أَنْ تَعْبُدَ اللَّهَ كَأَنَّكَ تَرَاهُ ...",
    "english": "Excellence (Ihsan) is to worship Allah as though you see Him...",
    "french": "L'excellence (Ihsân) consiste à adorer Allah comme si tu Le voyais...",
    "narrators": ["Abu Hurayra"],
    "refs": { "collection": "Sahih Muslim", "book": 1, "hadith": 9 }
  },
  {
    "id": "tirmidhi-2616",
    "book": "Tirmidhi",
    "number": "2616",
    "topic": "Bon caractère",
    "grade": "Hasan",
    "arabic": "خَيْرُكُمْ خَيْرُكُمْ لِأَهْلِهِ وَأَنَا خَيْرُكُمْ لِأَهْلِي",
    "english": "The best of you are those who are best to their families, and I am the best of you to my family.",
    "french": "Les meilleurs d'entre vous sont ceux qui sont les meilleurs envers leur famille, et je suis le meilleur d'entre vous envers ma famille.",
    "narrators": ["Aisha (ra)"],
    "refs": { "collection": "Jami' at-Tirmidhi", "book": 46, "hadith": 2616 }
  }
]

---

// FILE: scripts/import-json.ts
/**
 * Script d'import (optionnel) pour charger un JSON de hadiths (format similaire au sample)
 * Usage: pnpm import:json data/mon-fichier.json
 */
import { readFileSync } from 'node:fs'
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function run() {
  const path = process.argv[2] || 'data/hadiths.sample.json'
  const content = readFileSync(path, 'utf-8')
  const arr = JSON.parse(content)
  for (const h of arr) {
    await prisma.hadith.create({
      data: {
        collection: h.refs?.collection ?? h.book,
        book: h.book,
        number: String(h.number),
        arabic: h.arabic ?? null,
        english: h.english ?? null,
        french: h.french ?? null,
        grade: h.grade ?? null,
        topic: h.topic ?? null,
        narrators: h.narrators ? JSON.stringify(h.narrators) : null,
      }
    })
  }
  console.log(`Importé: ${arr.length} hadiths`)
}

run().then(()=>prisma.$disconnect())

---

// FILE: README.md
# Hadith — Site moderne (Next.js)

## Démarrage local
1. Clone: `git clone <ce-repo> && cd hadith-site`
2. Node 18+ conseillé. Installe: `npm i` (ou `pnpm i`)
3. Copie `.env.example` → `.env`
4. Initialise la base: `npm run db:migrate`
5. Seed d'exemple: `npm run db:seed`
6. Lance: `npm run dev` puis ouvre http://localhost:3000

## Import d'un dataset
- Place ton JSON similaire à `data/hadiths.sample.json` puis:
  ```bash
  npm run import:json -- data/ton-fichier.json
  ```

> **Important** : respecte les licences/sources des traductions. Les textes arabes classiques sont généralement dans le domaine public; les traductions modernes peuvent être sous droits. Affiche la source et le degré d'authenticité.

## Déploiement Vercel
1. Push sur GitHub
2. Crée un projet Vercel → import du repo GitHub
3. Variables: `DATABASE_URL` (laisse par défaut pour SQLite embarqué) ou utilise Postgres managé (+ `prisma migrate deploy`)

## Roadmap
- Index plein-texte (Meilisearch/Typesense) pour recherches + rapides et tolérantes aux fautes
- Fiche Hadith avec URL stable `/h/[id]`
- API publique (lecture seule)
- Ingestion Sunnah.com API (EN) avec cache local

---

// FILE: app/api/health/route.ts
import { NextResponse } from 'next/server'
export function GET(){ return NextResponse.json({ ok: true }) }
