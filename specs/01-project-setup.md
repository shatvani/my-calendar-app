# Spec 01 – Project Setup (Udemy Lecture 2)

## Branch
`feature/01-project-setup`

## Forrás
- `Project Preview.md` – 1. lecke, csak áttekintés, nincs kódolási feladat.
- `Project Setup.md` – 2. lecke transcript, ez a spec alapja.
- `calendar.png` – célkép, később kell (layout/styling leckéknél).
- `Calendar App/calendar-app` – kész referencia-megoldás, csak konvenció-ellenőrzésre használjuk, nem másoljuk.

## Cél
Tiszta Vite + React (JS) projekt létrehozása, boilerplate eltávolítva, egy üres `CalendarApp` komponens bekötve. Nincs még naptár-logika vagy JSX layout – az a 3. lecke.

## Követelmények
1. Új projekt: `npm create vite@latest my-calendar-app -- --template react`, a `Calendar App` mappa testvéreként (attól függetlenül).
2. `npm install` a projektben.
3. Boilerplate takarítás:
   - public/favicon.svg
   - public/icons.svg
   - src/assets/          (teljes mappa: react.svg, vite.svg, hero.png)
   - src/App.css
4. `index.html`:
   - favicon link törlése
   - `<title>` → `Calendar App`
   - boxicons CDN link hozzáadása (később ikonokhoz kell):
     `<link href="https://unpkg.com/boxicons@2.1.4/css/boxicons.min.css" rel="stylesheet" />`
   - Cseréld a <head> tartalmát erre:
      ```html
      <head>
      <meta charset="UTF-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <link href="https://unpkg.com/boxicons@2.1.4/css/boxicons.min.css" rel="stylesheet" />
      <title>Calendar App</title>
      </head>
      ```
5. `src/App.jsx`: alapértelmezett Vite tartalom törlése, helyette
   ```jsx
   import CalendarApp from './Components/CalendarApp'
   import './Components/CalendarApp.css'

   const App = () => {
     return (
       <div className="container">
         <CalendarApp />
       </div>
     )
   }

   export default App
   ```
6. Új mappa: `src/Components/`
   - `CalendarApp.jsx` – üres funkcionális komponens (arrow function, default export), egyelőre csak egy `<div>Calendar App</div>` vagy hasonló placeholder.
   - `CalendarApp.css` – üres fájl, később töltjük fel.
   - src/Components/CalendarApp.jsx:
      ```jsx
      const CalendarApp = () => {
        return (
          <div>Calendar App</div>
        )
      }

      export default CalendarApp
      ```
7. `src/main.jsx` – nem módosul, marad a Vite alapértelmezett bootstrap.

## Elfogadási kritérium
- `npm run dev` hibátlanul elindul, a böngészőben megjelenik a placeholder tartalom `localhost`-on.
- A mappa/fájl struktúra megegyezik a referencia-megoldás konvencióival (`Components` mappa nagybetűvel, `CalendarApp.jsx`/`.css` elnevezés).

## Nem tartozik ide (későbbi leckék)
- Naptár rács JSX-e (3. lecke)
- Esemény popup, state kezelés
- Stílusozás a `calendar.png` alapján
