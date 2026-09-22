# How to Change Fonts

This guide covers the two text elements in the scrollytelling page:

1. **Text sections** — the narrative paragraphs between image groups
2. **Callouts** — the captions overlaid on images

---

## Where fonts are configured

| Purpose | File | Line | Variable |
|---------|------|------|----------|
| Register font files | `src/styles/font.css` | top | `@font-face` blocks |
| Define font stacks | `src/styles/app.css` | `:root` | `--font-sans`, `--font-serif` |
| Text section font | `src/components/Index.svelte` | `.full-bleed-text` | `var(--font-sans)` |
| Callout font | `src/components/Index.svelte` | `.callout-text` | `var(--font-serif)` |

---

## Step-by-step: change a font

### Step 1 — Register the font in `src/styles/font.css`

Add a `@font-face` block for each weight you need. Look for the `/* ADD FONTS HERE */` comment.

**Option A: Local font file** (recommended for full control)

1. Put your `.woff2` file(s) in `static/fonts/`
2. Add a block like this:

```css
@font-face {
	font-family: "My Font";
	src: url("/fonts/my-font-regular.woff2") format("woff2");
	font-weight: 400;
	font-style: normal;
	font-display: swap;
}
```

**Option B: Remote/Google Fonts URL**

```css
@font-face {
	font-family: "My Font";
	src: url("https://fonts.gstatic.com/s/myfont/v1/regular.woff2") format("woff2");
	font-weight: 400;
	font-style: normal;
	font-display: swap;
}
```

> Repeat for each weight (400, 500, 700, etc.). The `font-family` name must be identical across weights.

---

### Step 2 — Assign the font in `src/styles/app.css`

Look for the `/* CHANGE FONTS HERE */` comment in `:root`. Update the CSS variable for the font type you want:

```css
:root {
	--font-sans: "My Font", -apple-system, sans-serif;   /* used by text sections */
	--font-serif: "Another Font", Georgia, serif;         /* used by callouts */
}
```

The first quoted name must match the `font-family` from your `@font-face` block. The fallbacks after it apply if the font fails to load.

---

### Step 3 — No changes needed in `Index.svelte` (unless swapping which variable is used)

The text sections and callouts already reference these variables:

```css
/* Text sections — .full-bleed-text */
font-family: var(--font-sans);   /* change to var(--font-serif) if you want */

/* Callouts — .callout-text */
font-family: var(--font-serif);  /* change to var(--font-sans) if you want */
```

To swap which variable an element uses, edit the `font-family` line in `Index.svelte`:

| Element | CSS class in `Index.svelte` | Currently uses |
|---------|----------------------------|----------------|
| Text sections | `.full-bleed-text` | `var(--font-sans)` |
| Callouts | `.callout-text` | `var(--font-serif)` |

---

## Quick reference: current setup

| Element | Variable | Font name | Registered in |
|---------|----------|-----------|---------------|
| Text sections | `--font-sans` | "Rubik" | `font.css` |
| Callouts | `--font-serif` | "Tiempos Text" | `font.css` |

---

## Example: swap callout font to Rubik

1. Open `src/components/Index.svelte`
2. Find `.callout-text` (look for `/* CALLOUTS */`)
3. Change:
   ```css
   font-family: var(--font-serif);
   ```
   to:
   ```css
   font-family: var(--font-sans);
   ```

## Example: add a brand new font

1. Drop `myfont-regular.woff2` and `myfont-bold.woff2` into `static/fonts/`
2. In `src/styles/font.css`, add two `@font-face` blocks (both with `font-family: "My Font"`, weights 400 and 700)
3. In `src/styles/app.css`, set `--font-sans: "My Font", sans-serif;`
4. Done — text sections now use "My Font"
