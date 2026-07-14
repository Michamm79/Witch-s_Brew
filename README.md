# Witch's Brew

A 2.5D handheld-style pixel game in the visual and gameplay tradition of *The Legend of Zelda: The Minish Cap*. You play a witch in a small medieval town, collecting ingredients and mixing them into potions at your cauldron to fulfill villager requests.

**Status:** In active development / prototype. Core loop is playable; content and polish are ongoing.

---

## The Hook

You do not know which ingredients make which potion until you try the combination. Mixing is a small discovery game:

- Get it right, and the recipe is permanently recorded in your journal and the potion goes on your shelf.
- Get it wrong, and the cauldron erupts in a harmless, cartoonish explosion. No penalty beyond losing the ingredients you used.

Recipes are order-independent (Ingredient A + B is the same as B + A) and use 1–3 ingredients per attempt.

## Tech Stack

Built with **Phaser 3 + TypeScript + Vite**. Runs in any browser, desktop or mobile, with no build step beyond the setup below.

**There are no image files in this project.** Every piece of art — the witch, villagers, houses, cobblestones, ingredients, potions, UI icons, and explosion effects — is generated at startup from hand-authored pixel grids in code.

## Running It Locally

```bash
npm install
npm run dev
```

Open the local URL printed in your terminal.

```bash
npm run build
```

Produces a static production bundle (HTML/JS/CSS) deployable to any static file host.

## Gameplay Features

- **Movement** — WASD/arrow keys, or an on-screen thumbstick for touch devices. Directional sprites with a 2-frame walk cycle.
- **Ingredient collection** — Walk over a scattered ingredient node to auto-collect it into your pouch. Nodes respawn after ~12 seconds.
- **Mixing** — Interact with the cauldron (E key or on-screen button) to open the Cauldron panel. Drag or tap up to 3 ingredients into slots and hit Mix.
- **Journal** — A running log of every recipe you have personally discovered, with its exact ingredient list.
- **Inventory** — At-a-glance view of current ingredient and brewed-potion counts.
- **Villagers & delivery** — Four townspeople with rotating potion requests. Nothing is missable or timed; orders simply wait.
- **Town layout** — A single 30×20 tile town with four villager houses, a well, and the witch's cottage/cauldron, all built from tiled, procedurally generated pieces. Depth (Y-sort) gives the witch correct front/behind layering relative to buildings and villagers, which is what produces the "2.5D" read on top of an otherwise top-down game.

## Current Content

- **10 ingredients** — Moon Petal, Fire Cap, Dew Root, Bat Wing, Honeycomb, Nightshade Berry, River Kelp, Ember Dust, Frostberry, Raven Feather
- **8 potions/recipes** — Healing, Night Vision, Courage, Sleepy, Love Charm, Swiftness, Warmth, Invisibility (a mix of 2- and 3-ingredient combinations)
- **4 villagers**, each cycling through 3 possible requests
- **14 ingredient spawn points** spread around the map

All content lives in small, flat data files, so adding a 5th villager or a 9th potion is a data-only change — no engine code needs to be touched.

## Architecture

```
src/
  data/          Plain data & constants — no logic
    ingredients.ts   IngredientId union type + name/description/color per ingredient
    potions.ts       PotionId union type + name/description/color per potion
    recipes.ts       The hidden recipe table + findRecipe() lookup (order-independent)
    villagers.ts     Villager definitions: name, colors, position, order pool
    townMap.ts       Map dimensions, house footprints, ingredient spawn points, well/cauldron/player positions
    palette.ts       Shared color palette + tile size / display scale constants

  gfx/           Procedural pixel-art texture generation
    PixelGrid.ts       Core helper: rasterizes a hand-authored grid of characters
                       (one per pixel) into a Phaser texture via fillRect
    WitchSprites.ts    The witch: hat, hair, face, robe, 3 facing directions x 2 walk frames
    VillagerSprites.ts A simplified villager, recolored per NPC
    TownTiles.ts       Cobblestone, grass, tudor-style walls/roof/door/window, well, path
    ItemSprites.ts     Ingredient icons, potion bottles, the cauldron, explosion/sparkle FX
    UiSprites.ts       HUD icons: journal, inventory, cauldron, close, checkmark, speech bubble

  entities/      Game objects that live in the world
    Player.ts             Movement, facing/animation state, depth sorting
    Villager.ts           Order state, speech bubble, name tag, deliver/reject FX
    IngredientPickup.ts   A collectible node: bob animation, collect(), respawn timer

  systems/       Cross-cutting state, not tied to any one scene
    GameState.ts       Single source of truth: ingredient counts, potion counts,
                       discovered recipes, and the mix() resolution logic.
                       Extends Phaser's EventEmitter so UI reacts without polling.
    SharedInput.ts     A tiny shared vector the on-screen joystick writes to
                       and the town scene reads each frame

  scenes/        Phaser Scenes (top-level game states)
    BootScene.ts       Generates every texture, creates GameState, starts the town
    TownScene.ts       The world: ground/houses/well/cauldron/villagers/pickups,
                       player, input, collision, proximity checks, interaction
    UIScene.ts         Runs parallel to TownScene: HUD, joystick, interact button,
                       toasts, and the three modal panels

  ui/            HUD panels (Phaser game objects, not DOM)
    Panel.ts            Shared parchment-modal chrome, reused by all panels
    JournalPanel.ts      Lists discovered recipes
    InventoryPanel.ts    Lists ingredient/potion counts
    CauldronPanel.ts     Mixing UI: slot selection, Mix button, success/failure FX
    Joystick.ts          On-screen touch thumbstick

main.ts          Creates the Phaser.Game instance (405x720 logical size,
                  pixelArt: true, FIT scaling) and registers the 3 scenes
```

## Key Technical Decisions

- **No binary art assets.** Every sprite, tile, and icon is a small 2D array of characters mapped to colors, rasterized with `fillRect` at one pixel per cell, and converted to a texture once via `generateTexture`. This keeps the entire game as plain, diffable TypeScript, with no separate art pipeline.
- **Pixel-perfect rendering.** Configured with `pixelArt: true` and an integer camera zoom (3x), so small source textures scale up crisply instead of blurring.
- **Event-driven UI.** `GameState` is a plain `EventEmitter`. Panels subscribe to events like `inventory-changed` and re-render themselves rather than being manually refreshed by scene code.
- **Manual collision, not Arcade Physics.** The map is simple enough (a handful of rectangular house footprints, the well, the cauldron) that player movement uses straightforward AABB overlap checks with per-axis sliding, rather than pulling in Phaser's physics engine — one less system to reason about at this scope.
- **Depth = Y position.** Every world sprite sets its render depth to its Y coordinate each frame, the standard technique for correct front/behind layering in a top-down game without a real 3D engine.

## License

TBD.
