# The Recipe Book

**[→ Open live app](https://neil293.github.io/the-dog-eared-page/)**

A personal recipe PWA — single-file, no build tools, installable on mobile.

## Features

- **Recipe cards** with prep time, cook time, and serves
- **All ingredients in grams** — consistent across the whole app
- **Ingredient tags** (Beef, Chicken, Pork, Lamb, Seafood, Fish, Vegetarian, Vegan, Pasta, Rice, Eggs, Dairy)
- **Method tags** (Soup, Slow Cook, Bake, Roast, Fry, Grill, Stir-fry, Steam, Salad, Sauce, Dessert, Breakfast, BBQ)
- **Live search** across recipe name, ingredient names, and tags
- **Simultaneous tag filtering** — ingredient and method filters stack
- **Add / edit recipes** with a form that supports custom tags
- **Favourites tab**
- **📷 Photo import** — photograph a recipe page, Claude AI reads it and converts all measurements to grams, pre-fills the editor
- **Firebase Firestore sync** — real-time sync across devices, connection dot in the header
- **Export / Import** JSON backup
- **Installable PWA** via `manifest.json`

## Stack

| Layer | Choice |
|---|---|
| Frontend | Vanilla HTML / CSS / JS — single `index.html` |
| AI | Anthropic Claude API (`claude-opus-4-20250514`), called directly from the browser |
| Sync | Firebase Firestore 10.x, loaded dynamically via ESM import |
| Fonts | Playfair Display · Lora · Caveat (Google Fonts) |
| Hosting | GitHub Pages |

No build step. No bundler. No dependencies to install.

## Getting started

1. Open `index.html` in a browser, or visit the GitHub Pages URL.
2. Go to **Settings ⚙** and enter:
   - **Anthropic API key** — for photo import (`sk-ant-…`)
   - **Firebase config** — API key, Project ID, App ID — for cross-device sync
3. Add recipes manually or tap **📷** to import from a photo.

### Firebase setup (optional)

1. Create a project at [console.firebase.google.com](https://console.firebase.google.com)
2. Add a Web app and copy the config values
3. Enable **Firestore Database** in the console
4. Paste API key, Project ID, and App ID into Settings → Firebase Sync → Connect

## Files

```
index.html      # Entire application
manifest.json   # PWA manifest
```

## Local development

No tooling required — open `index.html` directly, or serve it with any static file server:

```bash
npx serve .
# or
python3 -m http.server
```
