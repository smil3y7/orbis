# Changelog

Vse pomembnejše spremembe Orbisa so zabeležene tukaj. Isti seznam je viden tudi
znotraj aplikacije: `H` (Help) → link poleg naslova.

## v9.0.14 — 2026-07-21

- Megla se zdaj ob kreiranju nove lokacije animirano razkrije (ease-out,
  ~1.1s) namesto da se luknja pojavi takoj v polni velikosti — "razpoka v
  megli", ki se širi navzven od nove lokacije.
- Razpon radija luknje glede na Stability razširjen iz 89–170px na 60–240px
  — padec stabilnosti (Pozabljanje) se zdaj vidno pozna: megla dejansko
  "pogoltne" zanemarjeno lokacijo nazaj, namesto da se luknja samo rahlo
  skrči.
- Vseh 9 mest, ki so prej uporabljala blokirajoč native alert(), zdaj
  uporablja obstoječi, neblokirajoč toast sistem.

## v9.0.13 — 2026-07-20

- Po popravku pozicije megle (9.0.12) je bila megla še vedno praktično
  nevidna: risala je čisto črno (#000), ozadje strani pa je --bg:#050506 —
  razlika samo 5 stopničk sivine, neopazno na katerem koli zaslonu. Megla je
  zdaj polprosojna siva (rgba(32,34,40,0.82)), ki dejansko vidno kontrastira
  z ozadjem. Mehanika "luknja se reže okoli vsakega noda" (torej manj megle z
  vsakim dodanim nodom) je ostala nespremenjena — obstajala je že prej, samo
  ni je bilo mogoče videti.

## v9.0.12 — 2026-07-20

- Klik na "Megla" v Layer kontrolniku je preklopil stanje, a se vizualno ni
  nič spremenilo. Vzrok: drawFog() je črn pravokotnik vedno risala pri
  SVETOVNEM izhodišču (0,0), ne glede na to, kam je kamera dejansko poravnana
  (camX/camY) — dokler je bil Home node zaradi starega centerHomeNode()
  bug-a (glej 9.0.9) vedno prisiljen na canvas.width/2, je bila kamera po
  resetu vedno na (0,0), kar je to napako po naključju skrilo. Po popravku
  Home pozicije v 9.0.9 je camera pan skoraj vedno neničeln, zato je megla
  "izginila" — v resnici je bila samo narisana izven vidnega izreza. drawFog
  zdaj upošteva camX/camY in pravilno pokrije dejansko viden del karte.

## v9.0.11 — 2026-07-12

- Iskanje sanj je zgrešilo ujemanja, kadar se je iskalna beseda v besedilu
  pojavila z veliko začetnico (npr. "Čas" na začetku stavka, iskalni izraz
  "čas") — SQLite-ov vgrajeni LOWER() folda samo ASCII, šumnike (č/š/ž) pusti
  nespremenjene. Ujemanje je zdaj v celoti prestavljeno v JS (isti regex
  pristop, ki ga Timeline že uporablja), kjer .toLowerCase() šumnike pravilno
  obravnava. Popravljeno v vseh štirih mestih, ki so prej gradila SQL WHERE z
  LOWER()/LIKE: štetje sanj na node, seznam sanj v Master View (2 mesti), in
  ročno iskanje sanj za pripenjanje.

## v9.0.10 — 2026-07-10

- saveSearchTerms() ni osvežil frekvenčnega predpomnilnika (barva mehurčka po
  pogostosti) ob shranjevanju — ostal je zastarel do naslednjega nepovezanega
  refresha. Nov refreshNodeFreq() naredi ciljan recompute samo za ta node
  (freqMax in barve vseh preostalih se preračunajo iz že predpomnjenih
  count-ov, brez dodatnih SQL klicev).
- saveNodeEdits() (samo ime/tip) je po nepotrebnem klical poln refreshData()
  — enak razred nepotrebnega dela kot warp fix v 9.0.6, samo ni bil pokrit.
  Zdaj osveži samo tisto, kar je dejansko odvisno od imena (warp dropdowni,
  insights, timeline).
- flipWarp() in toggleWarpOneWay() zdaj podpirata undo/redo (create/delete
  warpa sta ga že imela, direction-toggle ne).
- Dodana startup diagnostika (console.warn), ki opozori, če sl.js in en.js
  nimata popolnoma istega nabora ključev — brez tega manjkajoč ključ tiho
  pade nazaj na goli ključ namesto berljivega besedila.
- Ime aplikacijske datoteke je zdaj stabilno (`orbis.html`, ne več
  `orbis_vX_Y_Z.html`) — `vercel.json` je posodobljen enkrat za vselej in se
  mu odslej ni več treba ročno slediti ob vsakem version bumpu.

## v9.0.9 — 2026-07-09

- Home node je imel v resnici NESTABILNO svetovno pozicijo: centerHomeNode() je
  vsak frame (in ob vsakem resize-u) prepisal h.x/h.y na trenutno
  canvas.width/2, canvas.height/2 — namesto da bi Home imel fiksno pozicijo iz
  baze kot vsak drug node. Na drugačni velikosti zaslona se je Home dejansko
  premaknil v svetovnem prostoru, medtem ko so ostali nodi ostali pri svojih
  shranjenih koordinatah — relativne pozicije so se razbile. Home zdaj obdrži
  svojo pravo, fiksno X/Y iz baze; centriranje na zaslon je izključno stvar
  kamere (resetCamera), ne premika samega noda. Preverjeno s headless testom
  pri treh različnih velikostih zaslona — relativni odmik med Home in ostalimi
  nodi zdaj ostane enak ne glede na velikost zaslona.

## v9.0.8 — 2026-07-08

- Timeline counter iz 9.0.7 je bil samo gola številka brez oznake (npr. "81"
  med dvema ločilnima črtama) — lahko ga je spregledati, saj ni bilo videti
  kot label. Dodana predpona "Prikazano:" ("Shown:" v EN), da je jasno, kaj
  številka pomeni. Preverjeno s headless brskalnikom: logika je bila pravilna
  že v 9.0.7 (izračun in izris sta delovala), šlo je zgolj za vizualno
  diskretnost.

## v9.0.7 — 2026-07-08

- Popravljeni trije jezikovni spodrsljaji: (1) datum v statusu Pozabljanja je
  bil vedno formatiran po slovenskih pravilih (npr. ime meseca), tudi v
  angleškem načinu; (2) enako za datum v Timeline tooltipu ob kliku na piko;
  (3) "Baza naložena: ime.db" pod izbirnikom datoteke je ostal v jeziku, v
  katerem si nazadnje naložil bazo, tudi po preklopu na drug jezik (isti vzrok
  kot pri changelog/insights, popravljenih v 9.0.4/9.0.5).
- Timeline: ob Filter/Top kontrolah je zdaj counter, ki pove, koliko lokacij
  je trenutno prikazanih (npr. "50/81", ali samo "81" če Top ne odreže
  ničesar).

## v9.0.6 — 2026-07-05

- Warp create/delete/flip/toggle-oneway so bili opazno počasni, ker je vsaka od
  teh operacij sprožila polni refreshData() — ta pa za VSAK node v bazi znova
  prešteje sanje (SQL LIKE-scan čez celotno SleepCycle tabelo), čeprav warpi
  nimajo nobene zveze s sanjami. Nova lahka refreshWarpsOnly() posodobi samo
  warpe; master view po warp operaciji zdaj osveži samo seznam povezav
  (renderConnectionsSection), ne celega panela.
- Timeline se je znova zgradil od začetka ob vsakem vstopu vanj, tudi če se od
  zadnjega ogleda ni spremenilo nič. Nova "tlDirty" zastavica: rebuild (in
  loader) se sproži samo, če so se nodi/warpi/sanje dejansko spremenili od
  zadnjega izrisa.
- Dodan prikaz trenutnega zoom nivoja karte (%) v status vrstici, med gumbi za
  zoom — prej ni bilo nobenega indikatorja, koliko je karta trenutno
  povečana/pomanjšana.

## v9.0.5 — 2026-07-05

- Esc zdaj zapre samo zadnje odprto okno, ne vseh naenkrat. Prej sta obstajala
  DVA ločena keydown listenerja, vsak je ob Esc zaprl svoj trdo kodiran seznam
  panelov (in tretji vir istega buga: iskalno polje na Timeline je imelo še
  svoj lasten Esc handler, ki je dogodek spustil naprej do obeh listenerjev).
  Zdaj vse skupaj upravlja en sam "panel stack": vsak odprt panel (master view,
  help, changelog, insights, iskanje lokacij, timeline) se potisne na vrh, Esc
  zapre samo vrh, naslednji Esc pa naslednjega spodaj.
- Timeline je zdaj tudi del tega sklada — Esc med gledanjem Timeline-a zapre
  nazaj na karto (prej je bilo to mogoče samo s klikom na "✕ Karta").

## v9.0.4 — 2026-07-05

- Timeline: dropdown "Top" je prej tiho odrezal seznam lokacij na najboljših
  30/50/80 po številu sanj, tudi ko je bilo lokacij več (npr. pri 81 se je nekaj
  vedno izgubilo, brez opozorila). Dodana je nova privzeta opcija "Vse" — brez
  omejitve, virtualizacija vrstic poskrbi, da to ne vpliva na hitrost izrisa.
- Changelog premaknjen iz glave (kjer je klik na verzijo kar kričal zase) v
  diskreten link znotraj Help panela; besedilo je zdaj dvojezično (SL/EN) in
  sledi izbiri jezika kot vsa ostala vsebina.

## v9.0.3 — 2026-07-05

- Timeline sanje zdaj veže na perception bubbles prek istega besedilnega ujemanja
  (search terms ↔ vsebina sanje) kot povsod drugje v aplikaciji — prej je uporabljal
  uvoženo polje lokacije spanja, kar je dalo napačne skupine in "ni v atlasu" napake.
- Timeline gradnja pospešena (eno SQL branje + regex ujemanje v pomnilniku namesto
  ločene poizvedbe na bubble) — odpravlja zamrznitev UI pri bazah s tisoči sanj.
- Klik na piko na Timeline zdaj odpre točno tisto sanjo (razširjeno, poskrolano,
  osvetljeno) v master view-u, ne le splošno lokacijo.
- Timeline: vertikalno drsenje in virtualizacija vrstic namesto stiskanja vseh
  lokacij v fiksno višino — berljivo tudi pri 100+ bubblih.
- Timeline: navaden scroll zdaj drsi po vrsticah, Ctrl/Cmd+Scroll zoomira časovno os.
- Timeline: Ctrl+F iskanje lokacij zdaj poskrola in osvetli ustrezno vrstico
  (namesto da je delovalo samo na karti).
- Popravljen skrit konflikt: scroll na karti brez odprtega Timeline-a je vseeno tiho
  spreminjal kamero karte, tudi ko si gledal Timeline.
- Dodan modal "Nalagam bazo…" ob nalaganju/menjavi baze.
- Verzija in changelog zdaj iz enega vira (`APP_VERSION` + `CHANGELOG` v HTML-ju) —
  prej je bila verzija hardkodirana ločeno v `<title>`, v HTML fallback besedilu IN
  v obeh `lang/*.js` prevodih, zato se je pri bumpu skoraj vedno kje pozabila
  posodobiti.

## v9.0.2 in starejše

Starejše spremembe niso bile beležene v tej datoteki.
