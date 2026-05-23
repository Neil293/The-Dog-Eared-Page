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
| `firebaseAuth` | Auth handle (`{ auth, GoogleAuthProvider, signInWithPopup, signOut }`); `null` until Firebase connected |
| `currentUser` | Firebase `User` object; `null` when signed out |
| `_unsubSnapshot` | Unsubscribe function for the active Firestore `onSnapshot` listener; `null` when not listening |
| `editingId` / `viewingId` | ID of the recipe currently open in the edit or view modal |
| `activeIngFilters` / `activeMethFilters` | `Set` of active filter tag strings |
| `window._pendingImageData` | Base64 data URL of photo staged in the edit form; `null` when no photo |

`localStorage` keys: `recipebook_recipes`, `recipebook_settings`.

## Key functions

| Function | What it does |
|---|---|
| `init()` | Loads recipes and settings from localStorage, renders filters and grid, reconnects Firebase if configured |
| `renderGrid()` | Filters recipes by tab / search / tag filters and re-renders the card grid |
| `renderFilters()` | Rebuilds the ingredient and method filter tag rows from `settings` |
| `toggleFilterSection(id)` | Collapses/expands a filter section; updates the active-count badge |
| `viewRecipe(id)` | Populates and opens the view modal |
| `openAddRecipe(prefill?)` | Clears and opens the edit form; accepts a prefill object from AI import |
| `saveRecipe()` | Reads the edit form, upserts into `recipes`, saves to localStorage, syncs to Firebase |
| `compressImage(file, maxWidth, quality)` | Canvas-based image compression — default 900 px wide, 72 % JPEG; returns a data URL |
| `setRecipePhotoPreview(dataUrl)` | Stores data URL in `window._pendingImageData` and shows the preview; `null` clears it |
| `handleRecipePhoto(file)` | Compresses a File and calls `setRecipePhotoPreview` |
| `removeRecipePhoto()` | Clears `window._pendingImageData` and hides the preview |
| `openImgSearch()` | Opens the image-search modal |
| `searchImages()` | Queries the Openverse API and renders a grid of results |
| `selectSearchImage(el)` | Reads `data-full` / `data-thumb` from a result tile, fetches the full image via proxy, compresses and sets it as the recipe photo |
| `fetchViaProxy(url)` | Fetches a URL through a CORS proxy chain: allorigins.win → corsproxy.io → codetabs.com |
| `callAI(messages)` | Calls Anthropic if `settings.anthropicKey` is set, otherwise Gemini; returns the text response |
| `analysePhoto()` | Sends the selected photo to AI via `callAI()`, parses the JSON response, calls `openAddRecipe(parsed)` |
| `initFirebase(silent?)` | Dynamically imports Firebase SDK + Auth, connects, sets up `onAuthStateChanged` listener |
| `setupFirestoreForUser(user)` | Scopes Firestore to `users/{uid}/recipes`, merges remote/local on first sign-in, starts `onSnapshot` |
| `signInWithGoogle()` | Opens Google sign-in popup; `onAuthStateChanged` triggers `setupFirestoreForUser` on success |
| `signOutFirebase()` | Signs out; detaches snapshot listener and clears `currentUser` |
| `updateAuthUI()` | Shows/hides sign-in button and user name in the Settings modal auth section |
| `syncToFirebase()` | Pushes all local recipes to `users/{uid}/recipes` (requires `currentUser`) |
| `loadSettingsFromFirebase(user)` | Reads `users/{uid}/meta/settings`; merges API keys and tag lists into local settings; pushes local settings on first sign-in |
| `syncSettingsToFirebase()` | Writes API keys and tag lists to `users/{uid}/meta/settings`; called from `saveSettings()` |
| `exportJSON()` / `importJSON()` | Serialise/deserialise the recipes array to/from a `.json` file |

## Patterns and constraints

