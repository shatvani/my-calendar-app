# Learnings – React Calendar App

Ez a projekt egy React naptár-alkalmazás (Vite + React, sima CSS), amit Udemy-kurzus alapján, spec-driven fejlesztéssel (SDD) építettem fel, feladatonként külön branch-csel és spec dokumentummal (lásd `specs/` mappa). Ez a dokumentum összefoglalja a menet közben megtanult/gyakorolt React és JavaScript koncepciókat, a talált és javított valós hibákat, és a CSS technikákat – portfólió-jegyzetként is szolgál.

## React koncepciók

**`useState` – többféle állapot-típus.** A projekt primitív (`currentMonth`, `currentYear`, `eventText`), objektum (`eventTime: { hours, minutes }`), tömb (`events`) és `null`-alapértékű (`editingEvent`, ahol `null` = "nincs szerkesztés folyamatban") state-eket is használ – jó áttekintés ad arról, hogy a `useState` bármilyen JS-értéket kezel, nem csak számokat/stringeket.

**Levezetett (derived) állapot vs. tárolt állapot.** A `daysInMonth` és `firstDayOfMonth` NEM `useState`, hanem sima `const`, mert minden renderkor kiszámolható a `currentMonth`/`currentYear`-ból. Ha ezek is state-ek lennének, minden hónapváltáskor külön szinkronizálni kellene őket – felesleges és hibalehetőség-forrás (ún. "derived state" csapda).

**Controlled komponensek.** Az input/textarea mezők értékét (`value`) a state vezérli, minden változást az `onChange` ír vissza – így a UI mindig az állapotot tükrözi, nem az input saját belső DOM-állapotát.

**Feltételes renderelés (`&&`).** Az esemény popup csak akkor jelenik meg, ha `showEventPopup` igaz: `{showEventPopup && (<div className="event-popup">...</div>)}`.

**Lista renderelés `.map()`-pel és `key`-jel.** Az események, a hét napjai és a hónap napjai is `.map()`-pel generálódnak. Tanulság: amíg a listához csak hozzáadunk, az index is jó kulcs lehet, de amint törölni/szerkeszteni is kell egy konkrét elemet, stabil azonosító (`id: Date.now()`) kell, mert az index törléskor "elcsúszhat".

**Functional state update és a stale closure elkerülése.** A `prevMonth`/`nextMonth` függvényekben a `setCurrentMonth((prevMonth) => ...)` forma azért kell, mert a state-frissítés aszinkron: a closure-ből olvasott `currentMonth` csak az adott render pillanatában érvényes, nem feltétlenül a legfrissebb érték egy esetleges gyors, egymást követő kattintásnál.

**Computed property name state-frissítéshez.** A `handleTimeChange` egyetlen függvénnyel kezeli mind az óra, mind a perc mezőt: `setEventTime((prev) => ({ ...prev, [name]: normalized }))` – a `name` attribútumból (`"hours"`/`"minutes"`) tudja, melyik mezőt frissítse.

**Props / lifting state up – tudatosan elhalasztva.** A `CalendarApp` jelenleg egyetlen, nagy komponens. Felmerült, hogy szét kellene bontani (fejléc, napok rácsa, esemény popup, esemény lista külön komponensekbe), de ez props-átadást és "lifting state up"-ot igényelne – ezt tudatosan egy külön, jövőbeli refaktor-feladatra hagytuk (lásd lentebb).

## JavaScript technikák

**`[...Array(n).keys()]` tartomány-generáláshoz.** `Array(n)` egy "lyukas" (üres slot-okkal teli) tömböt csinál, amit a `.map()`/`.forEach()` átugorna. A `.keys()` az indexeket adja iterátorként (lyukaktól függetlenül), a spread (`[...]`) pedig valódi, map-elhető tömbbé alakítja. Modernebb alternatíva: `Array.from({ length: n })`.

**Spread + felülírás – immutable update minta.** React state-et sosem mutálunk közvetlenül; mindig új objektumot/tömböt hozunk létre (`{ ...prev, mező: újÉrték }`, `[...events, newEvent]`), mert React referencia-egyenlőség alapján dönti el, változott-e valami.

**Date-aritmetika trükkök.** `new Date(year, month + 1, 0).getDate()` a hónap napjainak számát adja (a "0. nap" az előző hónap utolsó napjára esik vissza). `new Date(year, month, 1).getDay()` a hónap első napjának hét napját adja, ebből számoljuk ki, hány üres cella kell a rács elejére.

**`isSameDay` – időzóna/időpont-összehasonlítási csapda.** A `clickedDate >= today` önmagában hibás a mai napra, mert a kattintott dátum éjfélre áll, a `today` viszont a pontos aktuális időt tartalmazza. Külön függvény kell, ami csak év/hónap/nap szerint hasonlít, óra/perc nélkül.

