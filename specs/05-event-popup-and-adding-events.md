# Spec 05 – Enhancing Event Popup and Adding New Events (Udemy Lecture 7)

## Branch
`feature/05-event-popup-and-adding-events`

## Forrás
- `Enhancing Event Popup and Adding New Events.md` – 7. lecke transcript. Logikáról szól, megbízható forrás, ebből dolgozunk.
- Referencia – csak ellenőrzésre. **Fontos**: a referenciában már benne van a szerkesztés/törlés (`editingEvent`, `handleEditEvent`, `handleDeleteEvent`) és a `handleTimeChange` egységes, `padStart`-tal ellátott verziója is – ezek **nem** ehhez a leckéhez tartoznak, azok egy következő task lesznek. Ennél a spec-nél szándékosan az egyszerűbb, csak-hozzáadás verziót írjuk meg, a transcriptnek megfelelően.

## Cél
- Kattintásra megjelenjen az esemény popup, de csak a mai vagy jövőbeli napokon (múltbeli napon ne).
- Bezárás gomb működjön.
- Az óra/perc mezők és a szövegdoboz legyenek "controlled" komponensek (React state vezérelje az értéküket).
- "Add Event" gombra az esemény bekerüljön a listába, és megjelenjen a jobb oldalon.

## Új state-ek

```jsx
const [selectedDate, setSelectedDate] = useState(currentDate)
const [showEventPopup, setShowEventPopup] = useState(false)
const [events, setEvents] = useState([])
const [eventTime, setEventTime] = useState({ hours: '00', minutes: '00' })
const [eventText, setEventText] = useState('')
```

**Eltérés a transcripttől**: a transcript "nulla" értéket mond az `eventTime` alapértékéhez, ami hangzás alapján lehetne szám (`0`) is. Én **stringet** (`'00'`) használok, mert a `handleEventSubmit`-ban a `padStart` egy string metódus – ha `eventTime.hours` szám maradna (mert a felhasználó meg sem érinti az input mezőt, mielőtt rögtön "Add Event"-re kattint), a `.padStart()` hívás elszállna egy `TypeError`-ral. Stringgel ez a hiba nem fordulhat elő.

## Új segédfüggvények és logika

```jsx
const isSameDay = (date1, date2) => {
  return (
    date1.getFullYear() === date2.getFullYear() &&
    date1.getMonth() === date2.getMonth() &&
    date1.getDate() === date2.getDate()
  )
}

const handleDayClick = (date) => {
  const clickedDate = new Date(currentYear, currentMonth, date)
  const today = new Date()

  if (clickedDate >= today || isSameDay(clickedDate, today)) {
    setSelectedDate(clickedDate)
    setShowEventPopup(true)
    setEventTime({ hours: '00', minutes: '00' })
    setEventText('')
  }
}

const handleEventSubmit = () => {
  const newEvent = {
    date: selectedDate,
    time: `${eventTime.hours.padStart(2, '0')}:${eventTime.minutes.padStart(2, '0')}`,
    text: eventText,
  }

  setEvents([...events, newEvent])
  setEventTime({ hours: '00', minutes: '00' })
  setEventText('')
  setShowEventPopup(false)
}
```

**Miért kell az `isSameDay`, és miért nem elég a `clickedDate >= today`?** A `clickedDate` éjfélkor (00:00:00) jön létre (`new Date(year, month, day)` alapból éjfélre teszi az időt), a `today = new Date()` viszont a *pontos pillanatnyi* időt tartalmazza (pl. 14:32:07). Emiatt a mai napra kattintva `clickedDate < today` lenne (mert éjfél korábban van, mint a jelenlegi óra), és a feltétel hamisan hamis lenne. Az `isSameDay` az évet/hónapot/napot hasonlítja össze az óra/perc figyelmen kívül hagyásával, így a mai nap mindig átmegy a feltételen.

## JSX módosítások

**A napok span-jaihoz kattintás-kezelő**:
```jsx
<span
  key={d + 1}
  className={...}
  onClick={() => handleDayClick(d + 1)}
>
  {d + 1}
</span>
```

**Az event-popup feltételes megjelenítése** (a `.events` div elején, az `.event-popup` div köré):
```jsx
{showEventPopup && (
  <div className="event-popup">
    ...
  </div>
)}
```
Ezzel párhuzamosan a `CalendarApp.css`-ből vedd ki a `display: none;`-t az `.event-popup`-ról – mostantól a JSX feltétel (state) dönti el, látszik-e, nem a CSS.

