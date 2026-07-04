# Spec 04 – Calendar Navigation and Days (Udemy Lecture 6)

## Branch
`feature/04-navigation-and-days`

## Forrás
- `Calendar Navigation and Days.md` – 6. lecke transcript. Ez a lecke logikáról szól (nem szín/CSS-kódról), a transcript itt megbízható, ebből dolgozunk, nem a referenciából másolunk.
- Referencia (`Calendar App/calendar-app`) – csak ellenőrzésre.

## Cél
Az eddigi hardcode-olt "May", "2024" és a statikus nap-span-ek helyett **dinamikus** renderelés `useState`-tel, plusz a hónap-navigáció (előző/következő hónap gombok) működővé tétele.

## Előkészület
A `CalendarApp.css`-ben tedd `display: none`-ná az `.event-popup`-ot egy időre (a popup megjelenítés/elrejtés majd egy jövőbeli leckében lesz state-vezérelt, addig csak útban van):
```css
.event-popup {
  display: none;
  ...
}
```

## Új logika (`src/Components/CalendarApp.jsx`)

Ezt **kézzel írd meg** – ez az első valódi `useState`/hooks gyakorlatod, ez adja a React-tanulás lényegét ennél a leckénél.

```jsx
import { useState } from 'react'

const CalendarApp = () => {
  const daysOfWeek = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday']
  const monthsOfYear = [
    'January', 'February', 'March', 'April', 'May', 'June',
    'July', 'August', 'September', 'October', 'November', 'December',
  ]

  const currentDate = new Date()

  const [currentMonth, setCurrentMonth] = useState(currentDate.getMonth())
  const [currentYear, setCurrentYear] = useState(currentDate.getFullYear())

  const daysInMonth = new Date(currentYear, currentMonth + 1, 0).getDate()
  const firstDayOfMonth = new Date(currentYear, currentMonth, 1).getDay()

  const prevMonth = () => {
    setCurrentMonth((prevMonth) => (prevMonth === 0 ? 11 : prevMonth - 1))
    setCurrentYear((prevYear) => (currentMonth === 0 ? prevYear - 1 : prevYear))
  }

  const nextMonth = () => {
    setCurrentMonth((prevMonth) => (prevMonth === 11 ? 0 : prevMonth + 1))
    setCurrentYear((prevYear) => (currentMonth === 11 ? prevYear + 1 : prevYear))
  }

  return (
    <div className="calendar-app">
      <div className="calendar">
        <h1 className="heading">Calendar</h1>
        <div className="navigate-date">
          <h2 className="month">{monthsOfYear[currentMonth]}</h2>
          <h2 className="year">{currentYear}</h2>
          <div className="buttons">
            <i className="bx bx-chevron-left" onClick={prevMonth}></i>
            <i className="bx bx-chevron-right" onClick={nextMonth}></i>
          </div>
        </div>
        <div className="weekdays">
          {daysOfWeek.map((day) => (
            <span key={day}>{day}</span>
          ))}
        </div>
        <div className="days">
          {[...Array(firstDayOfMonth).keys()].map((_, index) => (
            <span key={`empty-${index}`} />
          ))}
          {[...Array(daysInMonth).keys()].map((d) => (
            <span
              key={d + 1}
              className={
                d + 1 === currentDate.getDate() &&
                currentMonth === currentDate.getMonth() &&
                currentYear === currentDate.getFullYear()
                  ? 'current-day'
                  : ''
              }
            >
              {d + 1}
            </span>
          ))}
        </div>
      </div>
      <div className="events">
        {/* event-popup és event JSX változatlan marad az előző leckéből */}
      </div>
    </div>
  )
}

export default CalendarApp
```

(Az `events` szekció JSX tartalma – event-popup + event – nem változik, hagyd úgy, ahogy a 02-es taskban megírtad.)

## Miért ez a logika

- `daysInMonth`: `new Date(currentYear, currentMonth + 1, 0)` egy trükk – a `0` nap a *következő* hónapban valójában az *aktuális* hónap utolsó napjára esik vissza, így `.getDate()` megadja, hány napos a hónap.
- `firstDayOfMonth`: `new Date(currentYear, currentMonth, 1).getDay()` az adott hónap 1-jének hét napját adja vissza (0 = vasárnap ... 6 = szombat) – ennyi üres span kell a rács elejére, hogy a hónap 1-je a helyes oszlopba essen.
- Az üres span-ek generálása (`[...Array(firstDayOfMonth).keys()]`) és a nap-span-ek generálása (`[...Array(daysInMonth).keys()]`) ugyanazt a mintát követi: `Array(n)` egy `n` hosszú, `undefined` elemekkel teli tömböt csinál, `.keys()` az indexeket adja vissza iterátorként, a spread (`[...]`) tömbbé alakítja, a `.map()` pedig JSX elemekre képezi le.
- A `key` prop miért kell: React ebből tudja megkülönböztetni a lista elemeit újrarendereléskor – enélkül figyelmeztetést kapsz a konzolban, és bugos lehet a lista frissítése.
- `prevMonth`/`nextMonth`: a functional update forma (`setCurrentMonth((prevMonth) => ...)`) azért kell, mert a state-frissítés aszinkron – ha közvetlenül a `currentMonth` változóra hivatkoznál a `setCurrentYear` hívásban, elavult (stale) értéket kapnál. Ezért a `setCurrentYear` feltétele is a *külső* `currentMonth`-ra néz (ami még a kattintás pillanatában érvényes érték), nem a `prevYear`-re.
- A jelenlegi nap kiemelése (`current-day` class) három feltétel egyidejű teljesülését nézi: nap, hónap ÉS év is egyezzen a mai dátummal – enélkül pl. minden hónap 2-ára rákerülne a kiemelés.

## Elfogadási kritérium
- `npm run dev` hibátlanul fut, nincs konzol-hiba (pl. hiányzó `key` warning).
- A hónap/év fejléc a mai dátumot mutatja (helyi gépi dátum szerint, nem feltétlenül "May, 2024").
- A nyilakra kattintva működik az előre/hátra navigáció, évváltáskor is helyesen (december → január, január → december).
- A mai nap ki van emelve a rácsban (`current-day` stílus).
- Az esemény popup egyelőre nem látszik (`display: none`).

## Nem tartozik ide (későbbi leckék)
- Dátumra kattintás → popup megnyitása (state-vezérelt megjelenítés)
- Esemény hozzáadás/szerkesztés/törlés
