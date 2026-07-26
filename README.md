# Rabbitgotchi

A fully self-contained Tamagotchi-style virtual pet built as an R1 Creation for the Rabbit R1.

Single file: `index.html` (no build step, no backend, no external assets).

## How it plays

- Egg hatches after 5 minutes.
- Grows Baby → Child (1 day) → Teen (3 days) → Adult (5 days), all measured in real elapsed time — the clock keeps running even while the creation is closed, and catches up the stats the next time you open it (like a real Tamagotchi).
- Needs: Hunger, Happiness, Energy, Health. Neglect them and it gets sick or dies; care well and it evolves into a "good" adult form at day 5, neglect it and it evolves "bad".
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

Uses `window.creationStorage.plain` (the real R1 Creations persistence API) when available, and falls back to `localStorage` automatically when running in a normal browser — so you can develop and test on your PC before deploying.

## Testing locally on this PC

Just open `index.html` directly in a browser, or serve the folder with any static server. The window is fixed at 240×282 to match the R1 screen.

## Deployed

- Live at: **https://miente1.github.io/rabbitgotchi/** (GitHub Pages, public repo `Miente1/rabbitgotchi`, served from `main` branch root)
- Icon: `icon.png`, also hosted alongside `index.html`
- Install QR code: `install-qr.png` in this folder — scan it from the R1's Creations card to install.

**QR code format**: the R1 Creations scanner does NOT accept a bare URL — it expects the QR to encode a JSON object: `{"title", "url", "description", "iconUrl", "themeColor"}` (confirmed from the official `qr` tool in `rabbit-hmi-oss/creations-sdk`). A plain-URL QR fails with "not a valid creation" on the device. Regenerate with the same schema if the URL/icon ever changes.

To redeploy after editing `index.html`, just commit and push — Pages rebuilds automatically:
```
git add index.html && git commit -m "update" && git push
```

## Notes / limitations

- I verified the game logic by careful manual trace (state machine, decay math, evolution thresholds, storage fallbacks) rather than a live screenshot — the sandboxed browser tool available to me refuses to load `localhost`/local servers, and I don't have physical R1 hardware to test the real `scrollUp`/`sideClick`/`creationStorage` calls against. Everything is built strictly against Rabbit's official `rabbit-hmi-oss/creations-sdk` reference (screen size, event names, storage API), but you should do a real on-device smoke test before relying on it.
- The "Play" action is a simplified stat boost rather than the original toy's number-guessing minigame, to keep this a single evening's build rather than a whole subsystem.
