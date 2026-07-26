# Mill & Table

A printable recipe collection. The web app is for finding and scaling;
the printed sheet is what you actually cook from.

## Deploy

Push to a GitHub repo, then **Settings → Pages → Deploy from branch → `main` / root**.
No build step, no dependencies.

## Add a recipe

1. Drop a `.json` file into `recipes/`.
2. Commit.

The included GitHub Action rebuilds `recipes/index.json` automatically.
If you'd rather not use the Action, delete `.github/` and add the filename
to `recipes/index.json` by hand.

Delete a recipe by deleting its file. Nothing else references it.

## Preview locally

Browsers block file reads from `file://`, so serve the folder:

```
python3 -m http.server
```

Then open `localhost:8000`.

## Recipe format

Only `id`, `title`, `ingredients`, and `steps` are required. Everything else
is optional and simply doesn't render when absent.

```json
{
  "id": "url-safe-slug",
  "title": "Recipe Name",
  "category": "breads",
  "tested": true,
  "source": "Grandma / adapted from X / generated",
  "yield": "1 loaf",
  "activeTime": "45 min",
  "totalTime": "18 hr",
  "berries": ["hard red winter"],
  "millSetting": "fine",
  "note": "One or two lines of orientation.",
  "ingredients": [
    { "qty": 400, "unit": "g", "item": "wheat berries, milled fine", "role": "flour" },
    { "qty": 340, "unit": "g", "item": "water", "role": "water" },
    { "qty": 9,   "unit": "g", "item": "salt", "note": "optional sub-line" }
  ],
  "steps": ["Step one.", "Step two."],
  "milling": "Notes specific to fresh-milled flour.",
  "substitute": "What to change if using store-bought."
}
```

Notes:

- `category` creates its own tab automatically. No code change needed.
- `role: "flour"` / `"water"` / `"levain"` drive the hydration math.
  Leave them off and the hydration field just doesn't appear.
- `role: "levain"` counts as both flour and water. Add `levainHydration`
  (default 100) and the app splits it correctly — 100 g of 100% starter
  is 50 g flour and 50 g water.
- `extraction` on a flour ingredient is the percentage that survives sifting.
  100 means whole and unsifted. 85 means you sifted out the coarse bran,
  so you have to mill ~18% more berries to end up with the same flour.
- `berry` on an ingredient overrides which grain it draws from. Omit it
  and the app uses the first entry in `berries`.
- `tested: false` prints an "untested" flag on the card and the sheet.
- Omit `unit` for countable things (3 garlic cloves).
- Weights are in grams because fresh-milled flour doesn't measure reliably by volume.

## In the kitchen

**Cook** opens one step at a time in large type and asks the browser to keep the
screen awake. Wake lock needs HTTPS, so it works on the live site but not over
plain `http://localhost`. Arrow keys move between steps, Escape exits.

**Mill & mix** shows berry weights to measure out — berry weight, not flour weight,
so sifted recipes correctly ask for more — and calculates water temperature.

The water temp formula multiplies your target dough temperature by the number of
temperature variables (flour, room, water, and levain when present), then subtracts
the ones you know. Fresh-milled flour is the reason this matters: it comes off the
mill 10-15°F above room temperature, and no standard recipe accounts for that.
Measure the flour rather than assuming.

## Printing

The Print button strips the interface and lays the recipe out as a letter sheet
with 0.5" margins — ingredients in a narrow left column, method in a wider right one.
Save as PDF from the browser's print dialog.

If a recipe won't fit one sheet, a warning appears above it with the overflow
percentage. That check measures the recipe at print dimensions in a hidden
container — close, but an estimate. Print one to confirm.

Test a sheet on your own printer before trusting the margins; Chrome and Safari
differ slightly at the edges.
