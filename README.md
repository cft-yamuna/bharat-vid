# Bharat Connect — category launcher (1080 x 1920)

`index.html` is the interactive version of the menu screen, authored at exactly
1080 x 1920 and scaled to fit whatever viewport it opens in.

The menu itself is **`new.png` drawn at its native 1080 x 1920** — the supplied
artwork, unaltered — with ten transparent hit targets laid over the tiles. A
render of `index.html` is pixel-identical to `new.png` (0 of 2,073,600 pixels
differ), and it returns to exactly that state after every video.

Tap/click any of the 10 category tiles -> the mapped video streams full-screen from
Supabase Storage (the source videos are themselves 1080x1920, so they fill the canvas
exactly). Each video plays **once**, then returns to the menu.

A single **Back** button sits at the bottom right - dark glass over the video, flipped
to a light treatment on the branded surface so it reads against either background.
**Esc** also exits, and on the error screen a tap anywhere goes back.

## Why the artwork is used directly

The earlier build redrew the whole screen — phone mock, status bar, icons, logo — in
CSS and SVG, traced from a 430 x 852 JPEG. That trace could only ever approximate the
original: the logo mark was wrong, and the photographic phone render is not something
CSS can reproduce. `new.png` arrives at the exact stage resolution, so it is used as
the plate instead. The design is now the client's asset byte for byte, there is no
second copy of the layout to keep in sync, and no font has to load for the menu to be
correct.

Only the chrome that never appears in the artwork is drawn in CSS: the loading
surface, the Back pill, the buffering ring and the error screen.

## Tile hit targets

Measured off `new.png`. Three columns pitched 200px apart, centred on the phone
screen (x 233..859); four rows band-fitted to the icon + label clusters. Both live in
`COLX` / `ROWY` near the bottom of `index.html`, and each button is inset 6px from its
cell so neighbouring tiles never touch.

| | column x (left edge) | | row y (top, height) |
|---|---|---|---|
| col 1 | 246 | row 1 | 572, 234 |
| col 2 | 446 | row 2 | 806, 224 |
| col 3 | 646 | row 3 | 1030, 195 |
| width | 200 | row 4 | 1225, 209 |

Hover paints a soft blue rounded highlight over the tile; pressing deepens it, adds a
hairline ring and scales it down slightly.

## Player states

| State | What shows |
|---|---|
| `booting`  | Frosted menu: Bharat Connect lockup, category name, blue->orange sweep bar |
| `needstap` | Same surface with a "Tap to play" button - only if the browser blocked autoplay |
| `loading`  | Mid-playback buffering: a small orange ring over the video, not the full surface |
| `failed`   | Same surface with "Video unavailable" and the object path that failed |
| `playing`  | Video full-bleed, Back the only thing drawn over it |

The loading surface **paints no background of its own**. It frosts whatever is behind
it with `backdrop-filter`, and while booting that is the artwork — the video is still
at `opacity:0`. So the menu -> loader -> video hand-off is seamless by construction,
with no gradient to keep in sync with the plate.

`booting` clears on the video's first `playing` event; a blocked autoplay is treated as
`needstap`, never as a failure. Genuine load failures come only from the `<video>`
element's own `error` event.

For the default public-bucket path the URL is built synchronously so `vid.play()` still
runs inside the tap's user-activation window - an async hop there is what makes browsers
reject playback and fall back to the tap button.

## Video source

Bucket `npci` (public), folder `bh_videos`:

```
https://ozkbnimjuhaweigscdby.supabase.co/storage/v1/object/public/npci/bh_videos/<file>
```

Nothing is loaded from disk — `preload="none"`, so a video is fetched only on tap.

## Tile -> video mapping

Unchanged from the previous build.

| Tile            | Object in bh_videos/ | Original file      |
|-----------------|----------------------|--------------------|
| Education Fees  | education.mp4        | Education.mp4      |
| Electricity     | electricity.mp4      | Electricity.mp4    |
| E-challan       | e-challan.mp4        | e-Challan.mp4      |
| Loan repayment  | loan.mp4             | Loan.mp4           |
| FASTag          | fastag.mp4           | FASTag.mp4         |
| LPG             | lpg.mp4              | LPG.mp4            |
| NPS             | ncmc.mp4             | NCMC.mp4  *(see note)* |
| Piped gas       | piped-gas.mp4        | Pipe gas.mp4       |
| EV Recharge     | ev-recharge.mp4      | EV.mp4             |
| Forex           | forex.mp4            | Forex.mp4          |

> **Note on NPS:** the screen has an *NPS* tile but there is no `NPS.mp4`; the folder has an
> `NCMC.mp4` with no matching tile. Every other tile paired 1:1, so NPS is wired to `ncmc.mp4`
> as the only remaining candidate. If that is wrong, change the `file` for `id:"nps"` in the
> `CATEGORIES` array in `index.html`, or upload a real `nps.mp4` and point it there.

`CATEGORIES` is in artwork order — left to right, top to bottom — and each entry is
placed into the grid by its index, so **reordering the array moves the hit targets**.
Keep it in artwork order and change only `file`.

## Files

| File | What it is |
|---|---|
| `index.html`          | The launcher |
| `new.png`             | The menu artwork, 1080 x 1920. Required at runtime |
| `bc-lockup.png`       | Bharat Connect lockup for the loading surface, cut from `new.png` and un-matted off its white background |
| `img1.jpeg`           | The original 430 x 852 reference. Superseded by `new.png` |
| `img1_1080x1920.png`  | Export of the previous CSS-drawn build. Superseded by `new.png` |

Brand values sampled from the artwork: mark `#185EB2`, wordmark `#FD5F00`, headline
navy `#20457B`, "POWERED BY" grey-blue `#4D6A95`.

## Changing things

Everything configurable sits in one block near the bottom of `index.html`:

- `SUPABASE_URL` / `BUCKET` / `FOLDER` — where videos are read from
- `CATEGORIES` — the tile labels and video filenames (order is load-bearing, see above)
- `COLX` / `TILE_W` / `ROWY` — hit-target geometry. Only needs touching if the artwork changes
- `PUBLIC` — set to `false` and supply an **anon** key in `SUPABASE_KEY` to switch to
  short-lived signed URLs if the bucket is ever made private.
  (Do not ship a service_role key to a browser.)

If the artwork is replaced, re-measure `COLX` / `ROWY` against the new file — nothing
else in the page depends on the layout.

## Scaling to the display

The stage is a fixed 1080 x 1920 box anchored to the viewport centre and scaled about
its own centre:

```js
stage.style.transform = "translate(-50%,-50%) scale(" + s + ")";
```

It fits by the smaller of the two ratios, so the design keeps its aspect and letterboxes
against the page background on any other viewport. At a native 1080 x 1920 display the
scale is exactly 1 and the artwork is pixel-for-pixel. Do not reintroduce margin-based
centring: combined with a centring parent it offsets the stage twice and pushes it
off-screen on anything smaller than 1080x1920.