**`parseInt` normalizálás + `Math.min`/`Math.max` clamp minta.** Az óra/perc mezőknél talált valós hiba: a `padStart` minden billentyűleütésnél lefuttatva a nyers, felhalmozódó szövegen (`"00"` + `"1"` → `"001"`) hibás eredményt ad. Megoldás: `parseInt` a beírt érték normalizálására, majd `Math.min(Math.max(érték, min), max)` a `[min, max]` tartományba szorításhoz (a HTML `min`/`max` attribútum önmagában csak a léptető nyilakra hat, kézi gépelést nem korlátoz).

**`Array.prototype.sort` stabilitása.** Valós hiba: az események eredetileg csak dátum szerint rendeződtek, az egy napon belüli sorrend a felviteli sorrendet követte, mert a `sort` stabil, és az `event.date` nem tartalmazta az órát/percet. Javítás: dátum + idő kombinálása egyetlen összehasonlítható időbélyeggé (`toTimestamp` segédfüggvény).

**`padStart` formázáshoz.** `"5".padStart(2, '0')` → `"05"` – konzisztens, két számjegyű időformátumhoz.

## Valós hibák, amiket megtaláltunk és kijavítottunk

Ezek jó beszédtémák egy interjún, mert valós, tesztelés közben felfedezett hibák voltak, nem csak "tankönyvi" gyakorlatok:

1. Percmező 60+ értéket engedett (`102`) – a HTML `max` attribútum nem korlátozza a kézi gépelést, kellett egy explicit clamp a `handleTimeChange`-ben.
2. Az óra/perc mező nullákat halmozott gépeléskor (`"012"` egy kétjegyű szám helyett) – a `padStart` minden leütésnél a nyers, nem normalizált szövegen futott.
3. Egy napon belüli események nem idő szerint jelentek meg – a rendezés csak a dátumot nézte, az időt nem.
4. Üres eseményszöveggel is menthető volt az esemény – hiányzó validáció, pótoltuk egy állapot-vezérelt hibaüzenettel.
5. A hét napjainak neve egymásba ért keskenyebb (laptop) képernyőn – mert a transcriptet szó szerint követve teljes napneveket (`Sunday`, `Wednesday`) használtunk a referencia rövidített (`Sun`, `Wed`) verziója helyett, ami a CSS-t túlfeszítette.
6. `git` munkakönyvtár-sérülés (index/reflog) többször előfordult a szinkronizált workspace-mappa törlésvédelme miatt – tanulság: git-műveleteket mindig a valódi, helyi fájlrendszeren érdemes futtatni, nem szinkronizált/hálózati mappán át.

## CSS technikák

- **`html { font-size: 62.5% }`**: 1rem = 10px konvenció, könnyebb fejszámolás a rem-alapú méretezéshez.
- **`clamp(min, preferred, max)` `cqi` (container query inline) egységgel**: reszponzív tipográfia media query nélkül, a szülő konténer szélességéhez igazodva.
- **CSS Grid középre igazítás**: `display: grid; place-items: center;`.
- **3D árnyék-trükk**: `perspective` a szülőn + `transform-style: preserve-3d` + `rotateX()` + `blur()` egy pszeudo-elemen (`::after`).
- **Abszolút pozicionálás relatív szülőn belül**: `.event-buttons`/`.event-popup` a közvetlen szülőjükhöz képest pozicionálva.
- **Pszeudo-elemek/osztályok**: `::placeholder`, `:focus`, `::-webkit-scrollbar` (görgetősáv elrejtése), `::-webkit-outer/inner-spin-button` (number input nyilainak elrejtése).

## Munkafolyamat

A projekt feature-branchekben készült (`feature/01-project-setup` ... `feature/06-editing-and-deleting-events`), mindegyikhez tartozik egy spec dokumentum a `specs/` mappában (cél, döntések, "miért" magyarázatok, elfogadási kritériumok) – ez a spec-driven fejlesztés (SDD) gyakorlása volt, AI-coworker segítséggel, valós munkafolyamat-szimulációként.

## Ismert korlátok / jövőbeli fejlesztési irányok

- **Komponens-szétbontás**: a `CalendarApp` jelenleg egyetlen nagy komponens; props és "lifting state up" bevezetésével szét lehetne bontani fejléc/napok-rács/esemény-popup/esemény-lista komponensekre.
- **Tailwind CSS átírás**: a jelenlegi kézzel írt CSS helyett megvizsgálható egy utility-first (Tailwind v4) átírás, összehasonlításképp.
- **Oxlint bevezetése**: a jelenlegi ESLint mellett/helyett kipróbálható a lényegesen gyorsabb Oxlint linter.
