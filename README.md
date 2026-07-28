# Rabbitgotchi

A fully self-contained Tamagotchi-style virtual pet built as an R1 Creation for the Rabbit R1.

Single file: `index.html` (no build step, no backend, no external assets).

## How it plays

- Egg hatches after 5 minutes.
- Grows Baby → Child (1 day) → Teen (3 days) → Adult (5 days), all measured in real elapsed time — the clock keeps running even while the creation is closed, and catches up the stats the next time you open it (like a real Tamagotchi).
- Needs: Hunger, Happiness, Energy, Health. Neglect them and it gets sick or dies.
- **Lifespan**: decided once, at the day-5 adult checkpoint, from `mistakes - discipline` (both hidden counters — see "Naughty" below). Net ≤1 → evolves "good" and lives ~21 days total. Net 2–4 → evolves "good" and lives ~14 days (the real Tamagotchi's average). Net 5+ → evolves "bad" and lives only ~7 days. Severe acute neglect (sustained 0 hunger/happiness, or heavy uncleaned poop) can still kill it outright before that, same as before.
- Age is displayed in **years**, not days — fixed scale where 70 years = the 14-day normal lifespan (5 years per real day). A well-cared pet living the full 21 days shows ~105 at natural death; a neglected one dying at 7 days shows ~35.
- Poop spawns randomly and must be cleaned or it hurts Health.
- Actions: 🍔 Feed (visible Meal/Snack picker — see below), 🎮 Play (Higher/Lower minigame — see below), 🧹 Clean, 💊 Medicine (only helps if actually sick), 💤 Sleep toggle, ✋ Discipline (see "Naughty" below).
- **Feed**: opens a picker with 🍔 Meal and 🍪 Snack; scroll to move the highlight, press to confirm. Meal restores more hunger and adds more weight; Snack restores less hunger but gives a small happiness bump.
- **Play minigame**: best of 5 rounds. Each round shows a number 1–9; scroll to pick Higher or Lower than the next number, press to lock in the guess. Each round pops a "🎉 Correct!" celebration or "💢 Wrong!" irritation animation. After all 5 rounds, the overall record (not each round) drives the happiness change: win the match (3+/5) → +14 to +22 happiness depending on how decisively; lose the match → −14 to −22. Energy/weight cost is paid once per match, not per round.
- **Naughty**: scheduled (not a random low-frequency roll) to happen reliably about **once a day** — the pet refuses to eat until disciplined; Feed just bounces off with "It refuses to eat!". ✋ Discipline opens a confirm screen (not an instant action) showing whether it's actually naughty right now and a fillable **discipline bar** (capped at 5, roughly one fillable point per day of the 5-day growth window, shown until adulthood) — scroll to pick "✋ Discipline" or "↩ Back", press to confirm. Disciplining while naughty costs a little happiness (−3) and fills the bar. At the day-5 checkpoint, `mistakes - discipline` decides the evolution/lifespan tier (see above), so consistently catching naughty episodes can meaningfully offset earlier neglect, and not disciplining enough leaves the net mistake count high → a lesser Tamagotchi. Disciplining when it's *not* naughty is a real mistake: **−15 happiness**, no discipline-bar gain — the confirm screen shows you the naughty status first specifically so this is an informed choice, not a misclick.
- Death shows a tombstone with an option to start a new egg.

## Controls (real R1 hardware events used)

- Scroll wheel: `scrollUp` / `scrollDown` move the menu selection, or the highlighted choice in the Feed picker / minigame.
- Side (PTT) button short press: `sideClick` activates the selected action or confirms the current choice.
- Side button held: `longPressStart`/`longPressEnd` also confirms (same as a short press) — no longer has a distinct meaning of its own.
- Tap the pet directly: small happiness nudge (petting).
- Also works with mouse/touch clicks on the menu icons, and arrow keys + Enter, for testing in a normal desktop browser.

## Storage

Writes to both `window.creationStorage.plain` (the documented R1 Creations persistence API) and `localStorage` on every save, and reads back whichever has data.

**Confirmed on real R1 hardware (2026-07-26): `window.creationStorage` does not exist on this device's firmware at all** (`CS:n` in the on-screen diagnostic). `localStorage` is what's actually keeping saves alive, and has been confirmed to survive a full exit/reopen cycle on-device. Keep the dual-write in place regardless — costs nothing and covers other firmware versions where `creationStorage` might exist.

A small always-on diagnostic reads in the bottom-right corner: `<BUILD_ID> · CS:y/n · save:ok/FAIL`. Useful for any future on-device debugging since there's no console access.

## Testing locally on this PC

Just open `index.html` directly in a browser, or serve the folder with any static server. The window is fixed at 240×282 to match the R1 screen.

## Deployed

- Live at: **https://miente1.github.io/rabbitgotchi/** (GitHub Pages, public repo `Miente1/rabbitgotchi`, served from `main` branch root)
- Icon: `icon.png`, also hosted alongside `index.html`
- Install QR code: `install-qr.png` in this folder — scan it from the R1's Creations card to install.

**QR code format**: the R1 Creations scanner does NOT accept a bare URL — it expects the QR to encode a JSON object: `{"title", "url", "description", "iconUrl", "themeColor"}` (confirmed from the official `qr` tool in `rabbit-hmi-oss/creations-sdk`). A plain-URL QR fails with "not a valid creation" on the device. Regenerate with the same schema if the URL/icon ever changes.

To redeploy after editing `index.html`: commit and push (Pages rebuilds automatically), but that alone is **not enough to get it onto the device** — see below.

**Updating the already-installed device copy**: confirmed on real hardware that the R1 only fetches a creation's content at install/QR-scan time — reopening from its list does NOT refetch. After every code change: bump `BUILD_ID` in `index.html`, bump the `?v=` query string in the QR payload, regenerate `install-qr.png`, push, and rescan the new QR on-device. Reusing the same URL/QR is not enough.

## Notes / limitations

- I verified the game logic by careful manual trace (state machine, decay math, evolution thresholds, storage fallbacks) rather than a live screenshot — the sandboxed browser tool available to me refuses to load `localhost`/local servers, and I don't have physical R1 hardware to test the real `scrollUp`/`sideClick`/`creationStorage` calls against. Everything is built strictly against Rabbit's official `rabbit-hmi-oss/creations-sdk` reference (screen size, event names, storage API), but you should do a real on-device smoke test before relying on it.
