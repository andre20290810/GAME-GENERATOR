# GAME-GENERATOR (GGP — Game Generation Platform)

GGP turns one working game engine plus one config file plus your own assets
into a different playable game. This repository is self-contained: the
engine, the reference game's assets, the config and the Generator UI all
live here, with no runtime dependency on any other repository.

```
Generator UI ──> game config (game.json shape) ──> GGP engine ──> playable game
```

The engine and default assets are the DARK OUT 2 TPS prototype, kept here as
the **Reference / Template Game**: with the default `game.json` you get that
game unchanged. Version: **GGP Generator v0.1**, imported from
`andre20290810/DARKOUT-TPS` branch `ggp-generator-v0.1`
(commit `a9b885915418bcd45726b8d86e6cb133ba4f367c`).

## Run locally

No build step. Serve the folder with any static web server:

```
python3 -m http.server 8000
# Generator:      http://localhost:8000/generator.html
# Default game:   http://localhost:8000/index.html   (reads game.json)
```

The Generator needs http(s); opened from `file://` its preview cannot read
the config. The game alone also runs from `file://` (it then uses the
built-in defaults in `ggp/ggp-boot.js`, identical to `game.json`).

## Deploy

Plain static site, output directory = repository root, no build command.
There is no `CNAME`; set a custom domain in the hosting settings if you want one.

**Cloudflare Workers (Static Assets)** — `wrangler.jsonc` serves the repository
root as static assets with no Worker script. `.assetsignore` keeps repository
files (`.git`, `README.md`, the config files) out of the upload.

```
npx wrangler deploy            # first run asks you to log in to Cloudflare
```

Or connect the repository in the Cloudflare dashboard (Workers & Pages >
Create > Import a repository) with deploy command `npx wrangler deploy` and
no build command. The site is then at `https://game-generator.<subdomain>.workers.dev`
(`/` = default game, `/generator` = Generator). Cloudflare drops `.html` from
URLs with a redirect that keeps the query string, so the Generator preview
(`index.html?ggp=preview`) works unchanged.

Also works as-is on GitHub Pages (`.nojekyll` included).

## Files

| Path | Role |
|---|---|
| `generator.html`, `ggp/generator.js`, `ggp/generator.css` | Generator UI: uploads, parameters, PREVIEW |
| `game.json` | the game config (default = reference game) |
| `ggp/ggp-boot.js` | resolves the config (Generator preview > `game.json` > built-in), exposes `window.GGP`, then loads `game.js` |
| `index.html`, `game.js`, `style.css`, `assets/` | GGP engine + reference game assets |

## game.json

```json
{
  "ggpVersion": "0.1",
  "title": null,
  "player": { "image": null, "hp": 300 },
  "enemy": {
    "image": null,
    "imageTargets": ["drone", "roid1", "roid2", "gabriel", "adamSphere", "adam"],
    "hp": 300,
    "speedMultiplier": 1
  },
  "stage": { "background": null, "escapeTimeLimitSec": 90 },
  "audio": { "battle": "assets/audio/after_the_limits.mp3" }
}
```

| Field | Engine value it sets |
|---|---|
| `title` | page title + MODE SELECT title (`null` = unchanged) |
| `player.image` | every frame in `assets/player/` and `assets/player_escape/` (`null` = original art) |
| `player.hp` | `PLAYER_MAX_HP` (default 300); also selects the matching PAUSE MENU LIFE tier |
| `enemy.image` + `enemy.imageTargets` | every frame of the listed enemies (`null` = original art) |
| `enemy.hp` | `ENEMY_MAX_HP` (default 300; ROID 1/2 stay at 2x, as in the original) |
| `enemy.speedMultiplier` | x `ENEMY_IDLE_APPROACH_SPEED` and `CLAW_STALK_SPEED` (COMBAT approach) |
| `stage.background` | backdrop painted behind the corridor (`null` = none) |
| `stage.escapeTimeLimitSec` | `ESCAPE_TIME_LIMIT_SEC` (COMBAT has no time limit in this engine) |
| `audio.battle` | battle BGM (`#bgm-audio`) |

Replacement images are fitted into each original frame's own canvas size and
measured body box before the engine sees them, so sprite scaling and
anchoring stay as tuned.

## Controls (reference game)

Gamepad-first, landscape only, with on-screen touch controls as a fallback.

- D-PAD: move · LEFT STICK: flashlight · RIGHT STICK: aim
- RB: fire · LB: reload · RT/LT: east/west dash · B/A: north/south dash
- X: stealth · Y: flash