- **No innerHTML with unsanitised user input.** Always pass user strings through `esc()` before inserting into HTML.
- **All ingredient amounts are stored and displayed in grams** — do not add other units.
- **Tags come in two flavours:** `ingTags` (ingredient-type) and `methodTags` (cooking method). Both arrays live on the recipe object and in `settings.ingTags` / `settings.methodTags`. Keep them separate.
- **Firebase is optional.** Code must work correctly when `firebaseDb` is `null`.
- **Dynamic Firebase import.** The Firebase SDK is loaded via `import()` only when the user clicks "Connect Firebase" — do not add it as a static `<script>` tag.
- **Anthropic API is called directly from the browser** with `anthropic-dangerous-direct-browser-access: true`. The API key is stored only in `localStorage`, never transmitted anywhere except to `api.anthropic.com`.
- **Gemini API** is used as a fallback when no Anthropic key is set. The key is stored only in `localStorage`.
- **Single file constraint.** Keep app logic in `index.html`. `sw.js` and `icon.svg` exist as required companion files (service workers must be a separate file; icons must be a URL, not a `data:` URI) — do not add further external JS/CSS files.
- **No comments explaining what code does** — only add a comment when there is a non-obvious constraint or workaround.
- **`backdrop-filter: blur`** must only be applied to `.modal-overlay.open`, not to all modal overlays — keeping it on all overlays causes continuous GPU compositing and visible screen flicker.
- **Image storage:** photos are compressed to ~80–150 KB before being stored as base64 on the recipe object. This keeps Firestore documents well under the 1 MB limit.

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
2. User clicks "Analyse" → `analysePhoto()` calls `callAI()` with the image
3. `callAI()` picks Anthropic (`claude-opus-4-20250514`) if key is set, else Gemini (`gemini-2.0-flash`)
4. Response is parsed and passed to `openAddRecipe(parsed)` → `prefillForm(parsed)`
5. The base64 image is stored on the recipe as `imageData`

## Recipe photo flow

1. User clicks "📷 Add photo" in the edit form → file picker opens → `handleRecipePhoto(file)`
2. `compressImage(file, 900, 0.72)` draws to canvas and exports as JPEG data URL
3. `setRecipePhotoPreview(dataUrl)` stores the URL in `window._pendingImageData` and shows the preview
4. "Add photo" and "Search web" buttons remain visible so the photo can be replaced at any time
5. "✕ Remove" clears `window._pendingImageData` via `removeRecipePhoto()`
6. `saveRecipe()` always reads `window._pendingImageData || null` — never the old `existing.imageData`

## Web image search flow

1. User clicks "🔍 Search web" in the edit form → `openImgSearch()` opens the search modal
2. User types a query and submits → `searchImages()` hits `api.openverse.org/v1/images/`
3. Results rendered as a grid of thumbnails (from `thumbnail` field, with `url` stored in `data-full`)
4. User taps a tile → `selectSearchImage(el)` fetches the full image via `fetchViaProxy()`
5. Image is compressed with `compressImage()` and passed to `setRecipePhotoPreview()`

## Firebase sync flow

1. User enters config in Settings → clicks "Connect Firebase" → `initFirebase(false)`
2. Firebase SDK + Auth imports dynamically; `onAuthStateChanged` listener registered
3. User taps "Sign in with Google" → popup → on success `onAuthStateChanged` fires with the user
4. `setupFirestoreForUser(user)` scopes Firestore to `users/{uid}/recipes`
5. If that collection is non-empty: remote recipes merged in (remote wins on ID conflicts)
6. If empty: local recipes pushed up (first sign-in migration)
7. `onSnapshot` listener keeps `recipes` in sync thereafter
8. Every `save()` call triggers `syncToFirebase()` which does `setDoc` for every recipe in `users/{uid}/recipes`
9. `deleteRecipe()` calls `deleteDoc` directly on `users/{uid}/recipes/{id}` instead of relying on push-all

**Firestore security rules** — wildcard covers both `recipes` and `meta/settings`:
```
match /users/{uid}/{document=**} {
  allow read, write: if request.auth != null && request.auth.uid == uid;
}
```

**Settings sync**: only `apiKey`, `geminiApiKey`, `ingTags`, and `methodTags` are synced — Firebase config fields are device-specific and never sent to Firestore.

## PWA / install flow

1. `sw.js` is registered in `index.html` via `navigator.serviceWorker.register('./sw.js')`
2. The service worker uses **network-first** for `index.html` (always picks up updates when online, falls back to cache offline)
3. The browser fires `beforeinstallprompt` when install criteria are met; the handler stores the event and shows the `#installBtn` (⬇) in the header
4. Tapping ⬇ calls `installApp()` which triggers `prompt()` on the stored event
5. After install the `appinstalled` event hides the button again
6. On iOS, install is via Safari's Share → Add to Home Screen (no `beforeinstallprompt` on Safari); the `apple-touch-icon` link in `<head>` controls the icon shown there

**Cache version:** `sw.js` uses network-first for HTML, so bumping the `CACHE` constant is only needed if `sw.js` or `manifest.json` itself changes.

## Common tasks

**Add a new default tag:** push to `DEFAULT_ING_TAGS` or `DEFAULT_METHOD_TAGS` constants. Existing users who already have saved settings won't automatically get new defaults — that's intentional (they may have deleted them).

**Add a new field to a recipe:** add the form element, read it in `saveRecipe()`, render it in `viewRecipe()`. The object is schema-less — just add the key.

**Change the AI model:** update the model string in `callAI()`.

**Change the AI prompt:** edit the `prompt` const in `analysePhoto()`. The model must return only a JSON object — preserve that constraint.
