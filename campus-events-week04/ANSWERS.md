# Week 03 Answers

### 1. In `EventList`, what goes wrong if you use the array index as `:key` and the list gets filtered while a card has local state (for example an open details panel)?
Vue uses `:key` to track each component instance across renders and employs an in-place patch strategy when keys are array indices (`0, 1, 2...`). When the list is filtered, a different event shifts into index `0`, so Vue reuses the existing component instance at index `0` instead of destroying and recreating it. As a result, the local state (like the open details panel) stays attached to the wrong event card.

### 2. Why is `visible` a `computed` and not a function called from the template? What is the difference when you type in the search box?
A `computed` property caches its result based on its reactive dependencies (`events`, `search`, and `category`) and only recalculates when one of them changes, whereas a function is re-executed on every component render and every place it is referenced in the template. In `EventList`, `visible` is used three times in the template (`visible.length` for the count, `v-if="visible.length > 0"`, and `v-for="event in visible"`). When you type a character in the search box, a `computed` filters the array only once per keystroke and reuses the cached result, whereas a function would run the filter loop three separate times per keystroke (and again on any unrelated re-render).

### 3. `EventCard` emits `book` instead of changing `event.booked`. Give two reasons.
First, Vue enforces **one-way data flow** (props down, events up): child components should not mutate props owned by a parent because it makes state changes unpredictable and hard to debug. Second, booking an event requires updating state outside of `EventCard` (`myBookings` and `localStorage` in `App.vue`), so the parent component that owns the full state must handle the business logic.

### 4. You write `const { booked } = reactive(event); booked++`. The page does not change. Why?
Destructuring a primitive property (`number`) from a `reactive` object copies its plain value into a normal local variable and disconnects the reactive proxy tracking. Incrementing `booked++` only modifies that disconnected local number in memory rather than mutating `event.booked` through the proxy, so Vue never detects the change or triggers a re-render.

### 5. Where does `myBookings` live, and why not in `EventCard` or `EventList`?
`myBookings` lives in `App.vue`, which is the lowest common ancestor of all components that need access to the booking data. It cannot live in `EventCard` because each card only knows about a single event, and it should not live in `EventList` because the `<header>` in `App.vue` needs `myBookings` to display the total `"You have booked N seats."` count. Lifting the state up to `App.vue` keeps a single source of truth shared between the header and the event list.