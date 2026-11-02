# BioCity

An app that helps people make their yards more biodiverse. They scan the yard, AI identifies the plants, and the app suggests new native and pollinator-friendly plants to add. It also shows local plant shops and has libraries of local plants and pollinators.

## Status (2026-11-01)

**Stage: clickable prototype only.** None of the real app has been built yet.

- `prototype/index.html` is one self-contained HTML/JS file with mock data. It has no build step.
- Run it locally by opening `prototype/index.html` in a browser.
- **Live:** https://bob-squad-biocity.netlify.app (Netlify project `bob-squad-biocity`).
- `netlify.toml` publishes the `prototype/` folder as a static site with no build step.
- **Deploys are manual on purpose:** run `netlify deploy --prod` from the repo root. Pushing to GitHub does not redeploy, and the owner does not want GitHub linked to Netlify.
- **Look:** solid, multicolored theme in light and dark, using lighter tints of safety orange, Pantone yellow, NCS green and royal purple, plus teal, berry and red. The page background is a cartoon scene: a flat Argentinian blue (`#6CB4EE`) sky, plain white clouds in a band at the top, a plain yellow sun and rolling green hills at the bottom. Dark mode uses a night sky with stars and a crescent moon in place of the sun. Only the header and the phone tab bar's current-section oval are tied to the section.
- **How it was checked:**
  - The inline script passes `node --check`.
  - Home, a project page and Shops were screenshotted in dark mode at 390px and 1440px wide; none of them scrolled sideways.
  - 2026-10-25: a Playwright click-through (guest scan → sign up → Yards near me → libraries) at 390px and 1440px had no JS errors and no sideways scrolling. All Wikimedia photo URLs returned 200.
  - 2026-11-01, by browser script (signed in):
    - A full run from the first quiz question to a saved project had no JS errors, and the answers were saved.
    - 12,288 quiz answer combinations all returned at least one plant. Plants that matched directly fit the answers.
    - Back navigation through the quiz, sample photos changing on each tap, the dark/light choice surviving a reload, and the Nearby tab at 390px and 320px all worked.
    - All newly added Wikimedia photo URLs returned 200.
  - 2026-11-01, by screenshot: the current look was checked on Home, Plant Library, the quiz and other main pages, in both modes at 390px and at desktop widths up to 2560px. Not every page was checked in every mode or width. Home was also seen at 560px and 800px.
  - 2026-11-01, by calculation: theme text colors on solid backgrounds meet 4.5:1 contrast. Text over the sun or moon was only judged by eye, and an error message on the lowest part of a night cloud falls just short at 4.46:1.
  - Not yet checked: the quiz as a guest, sign-up in the current version, most pages at tablet widths (560–960px), and a full click-through on a real phone.

### Open items
**Decisions for the owner**
- **Home button:** it is now last in the desktop sidebar. Confirm that's what was meant, or give it more space under the logo instead.
- **Header on large screens:** it stops at the content's maximum width, with sky on both sides. Running it full width is recommended.
- **Moon behind text:** in dark mode on phones, pale text crosses the moon on some pages. Dim the moon or move it.
- **Sample yard photos:** they show different gardens, not one yard from different angles. Keep them, or supply photos of one yard.
- **Guest tile colors:** Home tiles go by position, so a guest's tiles don't always match the section headers. Fix each section's tile color for everyone, or leave it.
- **Desktop sidebar:** the selected item is always purple. Match it to the header color, or leave it.
- **Shops teal:** teal is close in brightness to the sky. Deepen it, or leave it.
- **Non-native tag:** it stays sky blue so it doesn't look like the green native tag. Make it teal, or leave it.
- **Nearby label:** the phone bar says "Nearby" because "Yards near me" doesn't fit on one line. Use a two-line label, or leave it.
- **Recent projects red:** it is a soft coral because of the lighter palette. Use a stronger red, or leave it.
- **`CLAUDE.md`:** consider adding a rule to check visual changes in both modes at phone and desktop widths, and a rule that text on the sky must stay readable over the clouds, sun and moon.

