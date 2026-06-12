# ✅ Handover Brief — Training Log Web App (Stefano Capacci)

## 1. Context

* Single-page HTML app (`index.html`)
* No framework, vanilla JS
* Hosted on GitHub Pages
* Uses:
  * Firebase (Firestore + Auth)
  * Strava API (OAuth + activity fetch)

Goal:  
👉 Maintain a **stable, structured, production-safe training log system**  
👉 Improve UX, reliability, and maintainability  
👉 Avoid introducing syntax or regression issues

---

# ✅ 2. Backend / Sync Changes

## ✅ Firestore rules

Use this configuration:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /traininglog/{document} {
      allow read, write: if request.auth != null;
    }
    match /secrets/strava {
      allow read: if request.auth != null;
      allow write: if false;
    }
  }
}
```

👉 Requirement:

* Authenticated user only
* All quarters stored under `traininglog/*`

---

## ✅ Local storage versioning

Replace hardcoded key:

```js
"tri_v21"
```

with:

```js
const LS_STATE_KEY = "tri_v26";
const LS_UI_KEY = "tri_v26_ui";
```

👉 Use:

```js
localStorage.getItem(LS_STATE_KEY)
localStorage.setItem(LS_STATE_KEY, JSON.stringify(state))
```

---

## ⚠️ Critical constraint

👉 Do NOT break JSON / JS syntax  
👉 Avoid regex-based transformations of code

---

# ✅ 3. Structural UI Changes (Plan Locking)

## ❌ Remove completely

* "Remove week" button
* "+ Add a week" button
* "Reset to original plan" button

## ✅ Keep

* All editing within weeks:
  * sessions
  * checkboxes
  * comments
  * habits

👉 Weeks must be **immutable structure**

---

# ✅ 4. Dashboard Changes

## ✅ Rolling month table

Replace:

```
last 8 weeks
```

with:

```
last 4 weeks (rolling)
```

Implementation:

```js
for (let i = 3; i >= 0; i--)
```

---

# ✅ 5. Quarter Logic (core redesign)

## ✅ Problem to solve

Original code:

* Based on calendar quarters
* Uses hardcoded `type: live/future/archive`

## ✅ Required behaviour

Quarter status must be **dynamic** based on actual training calendar:

* Quarter becomes **live** when its first week starts
* Previous quarter becomes **archive**
* Future quarters remain **future**

## ✅ Example

* Q2 2026 remains live until week of **06 Jul 2026**
* Q3 2026 becomes live starting that week

---

## ✅ Required functions

```js
function currentQtrId()
function qtrStatus(qid)
```

👉 Must drive:

* dropdown labels
* dashboard behaviour
* rendering logic

---

# ✅ 6. Selected Quarter Context

## ✅ Requirement

UI must behave based on **selected quarter**, not only current date.

## ✅ Implementation logic

Create:

```js
function selectedQuarterWeekIndex(weeks)
```

Rules:

* If today inside selected quarter → return matching week
* If selected quarter is future → return first week
* If archive → return last week

---

## ✅ Apply to

* Timeline highlight
* Week card highlight

---

# ✅ 7. UX Improvements

## ✅ 7.1 Collapsible Performance Dashboard

Wrap entire dashboard section:

* trend cards
* table
* completion ring
* timeline

Add toggle:

```
Performance dashboard ▼
```

## ✅ Requirements

* Collapsible
* State persisted in localStorage
* No impact on functionality

---

## ✅ 7.2 Timeline usability

Enhance timeline:

### ✅ Must:

* Highlight active week
* Be clickable
* Scroll to corresponding week
* Expand collapsed week

---

## ✅ Visual indicators

```css
.seg.current
.week.current-week
```

---

## ❌ Do NOT implement

* Activity limiting (Strava list remains full)

---

# ✅ 8. Code Quality Improvements

## ✅ Centralise sport logic

Replace repeated checks with:

```js
const RUN_SPORTS = [...]
const BIKE_SPORTS = [...]
const SWIM_SPORTS = [...]

function isRunSport(s) {...}
function isBikeSport(s) {...}
function isSwimSport(s) {...}
```

---

## ✅ Strava error handling

Must NOT fail silently.

Add:

```js
syncStatus("Strava sync failed", true)
```

Also for:

* auth failure
* refresh failure

---

# ✅ 9. Known Bug Class (CRITICAL — avoid)

## ❌ Do NOT break syntax like:

```js
ifisRunSport(...)
```

## ✅ Always ensure:

```js
if (isRunSport(...))
```

---

## ❌ Do NOT introduce HTML entities into JS

Replace:

```js
&gt;   →  >
&amp;&amp; → &&
```

---

# ✅ 10. Constraints for next agent

### ✅ MUST:

* Apply changes manually or structurally
* Preserve JS validity at all times
* Keep single-file architecture

---

### ❌ MUST NOT:

* Use regex-based blind replacements
* Modify unrelated logic
* Break Firebase or auth flow

---

# ✅ 11. Expected Final Behaviour

After implementation:

### ✅ System should:

* Sync via Firebase correctly
* Allow Google sign-in
* Show rolling last 4 weeks
* Switch live quarter dynamically
* Lock week structure
* Allow editing inside weeks

---

### ✅ UX:

* Clean, no destructive actions
* Collapsible dashboard
* Clickable timeline navigation
* Clear Strava error feedback

---

# ✅ 12. Output expectation

Agent should produce:

* ✅ Updated `index.html`
* ✅ No JS syntax errors
* ✅ No regression in existing features

---

# ✅ 13. Testing Checklist

### ✅ Critical paths to test

- [ ] Sign in with Google
- [ ] Firebase data loads correctly
- [ ] Strava connect button appears when not authenticated
- [ ] Strava activities sync and populate week cards
- [ ] Quarter dropdown works and switches views
- [ ] Week data persists across page reloads (Firebase + localStorage)
- [ ] All editable fields save correctly:
  - [ ] Session checkboxes
  - [ ] Session text
  - [ ] Comments
  - [ ] Habits (if Q2+ 2026)
- [ ] Timeline renders correctly
- [ ] Performance dashboard (trends, table, ring) updates accurately
- [ ] No console errors or syntax issues

---

# ✅ 14. Browser compatibility

* Modern browsers (Chrome, Safari, Firefox)
* ES6 syntax (template literals, arrow functions, etc.)
* No polyfills required

---

# ✅ 15. Deployment notes

* Automatic via GitHub Pages
* Requires Firebase credentials in code (already present)
* Strava client secret stored in Firebase (do NOT commit)

---

# ✅ Bottom line

This is a **refactoring + UX stabilisation task**, not a rewrite.

Key priorities:

1. **Correctness (no JS errors)**
2. **Deterministic behaviour (quarters, weeks)**
3. **Safe UI (no destructive controls)**
4. **Data integrity (Firebase sync works)**

---

**Last updated:** 2026-06-12  
**File:** `index.html` (v22+)  
**Repo:** `stecapacci/Training-Log`
