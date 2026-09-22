# Grey Wipe Effect

A diagonal hard-wipe that desaturates the background image from right to left as the user scrolls through the vertical section (images 26–29).

## How it works

1. Two copies of `background.webp` are stacked in `.static-background`
2. The bottom copy is the original (full color)
3. The top copy (`.bg-grey`) has `filter: saturate(0)` applied — removing saturation without darkening
4. A `clip-path: polygon()` on `.bg-grey` creates the visible area, producing a hard diagonal wipe edge

## Key locations in `Index.svelte`

| What | Line | Search keyword |
|------|------|----------------|
| State variable | 191 | `bgGreyProgress` |
| Clip-path derived variable | 193–197 | `greyClipPath` |
| Scroll progress calculation | 254–265 | `vertical-stack-section` |
| Grey overlay `<img>` element | 384 | `bg-grey` |
| `.bg-grey` CSS class | 583–592 | `.bg-grey` |

## Adjusting the diagonal angle

The polygon formula at line 196:

```
polygon(${130 - bgGreyProgress * 130}% 0%, 100% 0%, 100% 100%, ${100 - bgGreyProgress * 130}% 100%)
```

- `130` = `100 + TILT` where **TILT = 30** is the diagonal offset in percent
- The top edge of the wipe leads the bottom edge by `TILT`% (bottom-left to top-right diagonal)
- **Steeper diagonal**: increase TILT (e.g. use `160` and `160` instead of `130`)
- **Shallower / more vertical**: decrease TILT (e.g. use `110` and `110` instead of `130`)
- **Vertical wipe (no diagonal)**: set both to `100` (i.e. `100 - bgGreyProgress * 100`)

## Adjusting when the wipe triggers

In the scroll handler (lines 254–265), `bgGreyProgress` is calculated based on the `.vertical-stack-section` position:

```js
const verticalEl = containerEl.querySelector(".vertical-stack-section");
if (verticalEl) {
    const rect = verticalEl.getBoundingClientRect();
    const vh = window.innerHeight;
    const items = verticalEl.querySelectorAll(".v-stack-item");
    const numImages = items.length || 4;
    const totalH = numImages * vh;
    const startTop = 0;
    const endTop = -(totalH);
    const range = startTop - endTop;
    bgGreyProgress = Math.max(0, Math.min(1, (startTop - rect.top) / range));
}
```

- `startTop = 0` means the wipe starts when the section top reaches the viewport top (image 27 entering at bottom)
- `endTop = -(totalH)` means the wipe ends when the section has fully scrolled past (text section after 29 is visible)
- Change `startTop` and `endTop` to shift when the wipe begins/ends

## Adjusting desaturation vs darkening

The `.bg-grey` CSS class uses `filter: saturate(0)` — this removes color without changing brightness. Alternatives:

- `filter: grayscale(1)` — luminance-based grayscale (slightly different brightness mapping)
- `filter: saturate(0) brightness(0.8)` — desaturate + darken

## Reversing the wipe direction

To wipe left-to-right instead, swap the polygon to clip from the left side:

```
polygon(0% 0%, ${bgGreyProgress * 130}% 0%, ${bgGreyProgress * 130 - 30}% 100%, 0% 100%)
```

---

## Adding the grey wipe to a new project — step by step

### 1. HTML — stack two copies of the background image

Place two `<img>` tags inside your fixed background container. The second one gets the `bg-grey` class and a dynamic `clip-path` style:

```svelte
<div class="static-background">
    <img src="images/background.webp" alt="" />
    <img src="images/background.webp" alt="" class="bg-grey" style="clip-path:{greyClipPath};" />
</div>
```

### 2. Script — add the state and derived variable

Add these in your `<script>` block, alongside your other state variables:

```js
let bgGreyProgress = $state(0);

let greyClipPath = $derived(
    bgGreyProgress <= 0
        ? "inset(0 100% 0 0)"
        : `polygon(${130 - bgGreyProgress * 130}% 0%, 100% 0%, 100% 100%, ${100 - bgGreyProgress * 130}% 100%)`
);
```

- `bgGreyProgress` ranges from `0` (no grey visible) to `1` (fully grey)
- `greyClipPath` converts that into a `clip-path` string
- `130` = `100 + 30` where `30` is the diagonal tilt — adjust to change the angle

### 3. Script — update `bgGreyProgress` on scroll

In your scroll handler, calculate how far the target section has scrolled and map it to 0–1. The example below uses a vertical stack section, but you can target any element:

```js
const targetEl = containerEl.querySelector(".your-section-class");
if (targetEl) {
    const rect = targetEl.getBoundingClientRect();
    const vh = window.innerHeight;
    // Customize these based on when you want the wipe to start/end:
    const startTop = 0;           // wipe starts when section top hits viewport top
    const endTop = -(rect.height); // wipe ends when section has fully scrolled past
    const range = startTop - endTop;
    bgGreyProgress = Math.max(0, Math.min(1, (startTop - rect.top) / range));
}
```

- `startTop` — the `rect.top` value at which the wipe should begin (`0` = section top at viewport top)
- `endTop` — the `rect.top` value at which the wipe should be complete (`-(totalH)` = section fully scrolled past)
- Adjust these values to change the trigger scroll positions

### 4. CSS — style the grey overlay

Add this class in your `<style>` block:

```css
.bg-grey {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    object-position: center;
    filter: saturate(0);
    pointer-events: none;
}
```

- `position: absolute; inset: 0` — overlays the original image exactly
- `filter: saturate(0)` — desaturates without darkening
- `pointer-events: none` — prevents the overlay from capturing clicks

### 5. That's it

No animation keyframes needed — the `clip-path` is driven entirely by scroll position via `bgGreyProgress`.

