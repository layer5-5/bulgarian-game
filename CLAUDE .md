# Само в България — Party Game App

A TV-displayed party game about the Bulgarian 2026 election chaos, built to be hosted from a laptop on a big screen with 6 players in the room.

---

## The Goal

A host runs the game from their laptop connected to a TV. 6 players, 3 teams of 2. The game walks through 4 rounds sequentially. The UI must be readable from 3 meters away — big text, high contrast, crystal clear at all times. The host clicks to advance. No player needs a device. The experience should feel like a premium game show interface: sharp, dramatic, fun. When someone reads a party name out loud and the room erupts — that's the win condition.

---

## Tech Stack

- **Framework:** Single HTML file. No build step, no server, no npm. One file the host double-clicks and opens in Chrome fullscreen (F11). This is the only acceptable format — it must be completely self-contained.
- **Styling:** Vanilla CSS with CSS variables. No frameworks. Custom design only.
- **Fonts:** Google Fonts loaded via CDN link tag — use Playfair Display (display/headers) + IBM Plex Mono (labels/scores/UI chrome)
- **State:** Vanilla JS, all in-memory. No localStorage. No external dependencies beyond fonts.
- **Data:** All game content (questions, statements, party data) lives in a single `const GAME_DATA` object at the top of the script section — clearly separated, easy to edit.

---

## Architecture

Single HTML file, structured as:

```
index.html
├── <style>          — all CSS, CSS variables, responsive rules
├── <body>           — screen divs, one per game state
└── <script>
    ├── GAME_DATA    — all content, easily editable
    ├── STATE        — current game state object
    └── functions    — render, navigate, score, reveal
```

**Screen model:** Only one screen visible at a time. Screens: `intro`, `round1`, `round2`, `round3`, `round4`, `scoreboard`. Navigation is always a large "Continue →" button at the bottom right. No tabs, no nav bar — the host just presses forward.

**GAME_DATA structure:**
```js
const GAME_DATA = {
  round1: { items: [ { name: "...", real: true, explanation: "..." }, ... ] },
  round2: { cards: [ { politician: "...", party: "...", statements: [ { text: "...", isLie: false }, ... ], lieExplanation: "..." }, ... ] },
  round3: { parties: [ { abbrev: "...", fullName: "...", leader: "...", polls: "...", color: "...", summary: "...", researchAngle: "..." }, ... ] },
  round4: { parties: [ { id: "...", abbrev: "...", seats: 0, vetoes: [], description: "..." }, ... ] }
}
```

Adding new questions = editing GAME_DATA only. Nothing else changes.

---

## Data Model

### Round 1 — Real Party or Fever Dream?
```
QuizItem {
  name: string           // party name to display
  real: boolean          // true = real Bulgarian party
  explanation: string    // shown after reveal — 1-2 sentences, witty
}
```
Start with 12 items. Mix: 8 real, 4 fake.

### Round 2 — Two Truths One Lie
```
TruthCard {
  politician: string
  party: string
  statements: [ { text: string, isLie: boolean } ]  // always 3, always 1 lie
  lieExplanation: string   // shown after reveal
}
```
Start with 4 cards (Borisov, Peevski, Radev, Kostadinov).

### Round 3 — Party Research Sprint
```
PartyCard {
  abbrev: string
  fullName: string
  leader: string
  polls: string          // e.g. "~20%"
  color: string          // hex, used for color bar
  summary: string        // 2-3 sentences
  researchAngle: string  // what to google — shown on card during research phase
}
```
3 parties: GERB, PP–DB, Vazrazhdane.

### Round 4 — Coalition Crisis
```
CoalitionParty {
  id: string
  abbrev: string
  seats: number
  vetoes: string[]       // array of party ids they refuse to work with
  description: string    // short veto explanation shown on card
}
```
7 parties available, threshold = 121 seats.

### Score
```
Team { name: string, scores: { r1: 0, r2: 0, r3: 0, r4: 0, bonus: 0 } }
```
3 teams.

---

## Features to Build

**1. Intro / Title Screen**
- Full-screen dramatic title: "САМО В БЪЛГАРИЯ" in huge serif font
- Subtitle: "Only in Bulgaria — A Political Party Game"
- Metadata bar: 6 players · 3 teams · 4 rounds
- Brief one-paragraph "The Story" section — 9 elections since 2021, gold bars, Magnitsky oligarch, president who quit to run for parliament without a manifesto
- Large "Start Game →" button

