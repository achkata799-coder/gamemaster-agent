---
name: questforge-visuals
description: Use native image generation as a Questforge table aid for scenes, objects, maps, inventory, merchants, outfits, symbols, recaps, persistent local galleries, interactive 360 POV viewers, and optional scene ambience with visual continuity.
---

# Questforge Visuals

Use this skill when a visual would clarify the shared imagination, preserve an important object or location, help spatial decisions, or make inventory/trade easier to understand.

If local file or browser access is unavailable, show generated static images in the conversation and skip local galleries and 360 viewers. Never block the next playable turn on an unsupported visual surface.

## Format Choice

Before generating, plan the visual beat:

```powershell
python ../../scripts/visual_planner.py --beat "<resolved player-facing beat>"
```

Use the returned `format`, `kind`, `continuity_requirements`, and `next_steps`.
If `should_generate` is false, answer in chat and skip image generation.
If `should_generate` is true, satisfy the continuity requirements before requesting native image generation.

A saved prompt is preparation, not a completed visual. Count a beat as generated only after an image asset exists, is registered, and is shown once in chat for static formats or linked through its viewer for `pov-360`.

The planner chooses among:

- `single_scene`: one location, one moment, one main action.
- `comic-page`: two or more places, times, or actions in one response.
- `map`: spatial decisions, routes, exits, fog of war, or tactical planning.
- `inventory`: current gear, loot, money, burden, or carried clues.
- `merchant`: buy/sell comparison boards.
- `reference_plate`: recurring character, object, outfit, creature, or place.
- `pov-360`: first-person spatial immersion.

Use `../../scripts/comic_panels.py classify --summary "<beat>"` when unsure. If the beat contains "después", "luego", "mientras", "antes de", "vuelve", "camino a", "paga y desayuna", "entra y sale", or several locations, do not request a single naturalistic scene unless the user explicitly asks for it.

## Cadence

For image-rich play, generate a visual for important turns where the fiction changes: decision consequences, dialogue turns with new posture or leverage, reveals, movement, danger, spell effects, item discoveries, maps, and scene transitions. Skip images for brief rules questions, quick clarifications, purely mechanical back-and-forth, or beats already covered by a recent useful image.

At the start of a new or continued session, the first player-facing scene should have a visual by default. After setup, file loading, or recap is complete, actually generate and register a fresh establishing image for the current scene or explicitly point to a still-current generated gallery image. Do not treat prompt-saved as completion and do not defer the opening visual merely because the scene is urgent. Do not let the first actionable scene be text-only unless the user asks for speed, the turn is only out-of-character setup, no in-fiction scene has started yet, or native image generation is unavailable.

Do not set a hard image cap by default. If the user enjoys a visual-first game, keep generating scene frames as the fiction advances. Static generated images should appear once in the Codex conversation so mobile users can see them, and they should also be registered in the local gallery for desktop history and review. Do not paste duplicate thumbnails or extra file links for the same static image. Use fewer images only when the user asks for speed, is asking rules questions, or is clearly trying to move quickly.

If native image generation is unavailable, say so only once when it first matters, log `visual_unavailable`, and continue play. Do not repeatedly save prompts, rebuild empty galleries, or imply that an image exists. When generation is available, invoke it; generating a prompt without invoking the tool is not a latency optimization.

Before returning any turn that selected a visual, check the corresponding visual-index status. `prompt-saved` requires one of two explicit outcomes in that same reply: generate/register/show the asset, or report the unavailable surface once, set the row to `unavailable`, and record `visual_unavailable`. Silent pending prompts are not allowed.

Default visual style should be immersive fantasy realism: grounded materials, clear staging, dramatic but plausible light, and no unwanted film grain. Use a different style only when the campaign has already established one or the user asks for it.

## Local Gallery

Use one stable local gallery per campaign as the desktop visual board and history. After each saved and registered native image, refresh it:

```powershell
python ../../scripts/visual_gallery.py --campaign-root <campaign-root> --title "<campaign title>"
```

The script writes `<campaign-root>/images/visual-gallery.html`, prints a `file:///` URL ending in `#latest`, and renders a chronological scroll log: older images at the top, newer images lower down, with every image in a stable same-size frame and one compact line such as `#35 · scene · Nela explica su olvido`. Panorama/360 entries are embedded as interactive viewer iframes in the gallery, not static images. If a panorama-like asset has no viewer yet, the gallery creates a default local viewer under `images/viewers/`.

After the first registered visual in each local play session, proactively return the stable `gallery_url` once as a compact Markdown link such as `[Open the live Questforge visual table](file:///.../visual-gallery.html#latest)`. Do not wait for the player to ask for a gallery or viewer. The link is opt-in: do not navigate the player's browser automatically unless the player asks to open or keep it beside the story. When left on `#latest`, the gallery refreshes itself and follows later static images and 360 viewers, so do not repeat the link after every beat.

