# Orbis v9

**Interaktivna karta sanjskega prostora** — vizualizacija lokacij iz sanjskega dnevnika kot mehurčkov (nodov) na animiranem canvasu.

Del [Sentria]('') ekosistema.

---

## Hiter začetek

```
orbis_v9.0.9.html   ← odpri v brskalniku, naloži .sqlite bazo sanj
```

Nobene namestitve, nobene odvisnosti, nobenega strežnika. Vse deluje lokalno, v brskalniku.

Ob prvem odprtju je karta prazna, dokler ne izbereš `.sqlite` datoteke prek "Naloži Bazo" panela zgoraj levo. Ime datoteke se v statusu prikaže v trenutno izbranem jeziku (SL/EN).

---

## Kaj je Orbis

Orbis je **single-file HTML aplikacija**, ki bere obstoječo SQLite bazo sanjskega dnevnika in ti omogoča:

- Kreirati **lokacije** (node/mehurček) iz sanjskega prostora in jih ročno postaviti na interaktivno karto
- Analizirati **frekvenco pojavljanja** vsake lokacije v sanjah (barva mehurčka odraža pogostost)
- Povezati lokacije z **warpi** — enosmernimi ali dvosmernimi prehodi med prostori (Ctrl+klik na dva noda)
- Gledati **Timeline** — kronološki pregled, katera lokacija se je kdaj pojavljala, s filtrom (Vse / Samo Orbis) in omejitvijo na Top-N najpogostejših lokacij
- Uporabiti **Insights** panel — najpogostejše, neaktivne in izolirane lokacije, splošna statistika
- Simulirati **pozabljanje** — postopen padec stabilnosti neaktivnih lokacij skozi čas (nastavljiva hitrost, DOM node lahko izvzameš)
- Preklopiti **minimap** za orientacijo na večjih kartah
- Delati v **slovenščini ali angleščini** — cel UI, vključno s changelogom, sledi izbranemu jeziku

---

## Panel sistem

Vsi paneli (Master View, Help, Changelog, Insights, iskanje lokacij, Timeline) se odpirajo na sklad — **Esc zapre samo zadnje odprto okno**, ne vseh naenkrat. Timeline je prav tako del tega sklada, torej Esc med gledanjem Timeline-a zapre nazaj na karto.

Changelog ni več dostopen prek klika na verzijo v glavi — najdeš ga kot diskreten link znotraj Help panela (`H`), poleg naslova.

---

## Tehnični stack

| Komponenta | Tehnologija |
|---|---|
| UI | Vanilla JS, HTML5 Canvas 2D |
| Baza | sql.js 1.6.2 (SQLite v WASM) |
| Pisave | Inter, JetBrains Mono (Google Fonts) |
| Distribucija | Single HTML file |
| Odvisnosti | Nobenih (sql.js se naloži s CDN) |

CDN: `https://cdnjs.cloudflare.com/ajax/libs/sql.js/1.6.2/sql-wasm.js`

---

## Struktura projekta

```
orbis_v9.0.9.html      ← celotna aplikacija
CHANGELOG.md           ← zgodovina sprememb (isto besedilo kot v Help → Changelog)
ORBIS_GUIDE.md         ← tehnični vodič za razvijalce (shema, barve, i18n)
vercel.json            ← konfiguracija za deploy na Vercel
lang/
  sl.js                ← slovenski prevodi
  en.js                ← angleški prevodi
```

Jezikovni datoteki se naložita pred glavno skripto:
```html
<script src="lang/sl.js"></script>
<script src="lang/en.js"></script>
```

Vsaka nova verzija poveča `orbis_vX.Y.Z.html` — ime datoteke v tem repu vedno odraža trenutno verzijo aplikacije (glej `CHANGELOG.md` za podroben seznam sprememb po verzijah).

---

## Baza podatkov

Orbis bere **zunanjo bazo sanjskega dnevnika** (read-only) in vzdržuje **lastne tabele** v isti SQLite datoteki.

### Zunanji tabeli (sanjski dnevnik)

```sql
Dreams
  DreamID     INTEGER PK
  Date_       NVARCHAR          -- format: YYYY-MM-DD ali DD.MM.YYYY
  Location    INTEGER FK        -- → Location.LocationId
  DreamTitle  VARCHAR

SleepCycle
  SleepCycleID  INTEGER PK
  DreamID       INTEGER FK      -- → Dreams.DreamID
  Contents      VARCHAR         -- besedilo sanje ← tukaj Orbis išče
```

### Orbis tabele (avtomatsko kreirane)

```sql
Atlas_Nodes
  Id           INTEGER PK AUTOINCREMENT
  Name         TEXT NOT NULL
  Stability    REAL DEFAULT 10      -- 1.0–10.0, vizualna intenziteta
  X            REAL DEFAULT 400     -- fiksna pozicija na karti (svetovni prostor)
  Y            REAL DEFAULT 300
  Type         TEXT DEFAULT 'Personal'  -- Personal | Archetype | DreamSign
  IsHome       INTEGER DEFAULT 0    -- 1 = centralni node (samo eden)
  ParentId     INTEGER DEFAULT NULL -- hierarhija (1 nivo)
  SearchTerms  TEXT DEFAULT NULL    -- "soteska, sotesko, soteski" (csv)
  Notes        TEXT DEFAULT NULL    -- prosta opomba

Atlas_Warps
  Id           INTEGER PK AUTOINCREMENT
  FromNodeId   INTEGER
  ToNodeId     INTEGER
  IsOneWay     INTEGER DEFAULT 0

Atlas_NodeDreams
  Id           INTEGER PK AUTOINCREMENT
  NodeId       INTEGER NOT NULL
  SleepCycleId INTEGER NOT NULL
  PinnedAt     TEXT                 -- datum pripenjanja
  Note         TEXT
  UNIQUE(NodeId, SleepCycleId)
```

