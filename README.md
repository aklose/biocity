# BioCity

An app that helps people make their yards more biodiverse. They scan the yard, AI identifies the plants, and the app suggests new native and pollinator-friendly plants to add. It also shows local plant shops and has libraries of local plants and pollinators.

## Status (2026-10-11)

**Stage: clickable prototype only.** None of the real app has been built yet.

- `prototype/index.html` is one self-contained HTML/JS file with mock data. It has no build step.
- Run it locally by opening `prototype/index.html` in a browser.
- **Live:** https://bob-squad-biocity.netlify.app (Netlify project `bob-squad-biocity`, deployed with the CLI and not yet linked to GitHub for auto-deploys).
- It is set up for Netlify: `netlify.toml` publishes the `prototype/` folder as a static site with no build step. Two ways to deploy:
  - **Netlify UI:** Add new site → Import from GitHub → `aklose/biocity`. Netlify reads `netlify.toml`, and each push to `main` redeploys.
  - **CLI:** run `netlify deploy --prod` from the repo root.
- **How it was checked:** the inline script passed a `node --check` syntax check. Nobody has clicked through it in a browser yet.

### What's mocked in the prototype
- **Plant ID:** it picks 3 random plants from `YARD_POOL`, one tree, one bush and one small plant. There is no real AI or database lookup.
- **Region data:** all plants and pollinators are Northeast US. The zip code only changes the region label, using the first digit (`REGIONS`).
- **Shops:** fictional shops on a hand-drawn SVG map. "Visit website" opens `https://example.com/`.
- **Storage:** zip code and projects are saved in `localStorage` under the key `biocity.v1`. Only a downscaled yard photo is stored; close-up photos are not.

## Original request

The design came from the user's Google Doc "Flowchart" (Drive file id `1N0rr7EZUIyLT7AGVlTYgbH6ShoqD46LXEwiwFJd7Pc8`), plus clarifications the user gave in chat.

### Flow from the doc
1. **Home** has 4 options: **My Yard**, **Shops**, **Plant Library**, **Pollinator Library**.
2. **First use only:** Home → **Next** → **Zip Code** entry → back to Home.
3. **My Yard:**
   - Take a picture of the yard.
   - The app asks for a closer picture of one plant at a time, repeating until all plants are done.
   - Answer a few questions about light levels.
   - Land on the **Improve Biodiversity** page.
4. **Improve Biodiversity:** a list of plants, organised by type (Tree, Small tree, Bush, Small bush, Small plant).
   - Each plant is tagged native, non-native or good for pollinators.
   - Each plant shows benefits, a description, and sunlight and water levels.
5. **Shops:** a map of the local area with pins for local shops. Tapping a pin opens that shop's website.
6. **Plant Library:** common native and invasive plants in your region. Tap a plant to learn more.
7. **Pollinator Library:** local pollinators and the plants they help.

### Clarifications from the user (these override the doc)
- **Buttons:** any button labelled "Picture" should say **My Yard**.
- **Zip code:**
  - It is used to find local shops and to set the region for native and invasive plant status.
  - The Next step and the zip prompt only appear on first use.
- **Saved projects:** yards are saved. My Yard lists all previous projects plus an option to start a new one.
- **Plant identification:**
  - Done with AI plus searches of reputable plant databases.
  - Use only the most reputable sources.
  - Results must be cross-referenced between sources.
- **Suggestions:** Improve Biodiversity does **not** list the plants in the photos. It suggests **new** plants that would broaden biodiversity.
- **Doc numbering:** the step numbers in the doc were inconsistent. The sections start at steps 11, 21, 25 and 29.
- **Build first:** the user asked for a quick prototype before the real build.

## Proposed real build (not started)
This was drafted with the user, but they chose to start with the prototype.

- **App:** Expo (React Native + TypeScript) with Expo Router.
  - `expo-camera` / `expo-image-picker` for photos
  - `react-native-maps` for the shop map
  - `expo-sqlite` to save projects on the device
- **Backend proxy** (Node or Cloudflare Worker) so API keys stay off the device. It would handle:
  - **Vision:** Claude (`claude-opus-5-5`) finds plants in the yard photo, decides which close-ups to ask for, and gives first-pass IDs.
  - **Cross-reference:** IDs are checked with Pl@ntNet or Plant.id and normalised against GBIF / USDA PLANTS.
    - If the sources agree, the ID is high confidence.
    - If they disagree, the app asks for another photo or shows the top candidates.
  - **Native/invasive status:** USDA PLANTS native status by state, plus state invasive lists and EDDMapS.
  - **Pollinators:** Xerces Society and Pollinator Partnership regional guides, plus iNaturalist observations.
  - **Suggestions:** rank region-native, database-verified species against the plants already in the yard and the light answers. Never suggest invasives.
  - **Shops:** OpenStreetMap Overpass or Google Places, searching for nurseries and garden centres near the zip code.
- **Open decisions:**
  - Final platform: Expo is assumed but not confirmed.
  - OpenStreetMap vs Google Places for shop data.
  - Which plant-ID API to use, and its cost.

## Prototype code map (`prototype/index.html`)
- **Data:**
  - `PLANTS`: plants keyed by id. Short field names: `n` name, `s` scientific name, `c` category, `st` status, `p` pollinator-friendly, `sun`, `w` water, `b` benefits, `d` description, `x` invasive action, `sg` can be suggested, `h` hue used for the leaf art.
  - `POLL`: pollinators, each with the plant ids it helps.
  - `SHOPS`: fictional shops and their map pins.
  - `YARD_POOL`: plants the mock scan can "find".
- **Routing:** an in-memory `stack` of `{v, ...params}`. Each view is a function `V.<view>` that returns HTML.
- **Event handling:** delegated through `data-act` attributes, which map to handlers in `ACT`.
- **Forms:** `data-form` = `zip` or `light`.
- **Uploads:** file inputs marked `data-upload`.
- **New project flow:** state lives in `scan`, which moves through the steps `photo → analyzing → closeup (loop) → light`. Submitting the light form saves the project.
- **Suggestions:** `suggestions(found, light)` returns at most 3 per category. It excludes plants already found and all invasives, and puts natives first.
