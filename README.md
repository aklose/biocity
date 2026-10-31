# BioCity

An app that helps people make their yards more biodiverse. They scan the yard, AI identifies the plants, and the app suggests new native and pollinator-friendly plants to add. It also shows local plant shops and has libraries of local plants and pollinators.

## Status (2026-10-25)

**Stage: clickable prototype only.** None of the real app has been built yet.

- `prototype/index.html` is one self-contained HTML/JS file with mock data. It has no build step.
- Run it locally by opening `prototype/index.html` in a browser.
- **Live:** https://bob-squad-biocity.netlify.app (Netlify project `bob-squad-biocity`).
- `netlify.toml` publishes the `prototype/` folder as a static site with no build step.
- **Deploys are manual on purpose:** run `netlify deploy --prod` from the repo root. Pushing to GitHub does not redeploy, and the owner does not want GitHub linked to Netlify.
- **How it was checked:**
  - The inline script passes `node --check`.
  - Home, a project page and Shops were screenshotted in dark mode at 390px and 1440px wide; none of them scrolled sideways.
  - 2026-10-25: a Playwright click-through (guest scan → sign up → Yards near me → libraries) at 390px and 1440px had no JS errors and no sideways scrolling. All Wikimedia photo URLs returned 200.
  - Not yet checked: light mode, tablet widths (560–960px), and a full click-through on a real phone.

### What's mocked in the prototype
- **Plant ID:** it picks 3 random plants from `YARD_POOL`, one tree, one bush and one small plant. There is no real AI or database lookup. Match confidence rises with each extra close-up.
- **Accounts:** email only, no password or phone. The email is just a key in `localStorage`; no sign-in email is sent.
- **Photo locations:** each photo gets a random spot inside "your property" on a drawn satellite view, turned into lat/lng around a fixed point (`BASE`). There is no real GPS or parcel data.
- **Yards near me:** the users and projects in `NEARBY` are fictional.
- **Photos:** plant and pollinator photos are hotlinked Wikimedia Commons thumbnails (`pic()`). If one fails to load, the old drawn leaf art is used instead.
- **Region data:** all plants and pollinators are Northeast US. The zip code only changes the region label, using the first digit (`REGIONS`).
- **Shops:** fictional shops on a hand-drawn SVG map. "Visit website" opens `https://example.com/`.
- **Storage:** accounts (email, optional zip, projects, opt-in) are saved in `localStorage` under `biocity.v2`. Data under the old `biocity.v1` key is ignored. Each project stores only its first yard photo, downscaled; other photos keep just their location.

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

### App notes, 2026-10-25 (Google Doc "App notes", id `1UhN2ITmxiGysqUDajDx3RXugyq56GFlbRjgcpdz7F-M`, plus chat)
These override the earlier flow where they conflict.
- **Expand in place:** tapping a plant or pollinator makes its card bigger instead of opening a new page.
- **No light questions** in the scan.
- **Real photos** instead of drawn art.
- **Take as many photos as you can:** the app prompts for more yard photos and close-ups to make the ID clearer, and has an **I'm done** button.
- **Photo locations** are saved to a satellite-style map showing property outlines. A separate light-level app can reuse them.
- **Commonly found in** section for each plant.
- **Pollinators** list the plants that help them and the plants that harm them.
- **Accounts:** email required, no phone, zip code optional.
  - **Guests:** no My Yard. They can scan but nothing is saved, and they enter a zip code every visit.
  - **Signed-in users without a saved zip** are also asked every visit.
- **Yards near me** (account only, in the sidebar): a map like the shop map, with pins for nearby users and projects.
  - It is opt-in through an **Opt in** slider that can be changed any time.
  - People who opt out still see everyone else's pins.
- The corner navigation ("like Google Classroom") is already done.

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
  - `PLANTS`: plants keyed by id. Short field names: `n` name, `s` scientific name, `c` category, `st` status, `p` pollinator-friendly, `sun`, `w` water, `b` benefits, `d` description, `loc` commonly found in, `x` invasive action, `sg` can be suggested, `h` hue for the fallback leaf art, `i` Wikimedia Commons path.
  - `POLL`: pollinators. `pl` is the plant ids it helps; `hm` is `[plant id or name, reason]` pairs for plants that harm it.
  - `NEARBY`: fictional opted-in yards (`k:"yard"`) and projects (`k:"proj"`) for Yards near me.
  - `SHOPS`: fictional shops and their map pins.
  - `YARD_POOL`: plants the mock scan can "find".
- **Routing:** an in-memory `stack` of `{v, ...params}`. Each view is a function `V.<view>` that returns HTML.
- **Navigation:** the sidebar and tab bar use `ACT.nav`, which resets the stack to `[home, section]`. `navList()` changes with sign-in: guests get "Scan a yard" in place of My Yard. "Yards near me" is sidebar-only (flag `1`), so the phone tab bar keeps 5 items. `section()` decides which nav item is highlighted.
- **State:** `D` is persisted (`users`, `me`, `seen`), and `user()` is the signed-in account. `zip` is per session: it is loaded from the account if one is saved, otherwise asked for again. `guest` holds an unsaved guest scan, which is saved to the account if the guest signs up from it.
- **Expanding cards:** `plantItem()` and `pollItem()` render collapsed rows. `openKey` (`p:<id>` / `b:<id>`) picks the one shown expanded, spanning the full grid row.
- **Event handling:** delegated through `data-act` attributes, which map to handlers in `ACT`.
- **Forms:** `data-form` = `zip` or `account`. The opt-in switch is `data-toggle="optin"`.
- **Uploads:** file inputs marked `data-upload`.
- **New project flow:** state lives in `scan`, which moves through the steps `photo (many) → analyzing → closeup (many per plant, loops over plants)`. Every photo adds a `newShot()` to `scan.shots`. On the last plant, **I'm done** calls `finish()`, which saves the scan to the account or holds it in `guest`.
- **Maps:** `baseMap()` draws the shared mock map for Shops and Yards near me, and `pinSVG()` draws its pins. `satMap(pr)` draws the property satellite view with photo pins. `geojson(pr)` builds the "Export locations for a light app" download.
- **Suggestions:** `suggestions(found)` returns at most 3 per category. It excludes plants already found and all invasives, and puts natives first. It no longer filters by light.
- **Responsive layouts:**
  - **Phone (under 960px):** a single column with a bottom tab bar.
  - **Web (960px and up):** a left sidebar for navigation (`sidebar()`) and wider content. Lists use `.list` grids, and detail pages use the two-column `.split` layout.
  - Onboarding and the scan flow show neither the sidebar nor the tab bar (see `chrome()`).

## Checking layout changes
Headless Chrome won't render a window narrower than 500px, so a 390px `--window-size` screenshot is just a cropped 500px page. To see a real phone width:
- Open the page inside a 390px-wide `<iframe>`, and run Chrome with `--allow-file-access-from-files`.
- To skip onboarding, add a script after the app's own script that sets `S.zip`, `S.projects` and `stack`, then calls `render()`.
