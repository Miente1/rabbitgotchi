# Rabbitgotchi

A fully self-contained Tamagotchi-style virtual pet built as an R1 Creation for the Rabbit R1.

Single file: `index.html` (no build step, no backend, no external assets).

## How it plays

- Egg hatches after 5 minutes.
- Grows Baby → Child (1 day) → Teen (3 days) → Adult (5 days), all measured in real elapsed time — the clock keeps running even while the creation is closed, and catches up the stats the next time you open it (like a real Tamagotchi).
- Needs: Hunger, Happiness, Energy, Health. Neglect them and it gets sick or dies.
- **Lifespan**: decided once, at the day-5 adult checkpoint, from the same hidden mistake counter that decides the good/bad evolution form. Great care (≤1 mistake) → evolves "good" and lives ~21 days total. Normal care (2–4 mistakes) → evolves "good" and lives ~14 days (the real Tamagotchi's average). Neglectful-but-not-fatal care (5+ mistakes) → evolves "bad" and lives only ~7 days. Severe acute neglect (sustained 0 hunger/happiness, or heavy uncleaned poop) can still kill it outright before that, same as before.
- Age is displayed in **years**, not days — fixed scale where 70 years = the 14-day normal lifespan (5 years per real day). A well-cared pet living the full 21 days shows ~105 at natural death; a neglected one dying at 7 days shows ~35.
- Poop spawns randomly and must be cleaned or it hurts Health.
- Actions: 🍔 Feed (tap = meal, hold PTT = snack), 🎮 Play, 🧹 Clean, 💊 Medicine (only helps if actually sick), 💤 Sleep toggle, ✋ Scold.
- Death shows a tombstone with an option to start a new egg.

## Controls (real R1 hardware events used)

- Scroll wheel: `scrollUp` / `scrollDown` move the menu selection.
- Side (PTT) button short press: `sideClick` activates the selected action.
- Side button held: `longPressStart`/`longPressEnd` — used for the Feed icon to give a snack instead of a meal.
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
- The "Play" action is a simplified stat boost rather than the original toy's number-guessing minigame, to keep this a single evening's build rather than a whole subsystem.