**Known limits**
- At 320px wide, the Pollinators label sits tight against Plants in the phone bar.
- Night clouds are silvery gray rather than white, so light text stays readable over them.
- The time question uses water needs as a stand-in, and the wildlife matches rely on placeholder data. See "What's mocked" below.
- About 10% of quiz answer combinations have no exact match because the plant list is small, so the app widens the search.

### What's mocked in the prototype
- **Plant ID:** it picks 3 random plants from `YARD_POOL`, one tree, one bush and one small plant. There is no real AI or database lookup. Match confidence rises with each extra close-up.
- **Accounts:** email only, no password or phone. The email is just a key in `localStorage`; no sign-in email is sent.
- **Photo locations:** each photo gets a random spot inside "your property" on a drawn satellite view, turned into lat/lng around a fixed point (`BASE`). There is no real GPS or parcel data.
- **Yards near me:** the users and projects in `NEARBY` are fictional.
- **Sample photos:** each tap of a sample button adds a different photo from `YARD_SAMPLES` or `CU_SAMPLES`, then the list repeats. The yard samples show different gardens, not one yard from several angles.
- **Photos:** plant and pollinator photos are hotlinked Wikimedia Commons thumbnails (`pic()`). If one fails to load, the old drawn leaf art is used instead.
- **Quiz matching:** plants have no real care data, so the time answer uses water needs as a stand-in. "A little" allows Low and Medium water, and "Some" excludes High. Pollinator matches come from `POLL` lists and words in each plant's benefits text. Songbird matches come from that text too, and the other animal matches are short hand-written lists in `QUIZ`. A plant only needs to match one chosen pollinator or animal.
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
- **No light questions** in the scan. *(Replaced on 2026-11-01: see the setup quiz under Clarifications.)*
- **Real photos** instead of drawn art.
- **Take as many photos as you can:** the app prompts for more yard photos and close-ups to make the ID clearer, and has an **I'm done** button.
- **Photo locations** are saved to a satellite-style map showing property outlines. A separate light-level app can reuse them.
- **Commonly found in** section for each plant.
- **Pollinators** list the plants that help them and the plants that harm them.
- **Accounts:** email required, no phone, zip code optional.
  - **Guests:** no My Yard. They can scan but nothing is saved, and they enter a zip code every visit.
  - **Signed-in users without a saved zip** are also asked every visit.