If the current Browser surface refuses `file:///` navigation by policy, return the link and continue; do not spend play time on browser workarounds.

## Chat Image Surface

For every useful static generated image (`scene`, `location`, `character`, `creature`, `item`, `spell`, `map`, `inventory`, `merchant`, `outfit`, `symbol`, `recap`, and `comic-page`), show the selected image once in the conversation after it has been saved or copied into `images/assets/`.

Use standard Markdown with an absolute filesystem path.

Keep the nearby text short: narrate the scene normally, then show the image, or show the image after the consequence paragraph. Do not also return a separate `Image` button, prompt link, or raw local file path for that same static image unless the user asks for debugging details. The one stable live visual-table link offered after the first registered visual of the session is the deliberate exception. Continue to refresh the local gallery in the background for history and desktop play.

For `pov-360`, return the local viewer link and optionally open it with `@Browser`; do not treat the raw panorama as the main chat image unless the user explicitly asks for a flat preview.

## Pre-Session Prep

For important scenes, prepare visuals before live play. Use this for recurring characters, NPCs, enemies, maps, locations, persistent items, rewards, symbols, outfits, and likely action beats.

Complete a campaign-specific copy of the neutral prep spec from the accepted conception and current visual ledger. Never run the bundled empty structure unchanged or infer its subjects from a documentation example. Then create reference prompts:

```powershell
python ../../scripts/visual_prep.py --campaign-root <campaign-root> --spec ../../templates/visual-prep-spec.json
```

Then generate native images from the saved prompts, register selected PNGs, and mark accepted references as `canon`.

## Live Reuse

Before generating a scene with recurring visual elements, list usable canon anchors:

```powershell
python ../../scripts/campaign_memory.py list-visual-assets --campaign-root <campaign-root> --status canon --format markdown
```

Create live scene-frame prompts from the relevant anchors instead of rewriting continuity by hand:

```powershell
python ../../scripts/visual_reuse.py --campaign-root <campaign-root> --session <n> --scene <n> --label "<scene label>" --action "<player-facing action>" --roll "<actual roll or check>" --outcome "<resolved table outcome>" --anchor-label "<canon label>" --require-asset
```

Also check `images/visual-ledger.md` and preserve active visible states such as bandaged hands, disguises, carried objects, object scale, faction marks, and stable room layouts. Do not regress a visible state unless the fiction changes it.

## POV 360

When the player asks to look around or inspect a scene from the character's point of view, when the character enters an important location, or when spatial immersion would help, request a native equirectangular 360 panorama. Generate, register, and build its viewer in the same turn instead of waiting for a second request.

```powershell
python ../../scripts/panorama_viewer.py --image <generated-360.png> --output <campaign-root>/images/viewers/<label>-360.html --title "<scene title>" --narration "<short in-scene narration>" --initial-zoom-level 14
```

Always return the `viewer_url` as a compact Markdown link. Do not automatically navigate the in-app browser; clicking the link is the player's opt-in action.

For a scene that benefits from sustained ambience, select one licensed or user-provided loop before creating the viewer. Prefer a campaign-local approved library at `<campaign-root>/audio/library.json`; if it does not exist, use the bundled starter pack at `../../assets/audio/library.json`. Choose audio when ambience helps.

## Visual Kinds

Use these kinds with `save-visual-prompt`:

- `scene`
- `location`
- `character`
- `creature`
- `item`
- `spell`
- `map`
- `inventory`
- `merchant`
- `outfit`
- `symbol`
- `recap`
- `pov-360`
- `comic-page`

## Persistence

Before requesting native image generation, save the prompt:

```powershell
python ../../scripts/campaign_memory.py save-visual-prompt --campaign-root <campaign-root> --session <n> --scene <n> --kind <kind> --label "<label>" --prompt "<prompt>"
```

After generation, copy the selected image into `images/assets/` and register it:

```powershell
python ../../scripts/campaign_memory.py register-visual-asset --campaign-root <campaign-root> --asset-path images/assets/<asset-name>.png --kind <kind> --label "<label>" --session <n> --scene <n>
```

Refresh the gallery after registration:

```powershell
python ../../scripts/visual_gallery.py --campaign-root <campaign-root>
```

To mark a reviewed reference as canonical:

```powershell
python ../../scripts/campaign_memory.py set-visual-status --campaign-root <campaign-root> --kind <kind> --label "<label>" --session <n> --scene <n> --status canon
```

## Fog Of War

For maps:

- show only explored or known areas;
- hide secret rooms, hidden doors, enemy counts, trap mechanisms, and unexplored paths;
- use parchment shadow, mist, blank space, or unlabeled boundaries for unknown areas;
- keep labels sparse and table-useful.

## Copyright Boundary

Prompt only for original fantasy imagery. Do not request official D&D style, official logos, named settings, named characters, commercial adventure maps, or copied product art.

Use only audio loops that the user owns, generated for the project, or whose license permits the intended local or redistributed use. Do not commit or package music without clear license notes and required attribution.