**Az input mezők legyenek controlled komponensek**:
```jsx
<input
  type="number"
  name="hours"
  min={0}
  max={24}
  className="hours"
  value={eventTime.hours}
  onChange={(e) => setEventTime({ ...eventTime, hours: e.target.value })}
/>
<input
  type="number"
  name="minutes"
  min={0}
  max={60}
  className="minutes"
  value={eventTime.minutes}
  onChange={(e) => setEventTime({ ...eventTime, minutes: e.target.value })}
/>
```

**A textarea is controlled, 60 karakteres limittel**:
```jsx
<textarea
  placeholder="Enter Event Text (Maximum 60 Characters)"
  value={eventText}
  onChange={(e) => {
    if (e.target.value.length <= 60) {
      setEventText(e.target.value)
    }
  }}
></textarea>
```

**A gombok**:
```jsx
<button className="event-popup-btn" onClick={handleEventSubmit}>Add Event</button>
<button className="close-event-popup" onClick={() => setShowEventPopup(false)}>
  <i className="bx bx-x"></i>
</button>
```

**Az esemény lista dinamikus renderelése** (a statikus, egyetlen hardcode-olt `.event` div helyett):
```jsx
{events.map((event, index) => (
  <div className="event" key={index}>
    <div className="event-date-wrapper">
      <div className="event-date">
        {`${monthsOfYear[event.date.getMonth()]} ${event.date.getDate()}, ${event.date.getFullYear()}`}
      </div>
      <div className="event-time">{event.time}</div>
    </div>
    <div className="event-text">{event.text}</div>
    <div className="event-buttons">
      <i className="bx bxs-edit-alt"></i>
      <i className="bx bxs-message-alt-x"></i>
    </div>
  </div>
))}
```
A szerkesztés/törlés ikonokhoz még **nem** kell `onClick` – azok egy következő lecke/task témája.

## A "miért"-ek, amikre érdemes figyelni

- **Controlled component**: az input/textarea `value`-ja a state-ből jön, és minden változást az `onChange` ír vissza a state-be – ez a React mintája arra, hogy a UI mindig tükrözze az állapotot, ne az input saját belső DOM-állapotát.
- **Spread + felülírás egy mezőn** (`{ ...eventTime, hours: e.target.value }`): az összes meglévő property-t átmásolja, majd felülírja csak a `hours`-t – enélkül a `minutes` értéke elveszne minden hours-váltáskor.
- **`key={index}` a `events.map()`-nél**: index használata kulcsként általában kerülendő, ha a lista sorrendje változhat (pl. törlés/rendezés miatt), de itt egyelőre elfogadható, mert a lista csak a végéhez ad hozzá elemeket. Ez fog megváltozni, amikor a törlés/szerkesztés bekerül.
- **`.map((event, index) => ( <div>...</div> ))` – zárójel, nem kapcsos zárójel**: ha `{ ... }`-t írnál a nyíl-függvény törzseként, az egy *function body*-t jelentene, amiben expliciten `return`-t kellene írnod a JSX elé. A `( ... )` viszont egy kifejezést zár körbe, amit a nyíl-függvény implicit módon visszaad. Ez egy klasszikus kezdő hiba, a transcript is belefutott élőben.
- **Az `Add Event` gomb `onClick` nélkül nem csinál semmit**: ez is elhangzik élőben a transcriptben (az oktató elfelejtette először) – jó példa arra, hogy egy JSX gomb önmagában nem csinál semmit, amíg nincs esemény-kezelő hozzákötve.

## Elfogadási kritérium
- `npm run dev` hibátlanul fut, nincs konzol-hiba.
- Múltbeli napra kattintva nem jelenik meg a popup, mai vagy jövőbeli napra kattintva igen.
- A Close (X) gomb bezárja a popupot.
- Az óra/perc mezőkbe lehet írni, a szövegmezőbe max. 60 karaktert.
- "Add Event"-re az esemény megjelenik a jobb oldali listában, helyes dátummal/idővel/szöveggel, és a popup bezáródik.
- Több esemény hozzáadható egymás után.

## Nem tartozik ide (későbbi leckék)
- Esemény szerkesztése (`editingEvent`, `handleEditEvent`)
- Esemény törlése (`handleDeleteEvent`)
- Egységes `handleTimeChange` (`name`/`value` alapú, `padStart`-tal az input onChange-ben)