**2. Round 1 — Real Party or Fever Dream?**
- Round intro screen first: title, instructions. Big "Begin Round →" button.
- Then the quiz screen: display all 12 party name cards in a responsive grid (3×4 or 4×3)
- Each card: large party name, small number label (01–12), neutral state
- Host clicks a card to reveal: card transitions to show green (real) or red (fake) + explanation text
- "Reveal All" button + "Reset" button at bottom
- Score entry: after reveal, host taps "+1" per team for each correct answer — running total shown
- Teams write answers on paper; host reveals and marks

**3. Round 2 — Two Truths One Lie**
- Round intro screen with rules
- Show one politician card at a time (host navigates forward/back between cards)
- Card shows: politician name + party tag + 3 lettered statements (A, B, C)
- All 3 statements visible at once — NO pre-reveal hiding
- Host clicks the lie statement to reveal it: it turns red with "← THE LIE" label, truths turn green
- Lie explanation text appears below after reveal
- Score entry per team per card (3 pts for correct)
- "Next →" button advances to next politician

**4. Round 3 — Party Research Sprint**

This round has 3 distinct phases. Build them as separate sub-screens within Round 3. NO timers anywhere in this round.

**Phase A — Research**
- Show all 3 party cards simultaneously in a row, large, readable from across the room
- Each card shows: color bar, party abbrev (huge), full name, leader, poll number, 2-sentence summary, and the research angle prompt ("🔍 What to Google: ...")
- No timer. Host controls the pace entirely.
- Large "Time's Up → Start Pitches" button at the bottom — host clicks when ready
- Teams use their phones to research. The screen is their reference.

**Phase B — Pitch & Pros/Cons Presentation (one team at a time)**
- Show one party card at a time, full screen, enlarged
- Below the party info: a PITCH TEMPLATE panel displayed as large empty labeled rows — visual guide for the room while the team reads from their paper notes:
  ```
  PARTY: [name]
  LEADER: [name]
  IN ONE SENTENCE: _______________
  ✅ PRO 1: _______________
  ✅ PRO 2: _______________
  ❌ CON 1: _______________
  ❌ CON 2: _______________
  😂 WILD CARD FACT: _______________
  ```
- No timer. Host clicks "Next Team →" when the team is done.
- After all 3 teams have pitched: "Go to Vote →" button

**Phase C — Vote**
- Full-screen: "Which team gave the best pitch?"
- Three large buttons, one per team: "Team 1", "Team 2", "Team 3"
- Host clicks the winner — 10 pts to winner, 5 pts to each other team, awarded automatically
- Bonus prompt shown below: "Did anyone find a genuinely shocking fact? Award +2 pts via Scoreboard"
- "Continue to Round 4 →" button

**5. Round 4 — Coalition Crisis**
- Round intro with rules: need 121 seats, vetoes apply, cooperative, no time limit
- Show all 7 party cards in a grid. Each card: party abbrev, seats, veto note
- Host clicks cards to toggle them into the coalition (selected = highlighted)
- Live seat counter: large number, updates in real time. Color: red (<100), amber (100–120), green (121+)
- Veto conflict detection: if two vetoing parties are both selected, show a red warning banner "⚠ VETO CONFLICT" and identify which pair
- Victory state: if 121+ seats and no conflicts → big green banner "GOVERNMENT FORMED 🇧🇬", confetti animation, 25 pts awarded to all teams
- Give up state: a small "We give up" button — triggers dark screen, "Caretaker Government Appointed. Election #10 Scheduled. See you in 6 months." Then a "...try again?" link to reset

