# The Recipe Book — Setup Guide

## Files
- `index.html` — the entire app
- `manifest.json` — PWA install support

---

## 1. GitHub Pages Deployment

1. Create a new GitHub repo (e.g. `recipe-book`)
2. Upload both files to the repo root
3. Go to **Settings → Pages → Source: Deploy from branch → main → / (root)**
4. Your app will be live at: `https://yourusername.github.io/recipe-book/`

---

## 2. Firebase Sync Setup (free)

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. **Create project** → give it a name → Continue
3. **Build → Firestore Database → Create database** → Start in test mode → Choose region → Done
4. **Project settings (gear icon) → Your apps → Web app (</> icon)**
5. Register app, copy the `firebaseConfig` values:
   - `apiKey` → paste into app Settings → Firebase API Key
   - `projectId` → paste into app Settings → Firebase Project ID
   - `appId` → paste into app Settings → Firebase App ID
6. In the app, go to ⚙ Settings → Firebase section → **Connect Firebase**
7. Sync dot (top right) turns **green** when connected ✓

> **Security note:** Before going public, set up Firestore Security Rules in Firebase Console to restrict access.

---

## 3. AI Photo Import Setup

1. Get an Anthropic API key from [console.anthropic.com](https://console.anthropic.com)
2. In the app: **⚙ Settings → Anthropic API Key** → paste key → Save
3. Use the **📷 button** in the header to import a recipe photo

The AI will:
- Read all text from the photo
- Convert ALL measurements to grams automatically
- Tag the recipe (Beef, Slow Cook, etc.)
- Pre-fill the recipe editor — you just review and save

---

## 4. Features Summary

| Feature | How to use |
|---|---|
| Add recipe | **＋** button top right |
| Photo import | **📷** button → snap or upload → Analyse |
| Filter by ingredient | Tap tags (Beef, Chicken…) under search |
| Filter by method | Tap tags (Soup, Slow Cook…) under search |
| Search | Type in search bar — searches name, ingredients, tags |
| Favourites | Open recipe → ☆ Favourite, then see in Favourites tab |
| Edit recipe | Open recipe → ✏ Edit |
| Export all | ⚙ Settings → Export JSON |
| Import backup | ⚙ Settings → Import JSON |
| Install as app | Browser → Share → Add to Home Screen (iOS/Android) |

---

## 5. Tag Customisation

Default **ingredient tags**: Beef, Chicken, Pork, Lamb, Seafood, Fish, Vegetarian, Vegan, Pasta, Rice, Eggs, Dairy

Default **method tags**: Soup, Slow Cook, Bake, Roast, Fry, Grill, Stir-fry, Steam, Salad, Sauce, Dessert, Breakfast, BBQ

Add custom tags directly in the recipe editor (type in the custom tag box below the tag grid).

---

## Gram Conversion Reference (used by AI)

| Unit | Grams |
|---|---|
| 1 cup flour | 125g |
| 1 cup sugar | 200g |
| 1 cup butter | 225g |
| 1 cup water/milk | 240g |
| 1 tbsp | ~15g |
| 1 tsp | ~5g |
| 1 oz | 28g |
| 1 lb | 454g |