- **Yards near me** (account only, in the sidebar and phone tab bar): a map like the shop map, with pins for nearby users and projects.
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
- **Setup quiz (2026-11-01):** a new project starts with five questions, shown one at a time: light, plant types, pollinators to attract, animals to attract and time available. Each question has its own solid color. The answers shape the suggestions.
- **Plant categories (2026-11-01):** the app now shows large trees, small trees, large shrubs, small shrubs, flowers and small plants. In the data these are still `Tree`, `Small tree`, `Bush`, `Small bush`, plus `Flower` and `Small plant`.
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
- **Navigation:** the sidebar and tab bar use `ACT.nav`, which resets the stack to `[home, section]`. `navList()` changes with sign-in: guests get "Scan a yard" in place of My Yard. "Yards near me" appears in the phone tab bar as "Nearby" for signed-in users, giving them 6 items; guests have 5. `section()` decides which nav item is highlighted. `navItems()` lists Home last in the sidebar and first in the tab bar.
- **State:** `D` is persisted (`users`, `me`, `seen`), and `user()` is the signed-in account. `zip` is per session: it is loaded from the account if one is saved, otherwise asked for again. `guest` holds an unsaved guest scan, which is saved to the account if the guest signs up from it.
- **Expanding cards:** `plantItem()` and `pollItem()` render collapsed rows. `openKey` (`p:<id>` / `b:<id>`) picks the one shown expanded, spanning the full grid row.
- **Recent projects:** the Home section uses a fixed red through `.recent`, outside the color rotation.
- **Dark/light toggle:** `themeBtn()` draws the button at the right of every header, and as a floating button on the welcome screen. The choice is saved as `D.theme` and sets `data-theme` on the root. Until someone picks, the app follows the device setting. The handler updates the button in place instead of calling `render()`, so typed form input isn't lost.
- **Event handling:** delegated through `data-act` attributes, which map to handlers in `ACT`.
- **Forms:** `data-form` = `zip` or `account`. The opt-in switch is `data-toggle="optin"`.
- **Uploads:** file inputs marked `data-upload`.
- **New project flow:** state lives in `scan`, which moves through the steps `quiz → photo (many) → analyzing → closeup (many per plant, loops over plants)`. Every photo adds a `newShot()` to `scan.shots`. On the last plant, **I'm done** calls `finish()`, which saves the scan to the account or holds it in `guest`.
- **Maps:** `baseMap()` draws the shared mock map for Shops and Yards near me, and `pinSVG()` draws its pins. `satMap(pr)` draws the property satellite view with photo pins. `geojson(pr)` builds the "Export locations for a light app" download.
- **Quiz:** `QUIZ` holds the options and `QSTEPS` the question order and colors. `scan.qi` is the current question, and `scan.quiz` is saved on the project as `pr.quiz`. Back steps through the questions with answers kept, and back from the photo step returns to the last question.
- **Suggestions:** `suggestions(found,quiz)` returns `{groups,relaxed}` with at most 3 plants per category. It excludes plants already found and all invasives, and puts natives first. With quiz answers, it filters by light, type, time, and wanted pollinators and animals. If nothing matches, it drops time, then wanted pollinators and animals, then type, then light, and sets `relaxed`. Projects with no quiz are unfiltered.
- **Colors:** named colors (`--green`, `--orange`, `--teal`, `--yellow`, `--purple`, `--berry`, `--red`) each have `-soft` and `-deep` versions, redefined for dark mode. `--sky` is only for the non-native tag. Royal purple is `--primary`, used for the selected nav item and map pins. The color of the header and the phone tab bar's current-section oval (`--h`, `--h-on`) changes by section through `data-sec` on `.app`: berry for Home, green for My Yard and scans, orange for Yards near me, teal for Shops, yellow for Plants and purple for Pollinators. Tiles and groups rotate through the colors by position using `--c`, `--c-on`, `--c-soft` and `--c-deep`. Buttons, standalone rows, cards, segmented tabs and chips inside `main` get a rotating `data-c` from `render()`, so one page mixes colors. Plant and pollinator cards get their own rotation, so opening one doesn't recolor the others. All fills are light enough for dark text (`--on-c`, `--on-primary`). The scene is a fixed `body::before` using `--sky-bg` and the inline SVGs in `--sun`, `--stars`, `--clouds` and `--hills`. `--scene-x` centers the sun over the content area, allowing for the sidebar on wide screens. Text sitting directly on the scene gets a soft `--halo` text shadow so it stays readable over the scene. The clouds use the same `--scene-x`, and their image leaves its middle empty, so they never overlap the sun at any screen width.
- **Responsive layouts:**
  - **Phone (under 960px):** a single column with a bottom tab bar.
  - **Web (960px and up):** a left sidebar for navigation (`sidebar()`) and wider content. Lists use `.list` grids, and detail pages use the two-column `.split` layout.
  - Onboarding and the scan flow show neither the sidebar nor the tab bar (see `chrome()`).

## Checking layout changes
Headless Chrome won't render a window narrower than 500px, so a 390px `--window-size` screenshot is just a cropped 500px page. To see a real phone width:
- Open the page inside a 390px-wide `<iframe>`, and run Chrome with `--allow-file-access-from-files`.
- To skip onboarding, add a script after the app's own script that sets `S.zip`, `S.projects` and `stack`, then calls `render()`.
