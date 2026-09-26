# ReDiverse — supplementary results site

Five static pages — `index.html` (landing, no media) plus one per family — 224 MB on disk, but **5–40 KB of HTML
and ~0.3–0.5 MB of media to open any page**. No build step, no external assets, `.nojekyll` included.

```
cd site && git init && git add . && git commit -m "ReDiverse supplementary"
git branch -M main && git remote add origin git@github.com:<user>/<repo>.git && git push -u origin main
# Settings -> Pages -> Deploy from a branch -> main / (root)
```
Preview: `python3 -m http.server 8000 --directory site`.

## What is on it

39 prompt cards: **H3 (10) → Wan (10) → SDXL (10) → SD3 (9)**, videos first. Columns are the 4-step methods —
Student, DFD (video) or DP-DMD + Noise Optimization (image), and ReDiverse. **No teacher column.** Within a card
every column uses the same initial noises, so a difference is the method.

* videos: five clips **stacked vertically** per column, so a row across columns is one initial noise. Each clip
  ships twice — an embedded display rendition (≤672 px wide, CRF 30, ~120 KB) and the **original-resolution**
  file (H3 1344×768×124 frames, Wan 832×480×81, CRF 28) linked as "original resolution" under every column;
* images: nine seeds per method in a 3×3 grid, the same seeds in every column.

## Why it opens fast

Three things were slowing it down, fixed in order:

1. **240 HD clips autoplaying at once** starved the browser's video decoders, which is also what pushed playback
   below real time. Every `<video>` now has `preload=none`, and an `IntersectionObserver` starts only the card in
   view and pauses the rest.
2. **Posters were still eager** — a `poster=` attribute has no lazy mode, so ~7 MB of first frames downloaded
   before anything appeared. They are now `data-poster`, attached by the same observer.
3. **One page carried 924 media elements.** The site is now a text-only landing page plus one page per family,
   and cards use `content-visibility:auto`, so offscreen cards are not laid out at all.

Opening a page costs 5–40 KB of HTML plus the first card's posters or images (~0.3–0.5 MB); a card's clips
(~1.9 MB for 15) arrive as you reach it. Frame rates are untouched: H3 24 fps, Wan 16 fps.

## Where the prompts and samples come from

| family | source |
|---|---|
| H3 | `bundle/user_study_v2/selection.json` for cards 1, 2, 4, 6; cards **3, 5, 7, 8, 9, 10** are the user's picks from the new qualitative prompt set (`qualitative/prompt_benchmark`, sheets `h3_v00/v02/v05/v07/v10/v15`): hot air balloon, forest in autumn, candle flickering, cable car, violinist, penguin — all **ReDiverse-FT**, with the clips the user listed (their numbers are the sheet's columns counted from 1, i.e. files `c<n-1>`) |
| SDXL | `bundle/user_study_v2/selection.json` — 10 prompts, 9 seeds per method |
| SD3 | `bundle/user_study/selection.json` (v1) — 10 prompts, 9 seeds; the dog prompt is dropped because App. E's direction-source figure uses it |
| Wan | not in the user study: the dog prompt from `bundle/selected` (its own per-clip picks), then **9 prompts of the new qualitative prompt set** (sheets `wan_v02/v03/v04/v06/v13/v08/v14/v16/v18`), all **ReDiverse-FT**: clips of v04, v06, v13 are the user's; the other six were picked by eye for the most diverse ReDiverse-FT set with one subject per clip. Clip numbers are the sheet labels = file index (`WAN_BENCH` in `tools/make_site.py`) |

Prompts used by the paper's own figures are excluded: Wan 141 / 294, the custom H3 sets 902 / 903, SD3 27 and 6.

## The ReDiverse column is a per-sample mix

As assembled for the user study, each ReDiverse sample comes from one of the method's variants — **FT**
(fine-tuned adapter), **TF** (training-free directions on the student) or **FT+TF** — chosen per noise. The
variant of every sample is printed under the column (e.g. `FT / FT+TF · per sample: FT FT+TF FT FT FT`); every
other column is a single model.

## Rebuild

```
python tools/make_site.py --out site [--img-px 512 --vid-crf 28 --disp-px 672 --disp-crf 30]
```
Earlier generators: `tools/make_site_v1.py.bak` (rows, with teacher), `tools/make_site_v2.py.bak` (grids,
full-set metric ranking).