**6. Scoreboard**
- Accessible at any time via a persistent small "Scores" button bottom-left (doesn't interrupt game flow)
- Overlay/modal style — opens on top of current screen
- Shows all 3 teams, all round scores, running totals
- Host can edit any score by clicking it (inline number input)
- Bonus points row: "+5 if coalition matches April 19 result"
- Close button returns to game

---

## UI & Design Direction

**Aesthetic:** Newspaper meets game show. Think The Economist crossed with Who Wants to Be a Millionaire. Dark, serious chrome with moments of color drama. NOT playful/rounded/cartoonish — the comedy comes from the content, not the design.

**Color palette:**
```
--bg: #0D0D0D (near black, main background for game screens)
--surface: #1A1A1A (card backgrounds)
--paper: #F5F0E8 (warm white, used for intro/content screens)
--red: #C0392B
--green: #1A6B3C
--gold: #C9973A
--white: #FFFFFF
--grey: #888888
```

**Typography:**
- Headlines/round titles: Playfair Display, 900 weight, very large (80–120px on TV)
- Body/labels/scores: IBM Plex Mono, uppercase tracking
- Content text: IBM Plex Mono regular

**Layout:** Everything centered, max-width 1200px, designed for 1920×1080. Generous padding. Nothing cramped.

**Cards:** Flat rectangles, 2px solid borders, sharp corners. No border-radius. No shadows. Color bar (4px) at top of each card indicating party color.

**Transitions:** CSS transitions only. Screen changes: 300ms fade. Card reveals: 200ms background-color transition.

**Buttons:** Large (min 48px height), full-width or fixed-width blocks, monospace label, uppercase, 2px border. Primary = white text on black. Danger = white on red. Success = white on green.

**TV-readiness rules:**
- Minimum body font size: 18px
- Card names/titles: minimum 28px
- Round titles: 72px+
- Seat counter in coalition round: 96px
- Never use font-weight below 400
- All interactive elements have clear hover states with color fill

---

## Content to Include

### Round 1 — Party Names (12 items, 8 real, 4 fake)

Real parties:
1. "There Is Such a People" — real, ITN, named after a Slavi Trifonov quote. He won an election then refused to form a government.
2. "Morality, Unity, Honour" — real, MECh, acronym means "sword" in Bulgarian.
3. "Movement of Non-Party Candidates" — real, a party specifically for non-party people. A party.
4. "Out of the EU and NATO" — real, registered for 2026. Exactly what it sounds like.
5. "People's Party — The Truth and Only the Truth" — real, apparently "The Truth" alone wasn't emphatic enough.
6. "Unrepentant Bulgaria" — real, "Непокорна България," nationalist movement.
7. "We Continue the Change" — real, PP, liberal reform party, briefly governed for 7 months.
8. "My Bulgaria" — real, "Моя България," 2026 coalition.

Fake parties:
9. "Movement of Honest Taxpayers" — fake, plausibly Bulgarian though.
10. "People's Alliance for Reasonable Decisions" — fake, reasonable decisions not available in this country.
11. "Democratic Forces for a New Beginning (Again)" — fake, though "New Beginning" IS a real party suffix.
12. "The Party That Will Finally Fix Things" — fake, but also the implicit campaign promise of every party above.

### Round 2 — Two Truths One Lie (4 cards)

**Boyko Borisov / GERB**
- A (LIE): Before politics, Borisov was a bodyguard for communist-era leader Todor Zhivkov and later Tsar Simeon II. → LIE: he was bodyguard for Sofia's mayor and Simeon, NOT Zhivkov.
- B (TRUTH): In 2020, photos leaked showing a man resembling Borisov asleep next to a handgun, stacks of cash, and what appeared to be gold bars on his nightstand.
- C (TRUTH): Despite losing power in 2021 amid mass corruption protests, GERB has remained the largest single party in parliament in every election since.

**Delyan Peevski / MRF–New Beginning**
- A (TRUTH): Peevski was placed on the US Magnitsky Act sanctions list in 2023 for "significant corruption," freezing US assets and banning Americans from doing business with him.
- B (LIE): In 2013, Peevski was briefly appointed head of national security DANS — triggering Bulgaria's largest protests since 1997, forcing his resignation within 24 hours. → LIE: it was two days, not 24 hours.
- C (TRUTH): Despite being Magnitsky-sanctioned, Peevski ran in 2024 as party leader, won a parliamentary seat, and continues to serve as an MP.

**Rumen Radev / Progressive Bulgaria**
- A (TRUTH): Radev resigned from the presidency approximately 8 months before his term ended, then immediately announced a new party and began leading in election polls.
- B (TRUTH): His party Progressive Bulgaria reached the top of polls with no published manifesto, no formal policy platform, and only two interviews given since leaving office.
- C (LIE): Before becoming president, Radev was Bulgaria's Minister of Defence and used that role to launch his presidential campaign. → LIE: he was Commander of the Air Force, not Minister of Defence.

**Kostadin Kostadinov / Vazrazhdane**
- A (TRUTH): Kostadinov's party openly opposes Bulgaria's NATO membership, calls for Russia sanctions to be lifted, and demands Bulgaria stop supporting Ukraine.
- B (LIE): Vazrazhdane's rise was partly fuelled by being the first Bulgarian party to make COVID vaccine passport opposition a central campaign issue. → LIE: several populist parties got there before them.
- C (TRUTH): Revival entered parliament for the first time in 2022 after mobilising anti-vaccine-mandate voters, and has grown its seat count in every election since.

### Round 3 — Party Research Cards (3 parties)

**GERB** | Citizens for European Development of Bulgaria | Leader: Boyko Borisov | ~20% polls | Color: #003087
Summary: Centre-right, pro-EU, pro-NATO. Bulgaria's dominant party since 2009. Has governed more than any other party. Currently polling second behind Radev's new formation.
Research angle: How many times has Borisov been PM? What exactly was in the 2020 photos? What is "the Eight Dwarfs scandal"?

**PP–DB** | We Continue the Change – Democratic Bulgaria | Leaders: Vasilev, Mirchev, Bozhanov, Atanasov | ~12% polls | Color: #009A44
Summary: Liberal, pro-EU, anti-corruption. Actually governed briefly — their cabinet lasted 7 months before a no-confidence vote. Currently has four co-leaders simultaneously.
Research angle: Who is Kiril Petkov? What did their government actually achieve? Why do they have four leaders?

**Vazrazhdane** | Revival | Leader: Kostadin Kostadinov | ~7% polls | Color: #CC0000
Summary: Far-right, pro-Russian, anti-NATO, anti-EU. Wants Bulgaria out of NATO, Russia sanctions lifted, Ukraine support stopped. Grew from nothing to parliament in 2022 and keeps growing.
Research angle: What percentage of Bulgarians have pro-Russian sympathies? What are their specific NATO positions? Any documented Russian connections?

### Round 4 — Coalition Parties (7 clickable blocks)

| ID | Abbrev | Seats | Vetoes | Note |
|----|--------|-------|--------|------|
| PB | Progressive BG | 74 | GERB, MRFNN | Radev's new party, leading in polls |
| GERB | GERB–SDS | 49 | MRFNN | Borisov's party, refuses Peevski (publicly) |
| PPDB | PP–DB | 28 | MRFNN, REVIVAL | Reform bloc, blocks Peevski and far-right |
| MRFNN | MRF–New Beginning | 24 | — | Peevski's party. No formal vetoes. Everyone avoids them. |
| REVIVAL | Vazrazhdane | 16 | PPDB | Pro-Russian, refuses reformists |
| BSP | BSP–United Left | 9 | MRFNN | Socialists, refuse Peevski |
| MECH | MECh | 8 | PPDB | Nationalist populists, refuse PP–DB |

Threshold: 121 seats. Total available: 208. The puzzle is solvable — barely.

---

## What Makes This Great

1. **TV readability is non-negotiable.** Every font size, contrast ratio, and layout decision must pass the "readable from 3 meters" test. When in doubt, go bigger.

2. **No timers anywhere.** The host controls the pace. Every phase ends when the host clicks a button, not when a clock runs out.

3. **Content is king, UI gets out of the way.** Party names and politician statements need space to breathe and land. No clutter around the content that matters.

4. **GAME_DATA is a first-class citizen.** Comment it clearly: `// ADD MORE ROUND 1 ITEMS HERE`, `// ADD MORE TRUTH CARDS HERE`. Make it trivially obvious where to paste new content. The host should be able to add 5 new quiz items in 2 minutes without touching any other code.

5. **The coalition give-up state should be genuinely funny.** Black screen, red text, "Caretaker Government Appointed. Election #10 Scheduled. See you in 6 months." Pause. Then a small "...try again?" link.

---

## Do NOT

- Do not use any JavaScript framework (React, Vue, Svelte). One HTML file, vanilla only.
- Do not use localStorage or any persistence. State lives only in memory.
- Do not add a navigation bar or tabs. The game flows forward. Back-navigation is only within rounds (e.g., previous/next politician in Round 2).
- Do not make cards small. This is a TV. Everything must be legible at distance.
- Do not use border-radius > 4px anywhere. Sharp, editorial aesthetic.
- Do not add background music or sound.
- Do not split into multiple files. One file, always.
- Do not add a login, user accounts, or any backend.
- Do not put score editing behind a modal that's hard to access — the host needs to update scores fast between rounds without losing their place.
- **Do not add any timers or countdowns anywhere in the app.** The host controls all pacing manually.
