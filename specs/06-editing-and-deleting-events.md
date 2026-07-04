# Spec 06 – Editing and Deleting Existing Events (Udemy Lecture 8)

## Branch
`feature/06-editing-and-deleting-events`

## Forrás
- `Editing and Deleting Existing Events.md` – 8. lecke transcript, logikáról szól, megbízható forrás.
- Referencia – ellenőrzésre használható, ennél a leckénél a végeredmény gyakorlatilag meg fog egyezni vele (a `handleTimeChange` egységesítése is most történik meg, ahogy a 05-ös spec-ben előre jeleztük).

## Cél
- Esemény szerkesztése (a ceruza ikonra kattintva a popup megnyílik az esemény adataival kitöltve, és frissíti a meglévő elemet, nem újat hoz létre).
- Esemény törlése (a törlés ikonra kattintva eltűnik a listából).
- A hónap/perc mezők egységes, közös `onChange` kezelőt kapjanak (`handleTimeChange`) a lesson 05-ös két külön inline handler helyett.
- A gomb szövege dinamikusan váltson "Add Event" / "Update Event" között.

**Megjegyzés a CSS-ről**: a görgetősáv elrejtése (`.events { overflow-y: auto }` + `::-webkit-scrollbar { display: none }`) már megvan a 03-as taskból, mert akkor a teljes referencia CSS-t bemásoltuk – itt nincs teendő.

## Új state

```jsx
const [editingEvent, setEditingEvent] = useState(null)
```

`null` = nincs szerkesztés folyamatban (új esemény létrehozása), egyébként az éppen szerkesztett esemény objektuma.

## `handleDayClick` kiegészítése

Az eddigi reset-ek mellé (`setEventTime`, `setEventText`) vegyél fel egy új sort:
```jsx
setEditingEvent(null)
```
Ez biztosítja, hogy ha egy meglévő esemény szerkesztése közben egy másik napra kattintasz, a popup "friss", "hozzáadás" módba álljon vissza, ne ragadjon benne a korábbi szerkesztés.

## `handleEventSubmit` átalakítása (create ÉS update egyben)

```jsx
const toTimestamp = (event) => {
  const [hours, minutes] = event.time.split(':').map(Number)
  const dateTime = new Date(event.date)
  dateTime.setHours(hours, minutes, 0, 0)
  return dateTime.getTime()
}

const handleEventSubmit = () => {
  if (!eventText.trim()) {
    return
  }
  
  const newEvent = {
    id: editingEvent ? editingEvent.id : Date.now(),
    date: selectedDate,
    time: `${eventTime.hours.padStart(2, '0')}:${eventTime.minutes.padStart(2, '0')}`,
    text: eventText,
  }

  let updatedEvents = [...events]

  if (editingEvent) {
    updatedEvents = updatedEvents.map((event) =>
      event.id === editingEvent.id ? newEvent : event,
    )
  } else {
    updatedEvents.push(newEvent)
  }

  updatedEvents.sort((a, b) => toTimestamp(a) - toTimestamp(b))

  setEvents(updatedEvents)
  setEventTime({ hours: '00', minutes: '00' })
  setEventText('')
  setShowEventPopup(false)
  setEditingEvent(null)
}
```

**Miért kell egyáltalán `id`?** Eddig az `events.map()`-nél a lista *indexét* használtuk kulcsként, ami rendben volt, amíg csak a végéhez adtunk hozzá elemeket. Most viszont törölni/frissíteni is kell egy *konkrét* elemet – ehhez egy stabil azonosító kell, ami nem változik meg attól, hogy közben más elemek bekerülnek vagy kikerülnek a listából (szemben az indexszel, ami törléskor "elcsúszhat"). Az `id: editingEvent ? editingEvent.id : Date.now()` azt jelenti: ha szerkesztünk, tartsuk meg a régi azonosítót (különben elveszítenénk, melyik elemet kell frissíteni); ha újat hozunk létre, generáljunk egyet a `Date.now()`-val (aktuális időbélyeg ezredmásodpercben – gyakorlatilag garantáltan egyedi ennél az alkalmazásnál).

**Miért `let updatedEvents = [...events]` és nem közvetlenül `events`-en dolgozunk?** Ugyanaz az elv, mint eddig mindenhol: nem mutáljuk közvetlenül a state-et, hanem másolatot készítünk, azt módosítjuk, és a végén egyetlen `setEvents`-szel írjuk vissza.

**A `.sort()` a dátum szerint rendezi újra a listát** minden hozzáadás/szerkesztés után, hogy az események mindig időrendben jelenjenek meg, függetlenül attól, milyen sorrendben vitted fel őket.

## `handleEditEvent` – új függvény

```jsx
const handleEditEvent = (event) => {
  setSelectedDate(new Date(event.date))
  setEventTime({
    hours: event.time.split(':')[0],
    minutes: event.time.split(':')[1],
  })
  setEventText(event.text)
  setEditingEvent(event)
  setShowEventPopup(true)
}
```
Bekötés a szerkesztés-ikonra:
```jsx
<i className="bx bxs-edit-alt" onClick={() => handleEditEvent(event)}></i>
```

**Miért `event.time.split(':')`?** Az esemény `time` mezője egy `"HH:MM"` formátumú string (pl. `"05:30"`). A `.split(':')` ezt egy két elemű tömbre bontja: `["05", "30"]` – az első elem az óra, a második a perc, pontosan úgy, ahogy az `eventTime` state-nek szüksége van rájuk.

## `handleDeleteEvent` – új függvény

```jsx
const handleDeleteEvent = (eventId) => {
  const updatedEvents = events.filter((event) => event.id !== eventId)

  setEvents(updatedEvents)
}
```
Bekötés a törlés-ikonra:
```jsx
<i className="bx bxs-message-alt-x" onClick={() => handleDeleteEvent(event.id)}></i>
```

