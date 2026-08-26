# Lost Patrol — WordPress embed instructions

One file: `lost-patrol-scenario.html`. No build step, no plugin, no cookies, no tracking, no browser storage. It can't break your theme and your theme can't break it.

**One external request:** Google Fonts, for Alfa Slab One and Source Sans 3. To drop it, delete the two `<link>` tags near the top of the file — system-font fallbacks are already in the CSS and everything still lays out correctly, it just looks less designed.

## 1. Upload the file

The WordPress Media Library blocks `.html` uploads by default, so use one of these:

**Option A — SFTP / cPanel File Manager (recommended)**
Upload to a folder outside the theme so a theme update never wipes it:

```
/wp-content/uploads/interactives/lost-patrol-scenario.html
```

It's then reachable at `https://yoursite.org/wp-content/uploads/interactives/lost-patrol-scenario.html`. Open that URL directly and click through a path before you embed it.

**Option B — allow HTML uploads**
Add to your child theme's `functions.php`:

```php
add_filter('upload_mimes', function ($mimes) {
    $mimes['html'] = 'text/html';
    return $mimes;
});
```

Only do this if the media library is restricted to trusted editors.

## 2. Embed it in the article

In the post editor, add a **Custom HTML** block:

```html
<iframe
  id="lost-patrol"
  src="/wp-content/uploads/interactives/lost-patrol-scenario.html"
  title="Lost Patrol: an interactive survival scenario"
  loading="lazy"
  style="width:100%; border:0; height:1000px; display:block;"
  scrolling="no"></iframe>

<script>
(function () {
  var frame = document.getElementById('lost-patrol');
  window.addEventListener('message', function (e) {
    if (!e.data || e.data.type !== 'lostPatrolHeight') return;
    if (frame.contentWindow !== e.source) return;
    frame.style.height = (e.data.height + 24) + 'px';
  });
})();
</script>
```

The `height:1000px` is only a fallback for the moment before the first height message arrives. The script resizes the iframe as the reader moves between screens, so there's never a scrollbar inside the frame.

If your host strips `<script>` from post content, use the iframe alone and raise the fixed height to about `2200px` — the ending screen with the trail map is the tall one. It works, there's just empty space on shorter screens.

**Same-origin note:** the resize script only works if the HTML file is served from the same domain as the article. On a subdomain or CDN, use the fixed height.

## 3. What the reader experiences

1. **One STOP screen.** A red octagonal stop sign, an illustration slot, and the four steps — Stop, Think, Observe, Plan — as four compact rows. One button into the scenario. The sign makes the point that you come to a full stop rather than rolling through.
2. **The scenario, one screen at a time.** Prior decisions collapse into small breadcrumb chips. No map, no overview — the reader only sees where they are, which is the point.
3. **The status panel.** Eight tiles above the story. The daylight tile draws a sun sinking along an arc with the sky shifting blue → sunset → night as time runs out. The others are gear and condition: patrol together or split, clothing dry or soaked, morale (a four-bar meter), injuries, shelter, signal, fire. Grey tiles light up in color as the patrol earns them, so "we have a fire now" is visible rather than stated.
4. **An ending**, badged *Everyone gets home*, *You make it — the hard way*, or *This goes wrong*, with the article's survival tip attached.
5. **The trail map.** Revealed only after an ending — see below.
6. **Back up one decision**, so a reader who hits a bad ending can retry that one choice without replaying from the top.

On phones the status panel collapses to a tighter two-row grid of smaller tiles so the story text stays near the top of the screen.

## 4. The trail map

A drawn map: ridge line, river, forest, compass, legend. Numbered teal pins are decision points; ending markers are green (safe), gold (safe the hard way), or orange (goes wrong). Your route is the bold gold trail; everything else is faint dotted line.

- **Hover or focus** a marker to preview that scene in the panel below the map — its outcome tag, and the lesson attached to it.
- **Click or tap** to lock that selection and reveal a jump button: *Play from this point* on a decision, *Replay the decision that led here* on an ending. It rebuilds daylight and every condition exactly as they'd have been on that route, so a reader can explore all ten endings without restarting each time.
- **Keyboard:** every marker is tabbable; Enter or Space acts as a tap.

