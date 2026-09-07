# Bharat Connect — category launcher (1080 x 1920)

`index.html` is the expanded, interactive version of `img1.jpeg`, authored at exactly
1080 x 1920 and scaled to fit whatever viewport it opens in.
`img1_1080x1920.png` is a static export of the same screen.

Tap/click any of the 10 category tiles -> the mapped video streams full-screen from
Supabase Storage (the source videos are themselves 1080x1920, so they fill the canvas
exactly). Each video plays **once**, then returns to the menu.

A single **Back** button sits at the bottom right - dark glass over the video, flipped
to a light treatment on the branded surface so it reads against either background.
There is no progress bar and no other overlay. **Esc** also exits, and on the error
screen a tap anywhere goes back.

## Player states

All of them use the same gradient, palette and Bharat Connect lockup as the menu, so
the transition in and out is seamless.

| State | What shows |
|---|---|
| `booting`  | Branded surface: logo, category name, indeterminate blue->orange sweep bar |
| `needstap` | Same surface with a "Tap to play" button - only if the browser blocked autoplay |
| `loading`  | Mid-playback buffering: a small orange ring over the video, not the full surface |
| `failed`   | Same surface with "Video unavailable" and the object path that failed |
| (playing)  | Video full-bleed, Back the only thing drawn over it |

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

## Changing things

Everything configurable sits in one block near the bottom of `index.html`:

- `SUPABASE_URL` / `BUCKET` / `FOLDER` — where videos are read from
- `CATEGORIES` — the tile order, labels and video filenames
- `PUBLIC` — set to `false` and supply an **anon** key in `SUPABASE_KEY` to switch to
  short-lived signed URLs if the bucket is ever made private.
  (Do not ship a service_role key to a browser.)

## Scaling to the display

The stage is a fixed 1080 x 1920 box anchored to the viewport centre and scaled about
its own centre:

```js
stage.style.transform = "translate(-50%,-50%) scale(" + s + ")";
```

It fits by the smaller of the two ratios, so the design keeps its aspect and letterboxes
against the page background on any other viewport. Verified centred to the pixel at
1780x948, 1366x768, 900x1600, 1080x1920 and 600x400 - equal gaps on both sides in every
case. Do not reintroduce margin-based centring: combined with a centring parent it
offsets the stage twice and pushes it off-screen on anything smaller than 1080x1920.