**Miért `.filter()` és nem valamiféle "keresd meg és töröld" logika?** A `.filter()` egy új tömböt ad vissza, amiben csak azok az elemek maradnak, amikre a feltétel igaz – itt: "az azonosítója NEM egyezik a törlendővel". Ez egy tipikus, idiomatikus React/JS minta a "töröld ki ezt az elemet a listából" művelethez, mert nem mutálja az eredeti tömböt.

## A gomb szövege dinamikus legyen

```jsx
<button className="event-popup-btn" onClick={handleEventSubmit}>
  {editingEvent ? 'Update Event' : 'Add Event'}
</button>
```

## `handleTimeChange` – egységesítés (a 05-ös lecke két külön handlere helyett)

```jsx
const handleTimeChange = (e) => {
  const { name, value, min, max } = e.target
  const clamped = Math.min(Math.max(parseInt(value || '0', 10), Number(min)), Number(max))
  const normalized = String(clamped).padStart(2, '0')

  setEventTime((prevTime) => ({ ...prevTime, [name]: normalized }))
}
```
Cseréld le mindkét input `onChange`-ét erre:
```jsx
<input type="number" name="hours" min={0} max={24} className="hours" value={eventTime.hours} onFocus={(e) => e.target.select()} onChange={handleTimeChange} />
<input type="number" name="minutes" min={0} max={59} className="minutes" value={eventTime.minutes} onFocus={(e) => e.target.select()} onChange={handleTimeChange} />
```

**Miért jobb ez, mint a két külön inline handler?** A `name` attribútum (`"hours"` / `"minutes"`) és a JS **computed property name** szintaxis (`{ [name]: value }`) lehetővé teszi, hogy *ugyanaz* a függvény kezelje mindkét mezőt – a `name`-ből tudja, melyik property-t kell frissítenie, nem kell külön-külön leírni a logikát. Ezúttal a `padStart` már itt, minden billentyűleütésnél megtörténik (nem csak a submit-nál), így az input mező mindig két számjegyen mutatja az értéket.

## Utólagos javítások (a felhasználó által tesztelve, valós hibák)

**A napnevek egymásba érnek keskenyebb (laptop) képernyőn.** A 4. leckénél a transcriptet szó szerint követve teljes napneveket használtunk (`Sunday`, `Wednesday`...), nem a referencia rövidített verzióját – ez a CSS-t (`calc(100% / 7)` széles cellák) keskeny képernyőn túlfeszíti. Javítás:
```jsx
const daysOfWeek = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
```
(A CSS `text-transform: uppercase`-e miatt a megjelenítés változatlanul nagybetűs marad.)

**Üres esemény-szöveggel is menthető volt az esemény, és nem volt hozzá visszajelzés.** Hiányzó validáció + hiányzó UI-visszajelzés. Ahelyett, hogy csendben csak kilépnénk, jelenjen meg egy hibaüzenet a popupban, state-vezérelve (ugyanaz a minta, mint a többi UI-állapotnál eddig).

Új state:
```jsx
const [eventTextError, setEventTextError] = useState('')
```

`handleEventSubmit` eleje:
```jsx
const handleEventSubmit = () => {
  if (!eventText.trim()) {
    setEventTextError('Please enter event text.')
    return
  }

  setEventTextError('')

  const newEvent = {
    id: editingEvent ? editingEvent.id : Date.now(),
    date: selectedDate,
    time: `${eventTime.hours.padStart(2, '0')}:${eventTime.minutes.padStart(2, '0')}`,
    text: eventText,
  }
  // ... a többi rész változatlan
}
```

A `.trim()` azért kell, hogy csupa szóközből álló szöveg se menjen át érvényesként.

**Az `eventTextError`-t nullázni kell ott is, ahol eddig `setEventText('')`-t hívtunk** (`handleDayClick`, `handleEditEvent`), hogy a popup mindig hibaüzenet nélkül, "tiszta lappal" nyíljon meg egy új/másik eseményhez:
```jsx
setEventTextError('')
```

JSX – a textarea alá, és a textarea `onFocus`-ába egy törlés, hogy a mezőre kattintva (ugyanúgy, mint az óra/perc mezőknél a `select()`) eltűnjön a hiba:
```jsx
<textarea
  placeholder="Enter Event Text (Maximum 60 Characters)"
  maxLength="60"
  value={eventText}
  onFocus={(e) => setEventTextError('')}
  onChange={(e) => setEventText(e.target.value)}
></textarea>
{eventTextError && <div className="event-popup-error">{eventTextError}</div>}
```

CSS (`CalendarApp.css`, az `.event-popup` szabályai közelébe):
```css
.event-popup-error {
  color: #ff6b6b;
  font-size: 1.2rem;
}
```

## Elfogadási kritérium
- `npm run dev` hibátlanul fut, nincs konzol-hiba.
- Ceruza ikonra kattintva a popup megnyílik az esemény adataival kitöltve, "Update Event" felirattal.
- Update Event-re kattintva a meglévő esemény frissül (nem jön létre új sor).
- Törlés ikonra kattintva az esemény eltűnik a listából.
- Több esemény hozzáadása után a lista dátum szerint rendezett.
- Az óra/perc mezők gépeléskor mindig két számjegyen jelennek meg (pl. "5" → "05").
- Üres (vagy csak szóközből álló) esemény-szöveggel "Add Event"-re kattintva megjelenik a hibaüzenet, és az esemény NEM kerül be a listába; gépelés közben a hibaüzenet eltűnik.

## Nem tartozik ide
- Komponensek szétbontása (ez a backlogban van, külön feladatként)
