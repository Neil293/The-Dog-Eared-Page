# CLAUDE.md

## Project overview

Single-file personal recipe PWA. The entire application lives in `index.html`. There is no build step, no bundler, and no package manager — everything runs directly in the browser.

## File structure

```
index.html      # Complete app — HTML, CSS, and JS in one file
manifest.json   # PWA manifest (icons, theme colour, display mode)
sw.js           # Service worker — caches app shell for offline use and enables install
icon.svg        # App icon referenced by manifest and apple-touch-icon
```

## Architecture

All state is managed in plain JS globals at the top of the `<script>` block:

| Variable | Purpose |
|---|---|
| `recipes` | Array of recipe objects, mirrored to `localStorage` and Firestore |
| `settings` | API keys, Firebase config, custom tag lists — stored in `localStorage` |
| `firebaseDb` | Firestore handle; `null` until the user connects Firebase in Settings |
| `editingId` / `viewingId` | ID of the recipe currently open in the edit or view modal |
| `activeIngFilters` / `activeMethFilters` | `Set` of active filter tag strings |

`localStorage` keys: `recipebook_recipes`, `recipebook_settings`.

## Key functions

| Function | What it does |
|---|---|
| `init()` | Loads recipes and settings from localStorage, renders filters and grid, reconnects Firebase if configured |
| `renderGrid()` | Filters recipes by tab / search / tag filters and re-renders the card grid |
| `renderFilters()` | Rebuilds the ingredient and method filter tag rows from `settings` |
| `viewRecipe(id)` | Populates and opens the view modal |
| `openAddRecipe(prefill?)` | Clears and opens the edit form; accepts a prefill object from AI import |
| `saveRecipe()` | Reads the edit form, upserts into `recipes`, saves to localStorage, syncs to Firebase |
| `analysePhoto()` | Sends the selected photo to the Anthropic API, parses the JSON response, calls `openAddRecipe(parsed)` |
| `initFirebase(silent?)` | Dynamically imports Firebase SDK, connects, merges remote and local recipes, starts `onSnapshot` listener |
| `syncToFirebase()` | Pushes all local recipes to Firestore (called after every `save()`) |
| `exportJSON()` / `importJSON()` | Serialise/deserialise the recipes array to/from a `.json` file |

## Patterns and constraints

- **No innerHTML with unsanitised user input.** Always pass user strings through `esc()` before inserting into HTML.
- **All ingredient amounts are stored and displayed in grams** — do not add other units.
- **Tags come in two flavours:** `ingTags` (ingredient-type) and `methodTags` (cooking method). Both arrays live on the recipe object and in `settings.ingTags` / `settings.methodTags`. Keep them separate.
- **Firebase is optional.** Code must work correctly when `firebaseDb` is `null`.
- **Dynamic Firebase import.** The Firebase SDK is loaded via `import()` only when the user clicks "Connect Firebase" — do not add it as a static `<script>` tag.
- **Anthropic API is called directly from the browser** with `anthropic-dangerous-direct-browser-access: true`. The API key is stored only in `localStorage`, never transmitted anywhere except to `api.anthropic.com`.
- **Single file constraint.** Keep app logic in `index.html`. `sw.js` and `icon.svg` exist as required companion files (service workers must be a separate file; icons must be a URL, not a `data:` URI) — do not add further external JS/CSS files.
- **No comments explaining what code does** — only add a comment when there is a non-obvious constraint or workaround.

## CSS conventions

CSS custom properties are defined in `:root`. Use them — don't hardcode colours or fonts:

```
--paper, --paper-dark, --paper-darker   background tones
--ink, --ink-light, --ink-faint         text tones
--accent, --accent-dark, --accent-light rust/terracotta highlights
--green, --green-light                  method tag colour
--font-display   Playfair Display (headings, titles)
--font-body      Lora (body text, form inputs)
--font-hand      Caveat (labels, tags, nav tabs)
```

## AI photo import flow

1. User selects an image → `handlePhotoFile()` reads it as base64
2. User clicks "Analyse" → `analysePhoto()` POSTs to `api.anthropic.com/v1/messages`
3. Model: `claude-opus-4-20250514`, vision message with the image + a strict JSON-only prompt
4. Response is parsed and passed to `openAddRecipe(parsed)` → `prefillForm(parsed)`
5. The base64 image is stored on the recipe as `imageData`

## Firebase sync flow

1. User enters config in Settings → clicks "Connect Firebase" → `initFirebase(false)`
2. Firebase SDK imports dynamically, Firestore initialised
3. If Firestore collection is non-empty: remote recipes merged in (remote wins on ID conflicts)
4. If empty: local recipes pushed up
5. `onSnapshot` listener keeps `recipes` in sync thereafter
6. Every `save()` call triggers `syncToFirebase()` which does a full `setDoc` for every recipe

## PWA / install flow

1. `sw.js` is registered in `index.html` via `navigator.serviceWorker.register('./sw.js')`
2. The service worker caches `index.html`, `manifest.json`, and `icon.svg` on install — the app works offline after first visit
3. The browser fires `beforeinstallprompt` when install criteria are met; the handler stores the event and shows the `#installBtn` (⬇) in the header
4. Tapping ⬇ calls `installApp()` which triggers `prompt()` on the stored event
5. After install the `appinstalled` event hides the button again
6. On iOS, install is via Safari's Share → Add to Home Screen (no `beforeinstallprompt` on Safari); the `apple-touch-icon` link in `<head>` controls the icon shown there

**To update cached assets:** bump the `CACHE` constant name in `sw.js` (e.g. `recipe-book-v2`). The activate handler deletes old caches automatically.

## Common tasks

**Add a new default tag:** push to `DEFAULT_ING_TAGS` or `DEFAULT_METHOD_TAGS` constants. Existing users who already have saved settings won't automatically get new defaults — that's intentional (they may have deleted them).

**Add a new field to a recipe:** add the form element, read it in `saveRecipe()`, render it in `viewRecipe()`. The object is schema-less — just add the key.

**Change the AI model:** update the `model` string in `analysePhoto()`.

**Change the AI prompt:** edit the `prompt` const in `analysePhoto()`. The model must return only a JSON object — preserve that constraint.
