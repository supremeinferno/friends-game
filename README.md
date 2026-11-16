# friends-game

Build a 2D top-down open-world browser game together.

**Play it:** https://supremeinferno.github.io/friends-game/ (republished from `main` after merges)

## Run it

```sh
npm install
npm run dev      # open the printed URL
npm test         # unit tests (Vitest)
npm run build    # type-check + production build into dist/
```

### Controls

| Key | Action |
| --- | --- |
| WASD / arrows | Move (Shift to run) |
| E | Talk / trade with a nearby villager |
| Q | Eat a berry to restore a heart |
| F | Cast a magic bolt (you're a wizard!) |
| 1 / 2 / 3 | Use a power stone |
| M | Toggle the minimap |
| V | Mute / unmute sound |
| Phone | Drag on the left half to move; RUN and MAP buttons on the right |

The game auto-saves (world, position and any registered feature state) to your browser and
continues where you left off. Add `?seed=123` to the URL to open a specific world, or `?new` for a fresh random one.

## Plain-JS prototype

`prototypes/plain-js/` holds the earlier no-build scaffold from PR #1, kept for reference and ideas. With `npm run dev` running, open `/prototypes/plain-js/` to try it.

## How the code is organised

Plain TypeScript + HTML Canvas, bundled with Vite. No game engine, so anyone can read it.

| Path | What it does |
| --- | --- |
| `src/main.ts` | Wires everything together and runs the game loop |
| `src/game.ts` | Shared `Game` state and the `Entity` interface for NPCs, items, ... |
| `src/engine/` | Reusable pieces: `loop.ts` (fixed-timestep loop), `input.ts` (keyboard), `camera.ts` |
| `src/world/` | Endless procedural world: `noise.ts`, `tiles.ts` (tile types, colours, solidity), `decor.ts` (trees, pines, bushes, flowers), `castles.ts`, `world.ts` (chunked generation, `getTile`, `getDecor`, `isSolidAt`) |
| `src/player/` | The player is a powerful wizard drawn on the animated pixel hero (`sprite.ts`); `spell.ts` (F casts a magic bolt; entities can react via `onSpellHit`); `health.ts` (5 hearts, Q eats a berry to heal, faint → wake at the last campfire or spawn) |
| `src/save/` | Auto-save to localStorage; features persist state with `game.saves.register(key, { save, load }, game.resume)` |
| `src/audio/` | Procedural WebAudio: footsteps, ambient tune, effects via `game.sound?.play('roar')` |
| `src/ui/` | Overlays: minimap, day/night tint and clock, hearts, inventory and power-stone HUDs, touch controls |
| `src/items/` | Collectible coins, berries, mushrooms and gems, placed per chunk from the seed; `game.inventory` holds counts and is saved |
| `src/npc/` | Villagers near the start: `villagers.ts` (names, lines, trades), `npc.ts` (strolling entity), `dialogue.ts` (conversation, trade and quest logic), `quests.ts` (one-time "bring me" quests, saved), `talk.ts` (E to talk, dialogue box, quest tracker) |
| `src/animals/` | Cute fairies that flutter over meadows, fly over to visit you and sprinkle fairy dust that restores a heart |
| `src/knights/` | Chess-knight guards around castles |
| `src/power/` | Rare power stones you collect and use with 1/2/3 |
| `src/wizard/` | A wizard who follows you and casts bolts (hide behind trees) |
| `src/campfire/` | Campfires placed per chunk from the seed: walk up to one to light it, and after fainting you wake up at the last one you lit. Lit fires glow through the night; saved |
| `src/render.ts` | Draws visible tiles and the player |
| `tests/` | Vitest unit tests |

World coordinates are in pixels; one tile is `TILE_SIZE` (32) pixels. The world is generated
lazily in 32×32-tile chunks and is fully determined by its seed.

## Adding things to the world

Implement `Entity` from `src/game.ts` (`x`, `y`, optional `update(dt, game)` and
`draw(ctx, camera)`, set `dead = true` to remove) and push it onto `game.entities`.
Entities are drawn in y order with the player. `window.game` is available in the browser console.

## Contributing

- Pick an open Issue, say you're on it, and work on a branch. Keep PRs small, and keep `npm test` and `npm run build` green.
- **Merge fast:** the owner wants every ready PR merged into `main` right away, yours or a teammate's. Rebase onto `main`, resolve conflicts, run the tests and build (plus a quick browser check), then merge. Don't leave PRs open.
- CI on GitHub Actions is currently blocked by an account billing issue, so local test runs are the check for now.
- Prefer adding new modules (e.g. `src/npc/`, `src/items/`) over growing `main.ts`. Hook them in with one line where possible, and keep game logic testable without the DOM.
