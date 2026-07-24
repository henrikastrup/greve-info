# Ugen på Greve Info

Statisk site til GitHub Pages. Ugentlige sammendrag udgivet automatisk af n8n hver torsdag morgen.
Hver udgave består af to dele:

1. **Redaktionel del** — AI-skrevet sammendrag af de mest omtalte opslag i Facebook-gruppen *Greve Info*.
2. **Politidel** — ordret uddrag af Midt- og Vestsjællands Politis døgnrapporter, filtreret til Greve,
   Hundige, Karlslunde og Mosede. Denne del er ren tekstudtrækning uden AI.

## Filer

```
index.html              hele sitet (HTML + CSS + JS i én fil)
data/index.json         arkivets indholdsfortegnelse — n8n opdaterer denne
data/uger/ÅÅÅÅ-Wuu.json én fil pr. uge — n8n opretter en ny hver torsdag
```

## Opsætning

1. Opret et repo, fx `greve-info`, og læg alle filerne i roden.
2. **Settings → Pages → Source: Deploy from a branch → `main` / `(root)`**.
3. Sitet ligger derefter på `https://<brugernavn>.github.io/greve-info/`.
4. Lav et fine-grained personal access token med **Contents: Read and write** på repoet,
   og gem det som en GitHub-credential i n8n.
5. I n8n: ret `EJER` og `REPO` øverst i code-noden **Byg commits**.

## Datamodel

`data/index.json`:

```json
{
  "opdateret": "2026-07-23T07:15:00+02:00",
  "udgaver": [
    { "id": "2026-W30", "aar": 2026, "uge": 30, "maaned": 7,
      "dato_fra": "2026-07-16", "dato_til": "2026-07-22",
      "titel": "…", "antal_opslag": 41, "fil": "data/uger/2026-W30.json" }
  ]
}
```

`data/uger/2026-W30.json` er det samme plus:

| Felt | Bruges til |
|---|---|
| `kilde` | `{navn, url}` — vises som kildelink i toppen. Falder tilbage til konstanten `KILDE` i `index.html` |
| `top` | `{forfatter, likes, kommentarer, url}` — tegner "Ugens puls" |
| `aktivitet` | 7 tal, opslag mandag til søndag. Tegner bølgen. Mangler feltet, vises en dekorativ bølge uden tal |
| `politi` | Døgnrapport-uddragene, se nedenfor |
| `html` | Selve artiklen fra AI-agenten |

`politi`-objektet:

```json
{
  "kilde": "Midt- og Vestsjællands Politi",
  "kilde_url": "https://politi.dk/doegnrapporter?district=Midt-og-Vestsjaellands-Politi",
  "filter": ["greve","hundige","karlslunde","mosede"],
  "antal_haendelser": 6, "antal_anmeldelser": 2,
  "dage": [
    { "dato": "20. juli 2026", "iso": "2026-07-20", "url": "https://politi.dk/…",
      "haendelser": [ {"titel":"…","sted":"Lærkehegnet, Greve","tid":"17.25","tekst":"Kl 17.25 …"} ],
      "anmeldelser": [ {"kategori":"Indbrud i fritids-/sommerhus","sted":"Brittavej, Greve"} ] }
  ]
}
```

## Navne og kilder

* Forfattere i den redaktionelle del vises **kun med fornavn**. Foreninger og butikker
  (navne der matcher `kors|forening|klub|butik|center|…`) beholder deres fulde navn.
* Kilden til begge dele står i heroen og i footeren med direkte link til henholdsvis
  Facebook-gruppen og politi.dk.
* Politiuddragene gengives **ordret**. Der er ingen AI involveret i den del, netop fordi
  omskrivning af kriminalstof kan ændre betydningen.

## Sikkerhed

Artiklens HTML kommer indirekte fra Facebook-indhold. `index.html` renser den før visning:
`<script>`, `<iframe>`, `style`-attributter og alle `on*`-handlere fjernes, og links tvinges
til `target="_blank" rel="noopener nofollow"`. Sitet er sat til `noindex, nofollow`.

## Hvis politi.dk ændrer opbygning

Noden **Find ugens rapporter** kaster en fejl med teksten
*"Kunne ikke finde ng-init-blokken"*, hvis oversigtssiden laves om. Listen af rapporter
ligger i dag som HTML-escaped JSON i attributten `ng-init="init({…})"`.
Selve rapporterne parses ud fra `<div class="rich-text">` med `<h3>Titel - Sted, Kommune</h3>`
efterfulgt af `<p>`.