It's deliberately withheld during play. A reader who can see the map isn't lost, which would undercut the whole lesson.

**Swapping in a hand-drawn map later.** Near the top of the script:

```js
var MAP_BACKGROUND = null;   // e.g. '/wp-content/uploads/trail-map.jpg'
```

Point it at an image and the drawn terrain, legend, and compass are skipped — your artwork sits underneath, with the trail lines, pins, and labels drawn on top. Design the image at **1060 × 700** and the existing pin coordinates will land sensibly; if you need to nudge them, the layout constants are right above it:

```js
var COL_X = [95, 300, 520, 700];   // x of each decision column
var ROW_TOP = 116, ROW_H = 56;     // vertical spacing
```

## 5. Adding illustrations

Search the file for `art:`. Every scene has a slot:

```js
art: { slot:'river-crossing', note:'Rocky river crossing at dusk, two Scouts soaked on the bank' },
```

Add a `src` and an `alt` and that scene renders the image instead of the placeholder:

```js
art: { slot:'river-crossing',
       note:'Rocky river crossing at dusk, two Scouts soaked on the bank',
       src:'/wp-content/uploads/2026/09/river-crossing.jpg',
       alt:'Two Scouts on slippery rocks in a shallow river',
       credit:'Illustration by …' },
```

`credit` is optional and renders as a caption. Fill in one slot or all nineteen; anything without a `src` keeps its placeholder.

Target size: **1200 × 525 px** (placeholders are 16:7). Images are `width:100%`, so anything wider scales down.

To hide the empty placeholders before art is ready:

```js
var SHOW_ART_PLACEHOLDERS = false;
```

The nineteen slots: `stop-hero`, `opening-woods`, `backtrack`, `found-trail`, `river-crossing`, `night-fire`, `hypothermia`, `campsite`, `poncho-signal`, `argument`, `second-day`, `scattered`, `running`, `twisted-ankle`, `recovered-camp`, `split-group`, `deeper-woods`, `late-camp`, `lost-dark`.

## 6. Meeting Mode

The button in the top right. Off by default for web readers; it unlocks three things for leaders:

- Type scales up about 40% and the layout widens, so it reads across a room on a projector or TV.
- Choices get big **A / B / C** keys for a show-of-hands vote. The presenter can press A, B, or C (or 1, 2, 3) instead of clicking.
- A dashed **Talk it over** panel appears on every screen with two discussion questions that go past the survival tip.

Per-session and resets on reload — nothing is stored on the reader's device.

## 7. What changed from the article draft

- **Path 3 got a recovery branch.** In the draft, running ended in two failures with no way out. Now both the twisted ankle and the deeper-woods node offer "stop, treat, signal, stay together" as a route to rescue. A reader who panicked in the first hour can still get the patrol home — the most useful thing the activity teaches.
- **Path 1-B and Path 2-A also continue.** Falling in the river leads to a dry-clothes-and-fire choice (your hypothermia caption, made into a decision). The camp argument leads to a morning choice: regroup, or scatter and search alone.
- **Ten endings:** four good, two "safe, the hard way," four bad.
- **Daylight is load-bearing.** All four bad endings run the clock to zero. Every good ending still has 30 minutes to 1h15m of light left when the patrol stopped moving and started working, and the ending screen says so outright. Running costs 1h10m; making camp costs 30 minutes.

## 8. Accessibility and privacy

- Keyboard operable throughout; focus moves to the new screen's first choice after each decision. Map pins are focusable with `aria-label`s naming the decision or ending.
- Status is never conveyed by color alone — every tile and badge is labeled in text, and endings carry a check or X mark as well as a color.
- `aria-live` on the stage so screen readers announce each new screen.
- Honors `prefers-reduced-motion`.
- No storage, no cookies, no data leaves the reader's browser. The only third-party request is the Google Fonts stylesheet; delete the two `<link>` tags to make the file fully self-contained.

If you later want to know which mistakes readers actually make, that's a small addition — a `gtag`/Plausible event on each choice. Keep it aggregate and cookieless.
