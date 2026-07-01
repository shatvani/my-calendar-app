# Spec 02 – Creating JSX (Udemy Lecture 4)

## Branch
`feature/02-creating-jsx`

## Forrás
- `Creating JSX.md` – 4. lecke transcript, ez a spec alapja.
- `calendar.png` – célkép (a napok száma ebből jön, ld. lentebb).
- Referencia (`Calendar App/calendar-app`) – csak class-név konvenciók ellenőrzésére, NEM másoljuk 1:1, mert az a végleges, dinamikus (useState-es) állapotot tartalmazza. Ez a lecke még csak a statikus, hardcode-olt JSX-ről szól.

## Cél
A `CalendarApp` komponens teljes, **statikus** JSX struktúrájának felépítése: naptár fejléc, hónap/év navigáció, hét napjai, napok rácsa, esemény popup, egy minta esemény. Még **nincs** `useState`, `onClick`, `.map()` vagy dinamikus adat – ez csak a HTML-szerű váz, kemény kódolt (hardcoded) értékekkel. A state-kezelés és interaktivitás egy későbbi leckében jön.

## Kiindulás
Az `App.jsx`-ben a `container` className már megvan (01-es taskból) – ezt nem kell újra hozzáadni.

## Elvárt struktúra (`src/Components/CalendarApp.jsx`)

```jsx
const CalendarApp = () => {
  return (
    <div className="calendar-app">
      <div className="calendar">
        <h1 className="heading">Calendar</h1>
        <div className="navigate-date">
          <h2 className="month">May</h2>
          <h2 className="year">2024</h2>
          <div className="buttons">
            <i className="bx bx-chevron-left"></i>
            <i className="bx bx-chevron-right"></i>
          </div>
        </div>
        <div className="weekdays">
          <span>Sunday</span>
          <span>Monday</span>
          <span>Tuesday</span>
          <span>Wednesday</span>
          <span>Thursday</span>
          <span>Friday</span>
          <span>Saturday</span>
        </div>
        <div className="days">
          <span>1</span>
          <span>2</span>
          {/* ... folytasd 3-tól 31-ig. A calendar.png alapján május 31 napos,
              a transcript "30-szor duplikálva" 30-at mondott, de a mockuphoz
              31 nap illik jobban - ez itt egy hardcoded placeholder, a pontos
              szám úgyis mindegy lesz, ha bekötjük a dinamikus logikát. */}
          <span>31</span>
        </div>
      </div>
      <div className="events">
        <div className="event-popup">
          <div className="time-input">
            <div className="event-popup-time">Time</div>
            <input type="number" name="hours" min="0" max="24" className="hours" />
            <input type="number" name="minutes" min="0" max="60" className="minutes" />
          </div>
          <textarea placeholder="Enter Event Text (Maximum 60 Characters)" maxLength="60"></textarea>
          <button className="event-popup-btn">Add Event</button>
          <button className="close-event-popup">
            <i className="bx bx-x"></i>
          </button>
        </div>
        <div className="event">
          <div className="event-date-wrapper">
            <div className="event-date">May 15, 2024</div>
            <div className="event-time">10:00</div>
          </div>
          <div className="event-text">Meeting with John</div>
          <div className="event-buttons">
            <i className="bx bxs-edit-alt"></i>
            <i className="bx bxs-message-alt-x"></i>
          </div>
        </div>
      </div>
    </div>
  )
}

export default CalendarApp
```

## Megjegyzések / miért ilyen a szerkezet
- `bx bx-*` / `bx bxs-*` osztályok a Boxicons ikonkészletből jönnek (CDN már be van kötve a 01-es taskból az `index.html`-ben).
- A `weekdays` itt teljes névvel (Sunday, Monday, ...) szerepel, nem rövidítve – ez a lecke pontosan így csinálja, a rövidítés/dinamikus tömb egy későbbi lecke témája.
- Class name, nem `class` – React JSX-ben ez kötelező, `class` attribútum hibát dob.

## Elfogadási kritérium
- `npm run dev` hibátlanul fut.
- A böngészőben megjelenik a teljes statikus naptár-váz: cím, hónap/év + két nyílikon, hét napja, napok rácsa (számozva), egy esemény popup a mezőivel, egy minta esemény szerkesztő/törlő ikonokkal.
- Még nincs stílus (CSS), úgyhogy csúnyán, egymás alatt fog kinézni – ez így helyes, a stílusozás a következő lecke.
- Kattintásra semmi nem történik még (nincs `onClick`) – ez is helyes ezen a ponton.

## Nem tartozik ide (későbbi leckék)
- CSS stílusozás
- `useState`, dinamikus hónap/nap generálás
- Esemény hozzáadás/szerkesztés/törlés logika
- Kattintás-kezelők
