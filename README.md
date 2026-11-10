# Rokumon Cats

A self-contained drop-and-merge game for the Rokumon Games catalogue. Open `index.html` in a browser. No build step, no network, no external images or fonts.

Two of the same cat touch, they become the next larger cat. Keep the stack under the limit line.

## Play

- **Free play** — a fresh drop order every run. Three undos.
- **Daily challenge** — every player gets the same drop order for the UTC date. One undo. The order changes at UTC midnight; a run already in progress keeps the seed it started with.

Local best score and highest cat are stored in `localStorage` under `rokumon-cats.v1`.

## Rules

1. A cat waits above the bowl. Aim, then drop it.
2. Cats of the same tier merge on contact into the next tier, at their shared center.
3. The new cat can merge again. Merges within 1.2 seconds raise a chain. Chain ×2 doubles that merge's points, ×3 triples them, and so on.
4. The largest cat, Rokumon, does not merge.
5. If any cat that has been in the bowl for a moment rests above the limit line for about a second, the run ends.
6. Undo puts the last drop back, including score and the upcoming cats. It cannot be used once you are out of tokens.

Leaving from the pause menu discards the run and does **not** submit a leaderboard result. Only an overflow submits `gameOver`.

## Controls

| Input | Action |
| --- | --- |
| Mouse move | Aim |
| Click or release | Drop |
| Touch drag, then release | Aim and drop |
| Left / A | Aim left |
| Right / D | Aim right |
| Down, Space, or Enter | Drop |
| U | Undo |
| M | Mute |
| Esc | Pause, or close the open panel |

## Scoring

`score = merge points + crest bonus`

Merge points are awarded for the cat **created** by a merge, then multiplied by the current chain.

| New tier | Name | Merge points | Crest bonus (once, for reaching this tier) |
| --- | --- | --- | --- |
| 1 | Mote | — (only dropped) | 0 |
| 2 | Kit | 2 | 10 |
| 3 | Pip | 4 | 30 |
| 4 | Miso | 8 | 60 |
| 5 | Nori | 16 | 100 |
| 6 | Mochi | 30 | 160 |
| 7 | Sora | 52 | 240 |
| 8 | Hana | 84 | 360 |
| 9 | Kumo | 130 | 520 |
| 10 | Yuki | 200 | 760 |
| 11 | Rokumon | 320 | 1100 |

Crest uses the highest tier reached, including a cat that was dropped rather than merged. It is not multiplied by the chain. The number reported to the hub is the integer total.

Dropped cats are weighted toward the small end: Mote 46, Kit 28, Pip 16, Miso 7, Nori 3. Daily runs use that same table with a fixed seed.

## Tier list

Sizes are the physics radius in bowl units. The bowl's inner width is 352.

| Tier | Id | Name | Radius | Reads as |
| --- | --- | --- | --- | --- |
| 1 | `mote` | Mote | 16 | Tiny ivory kitten, sleepy eyes |
| 2 | `kit` | Kit | 22 | Peach kitten |
| 3 | `pip` | Pip | 29 | Apricot, whiskers |
| 4 | `miso` | Miso | 36 | Wheat, forehead marks |
| 5 | `nori` | Nori | 44 | Sand, sage collar |
| 6 | `mochi` | Mochi | 53 | Clay pink, collar and bell |
| 7 | `sora` | Sora | 63 | Rose, bell |
| 8 | `hana` | Hana | 74 | Berry, bow |
| 9 | `kumo` | Kumo | 86 | Slate, scarf |
| 10 | `yuki` | Yuki | 99 | Mist blue, spark |
| 11 | `rokumon` | Rokumon | 112 | Ivory moon cat, ink ears, halo |

## Hub bridge

The game installs `window.Hub` if the host has not already provided one, and always mirrors calls to the parent frame.

```js
window.parent.postMessage({
  source: "rokumon-hub",
  game: "rokumon-cats",
  version: "1.0.0",
  method: "score",
  payload: {},
  timestamp: 0
}, "*");
```

| Method | When | Payload |
| --- | --- | --- |
| `Hub.ready(info)` | Once, on boot | `{ id, title, version, modes }` |
| `Hub.score(payload)` | Start, each drop, each merge, undo, and finish | Score object below |
| `Hub.event(name, data)` | `start`, `drop`, `merge`, `undo` | `{ name, ...data }` on the wire |
| `Hub.gameOver(payload)` | Stack stays above the limit | Same score object, plus `reason: "overflow"` |

`Hub.on(method, fn)` subscribes in standalone play and returns an unsubscribe function.

Score object:

```json
{
  "score": 480,
  "highestTier": 6,
  "highestTierId": "mochi",
  "highestTierName": "Mochi",
  "mode": "daily",
  "seed": "2026-11-10",
  "merges": 12,
  "maxCombo": 3,
  "session": 1,
  "personalBest": 480,
  "reason": "overflow",
  "durationMs": 84000,
  "undosUsed": 0
}
```

`score` and `highestTier` are integers. Sort leaderboards by `score` descending. Break ties with `highestTier`. Group daily boards by `seed`.

`game-hub-manifest.json` repeats this contract for the catalogue.

## Files

- `index.html` — the game (markup, CSS, and JS)
- `game-hub-manifest.json` — catalogue manifest
- `README.md` — this file

Physics is a small fixed-step circle solver (120 Hz, position correction, friction, low bounce). Sound is synthesized with the Web Audio API and starts after the first tap or key. Nothing is loaded from the network.
