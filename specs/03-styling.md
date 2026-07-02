# Spec 03 – Customizing and Styling the App (Udemy Lecture 5)

## Branch
`feature/03-styling`

## Forrás
- `Customizing and Styling the App.md` – 5. lecke transcript (a "miért" magyarázatokhoz).
- Referencia (`Calendar App/calendar-app/src/index.css` és `src/Components/CalendarApp.css`) – a **pontos** cél-tartalom. A transcript hangfelismerése sok hex-színkódot és számot elrontott (pl. "23542" → valójában `#2c3542`), ezért ennél a leckénél a referenciát másoljuk, nem a transcriptből gépeljük vissza a számokat.

## Döntés
Megegyeztünk, hogy ezt a kört úgy csináljuk, hogy a kész projektből másolod be a CSS-t (mivel ez nem React-specifikus tudás, hanem általános CSS, és a pontos értékek úgyis a referenciából valók).

## Teendő
1. **`src/index.css`** – teljes tartalom **lecserélése** (a jelenlegi, Vite-demóból megmaradt tartalom kikerül):
   - Google Fonts import (Bebas Neue + Comfortaa)
   - globális reset (`* { margin:0; padding:0; box-sizing:border-box; outline:none; font-family:... }`)
   - `html { font-size: 62.5%; }` – ez teszi, hogy 1rem = 10px legyen a szokásos 16px helyett, könnyebb fejszámolás
   - `.container` stílusai (100vh, grid + place-items center, perspective)
   - mobil media query a font-size-hoz

2. **`src/Components/CalendarApp.css`** – teljes tartalom **másolása** a referenciából (üres fájl volt eddig).

Másold át a két fájl tartalmát a referencia projektből (`Calendar App/calendar-app/src/...`) a sajátodba.

## A "miért" – amit érdemes érteni, nem csak bemásolni

- **`html { font-size: 62.5% }` + `rem`**: mivel az összes méret ezután `rem`-ben van megadva, ez a trükk 1rem-et 10px-re konvertál (16px × 62.5% = 10px), így a rem-értékek fejben is könnyen pixelre válthatók.
- **`clamp(min, preferred, max)`**: reszponzív betűméretezés media query nélkül – a middle érték `cqi` (container query inline) egységben van, ami a szülő konténer szélességéhez igazodik, nem a viewporthoz. Így a szöveg a `.calendar-app` konténer méretéhez skálázódik, nem a böngészőablakhoz.
- **`display: grid; place-items: center;`** a `.container`-en: ez az egysoros módja annak, hogy egy elemet vízszintesen és függőlegesen is középre igazíts.
- **A 3D árnyék-trükk** (`.calendar-app::after`): `perspective` a szülőn (`.container`) + `transform-style: preserve-3d` + `rotateX(50deg)` + `blur()` egy elforgatott, elmosott, félig átlátszó pszeudo-elemet hoz létre a kártya alatt, ami perspektivikus árnyékként hat.
- **`aspect-ratio`**: fix arányt tart méret helyett (pl. a naptár mindig 3:2 arányú marad, bármekkora is a szélessége).
- **Abszolút pozicionálás relatív szülőn belül**: az `.event-buttons` és `.event-popup` `position: absolute`-ot kap, a szülő (`.event`, illetve `.calendar-app`) pedig `position: relative`-et – ez a klasszikus React/CSS minta arra, hogy egy elemet a közvetlen szülőjéhez képest pozicionálj, ne az egész oldalhoz képest.
- **`::-webkit-outer/inner-spin-button { appearance: none }`**: ez tünteti el a number input alapértelmezett fel/le nyilait.
- **`::placeholder` + `:focus`**: pszeudo-osztályok/elemek a textarea placeholder stílusozására és a fókusz-állapot (kék keret, átmenet) kezelésére.

## Elfogadási kritérium
- `npm run dev` hibátlanul fut.
- A böngészőben a naptár immár a `calendar.png` mockuphoz hasonlóan néz ki: sötét kártya lekerekített sarkokkal, 3D árnyékkal, bal oldalt a naptár rács, jobb oldalt az esemény lista, a popup (egyelőre mindig látható, mert még nincs `display: none` / state-vezérlés) a bal alsó részen.
- Az oldal nem reszponzív logikailag még (a media query-k a referenciában benne vannak, ez bónusz, nem kötelező ezen a ponton ellenőrizni).

## Nem tartozik ide (későbbi leckék)
- Dinamikus hónap/nap generálás (`useState`, `.map()`)
- Esemény popup megjelenítés/elrejtés state-tel
- Esemény hozzáadás/szerkesztés/törlés logika
