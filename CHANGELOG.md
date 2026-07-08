# Changelog

Vse pomembnejše spremembe Orbisa so zabeležene tukaj. Trenutna verzija je vidna tudi
v aplikaciji zgoraj levo (klikni nanjo za isti seznam znotraj UI-ja).

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