> **Home node je fiksen v svetovnem prostoru** — njegova X/Y se, tako kot pri vsakem drugem nodu, bere iz baze in se ne spreminja glede na velikost zaslona. Centriranje Home-a na sredino zaslona (ob nalaganju, ob resetu kamere) je izključno stvar kamere, ne premika samega noda — relativne pozicije med Home in ostalimi nodi zato ostanejo konsistentne ne glede na velikost okna.

---

## Iskanje sanj

Iskanje je **besedilno** — Orbis išče po `SleepCycle.Contents` z besednimi mejami.

```
"soteska" → poišče: "soteska ...", "... soteska ...", "... v soteska ..."
           NE poišče: "predgorje", "nasoteska" (ni besedna meja)
```

Prioriteta iskalnih izrazov:
1. `SearchTerms` — eksplicitni seznam (csv), če je nastavljen
2. `Name` — ime noda, kot fallback

Ročno pripete sanje (`Atlas_NodeDreams`) se vedno prikažejo, neodvisno od iskalnih izrazov.

Ista iskalna logika (search terms ↔ vsebina sanje) se uporablja tako za mehurčke na karti kot za Timeline — kar pomeni, da se lokacija na obeh mestih obnaša konsistentno.

---

## Warpi (povezave med lokacijami)

- **Ustvari**: Ctrl+klik na prvo lokacijo, nato Ctrl+klik na drugo — ali prek Master View panela z iskalnikom.
- **Enosmerno / dvosmerno**: privzeto dvosmerno (`↔`); v Master View lahko preklopiš na enosmerno (`→`) in obrneš smer (`⇄`).
- **Izbriši**: `✕` poleg posamezne povezave v Master View.

Vse štiri warp operacije (ustvari/izbriši/preklopi/obrni) so optimizirane, da osvežijo samo prizadeti del UI-ja (seznam povezav), ne celotne karte — na večjih bazah je razlika v odzivnosti opazna.

---

## Timeline

- **Filter**: Vse lokacije / Samo tiste na karti ("Orbis only")
- **Top**: omeji prikaz na N najpogostejših lokacij po številu sanj, ali "Vse" brez omejitve
- Ob Filter/Top kontrolah je **counter**, ki pove, koliko lokacij je trenutno prikazanih glede na izbrano kombinacijo (npr. `Prikazano: 50/81`)
- Klik na piko odpre pripadajočo sanjo neposredno v Master View
- Timeline se znova zgradi samo, če so se podatki od zadnjega ogleda dejansko spremenili (ni nepotrebnega re-buildanja ob vsakem preklopu)

---

## Pozabljanje (stability decay)

Opcijska simulacija postopnega slabljenja stabilnosti neaktivnih lokacij:

- Nastavljiva hitrost padca: blago / srednje / agresivno
- DOM node lahko izvzameš iz izračuna
- Referenčna točka je današnji datum ali datum zadnjega vnosa v dnevnik, karkoli je bolj smiselno glede na to, kako sveža je baza

---

## Keyboard shortcuts

| Tipka | Akcija |
|---|---|
| `Ctrl+F` | Iskanje nodov |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` | Redo |
| `Ctrl+S` | Izvozi bazo |
| `Ctrl+0` | Reset kamere na home |
| `G` | Pojdi na Home node |
| `T` | Preklopi Timeline |
| `I` | Odpri/zapri Insights |
| `D` | Fokus na Discovery search |
| `M` | Toggle minimap |
| `H` | Toggle Help (in dostop do Changelog) |
| `+` / `-` | Zoom (odstotek viden v statusni vrstici) |
| `0` | Reset zoom |
| `Esc` | Zapri **zadnje odprto** okno (ne vseh naenkrat) |

---

## Barvna shema

| Barva | Hex | Uporaba |
|---|---|---|
| Accent / Personal | `#66fcf1` | Osnovni nodi, UI poudarki |
| Warp | `#ff4d4d` | Povezave med nodi |
| Archetype | `#f7d794` | Arhetipski nodi |
| Dream Sign | `#a29bfe` | Dream Sign nodi |
| Home | `#ffd700` | DOM node |
| Background | `#050506` | Ozadje |
| Card | `#111216` | Paneli, kartice |
| Muted | `#555` | Sekundarno besedilo |

CSS custom properties:
```css
--bg: #050506
--card: #111216
--accent: #66fcf1
--warp: #ff4d4d
--archetype: #f7d794
--ds: #a29bfe
--home: #ffd700
--text: #c5c6c7
--muted: #555
--border: #45a29e33
--group: rgba(102,252,241,0.06)
```

---

## Changelog

Poln seznam sprememb po verzijah: [`CHANGELOG.md`](./CHANGELOG.md), ali znotraj aplikacije: `H` (Help) → link poleg naslova. Besedilo sledi izbranemu jeziku (SL/EN).

Za tehnične podrobnosti (arhitektura, i18n mehanizem, znane omejitve) glej [`ORBIS_GUIDE.md`](./ORBIS_GUIDE.md).
