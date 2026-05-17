# The Dog-Eared Page

**[→ Open live app](https://neil293.github.io/The-Dog-Eared-Page/)**

A personal recipe PWA — single-file, no build tools, installable on mobile.

## Features

- **Recipe cards** with prep time, cook time, and serves
- **All ingredients in grams** — consistent across the whole app
- **Ingredient tags** (Beef, Chicken, Pork, Lamb, Seafood, Fish, Vegetarian, Vegan, Pasta, Rice, Eggs, Dairy)
- **Method tags** (Soup, Slow Cook, Bake, Roast, Fry, Grill, Stir-fry, Steam, Salad, Sauce, Dessert, Breakfast, BBQ)
- **Collapsible filter sections** — ingredient and method filters collapse/expand to save screen space on mobile
- **Live search** across recipe name, ingredient names, and tags
- **Simultaneous tag filtering** — ingredient and method filters stack
- **Add / edit recipes** with a form that supports custom tags
- **Recipe photos** — add a photo to any recipe from your camera roll or by searching the web; images are compressed browser-side (max 900 px wide, 72 % JPEG) before storage
- **Web image search** — searches the [Openverse](https://openverse.org) catalogue (no API key required); tap a result to set it as the recipe photo
- **Favourites tab**
- **📷 AI recipe import** — photograph a recipe page, AI reads it and converts all measurements to grams, pre-fills the editor
- **Firebase Firestore sync** — real-time sync across devices with Google Sign-In; recipes, AI keys, and custom tags are all stored privately per account — sign in on a new device and everything loads automatically
- **Export / Import** JSON backup
- **Installable PWA** — service worker for offline use; ⬇ install button appears automatically in browsers that support it (Chrome, Edge, Android)

## Stack

| Layer | Choice |
|---|---|
| Frontend | Vanilla HTML / CSS / JS — single `index.html` |
| AI (recipe import) | Anthropic Claude API (`claude-opus-4-20250514`) **or** Google Gemini API — called directly from the browser |
| Image search | Openverse API (`api.openverse.org/v1/images/`) — free, no key required |
| Sync | Firebase Firestore 10.x, loaded dynamically via ESM import |
| Fonts | Playfair Display · Lora · Caveat (Google Fonts) |
| Hosting | GitHub Pages |

No build step. No bundler. No dependencies to install.

## Getting started

1. Open `index.html` in a browser, or visit the GitHub Pages URL.
2. Go to **Settings ⚙** and enter:
   - **Anthropic API key** — for AI photo import (`sk-ant-…`), **or** a **Gemini API key** (free tier available)
   - **Firebase config** — API key, Project ID, App ID — for cross-device sync (optional)
3. Add recipes manually, tap **📷** to import from a photo, or use **🔍 Search web** inside the editor to find a photo.

### AI photo import

The app supports two AI providers for the recipe import feature. Configure one in Settings:

| Provider | Key format | Notes |
|---|---|---|
| Anthropic Claude | `sk-ant-…` | Best accuracy |
| Google Gemini | `AIza…` | Free tier available at [aistudio.google.com](https://aistudio.google.com) |

### Firebase + Google Sign-In setup (optional)

1. Create a project at [console.firebase.google.com](https://console.firebase.google.com)
2. Add a **Web app** and copy the config values
3. Enable **Firestore Database** in the console
4. Enable **Authentication → Sign-in method → Google**
5. Set Firestore security rules:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{uid}/{document=**} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```
6. Paste API key, Project ID, and App ID into Settings → Firebase Sync → Connect
7. Tap **Sign in with Google** — your recipes, AI keys, and custom tags sync to your account and load automatically on any device you sign in on

### Storage and Firebase free tier

Recipe photos are compressed to roughly 80–150 KB each before being stored in Firestore. The Spark (free) plan includes 1 GiB of storage, which comfortably holds hundreds of photo recipes.

## Files

```
index.html      # Entire application
manifest.json   # PWA manifest
sw.js           # Service worker — offline caching, enables install
icon.svg        # App icon
```

## Local development

No tooling required — open `index.html` directly, or serve it with any static file server:

```bash
npx serve .
# or
python3 -m http.server
```
