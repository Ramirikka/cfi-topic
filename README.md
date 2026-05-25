# Hot Topic — CFI Image Generator

Generates custom branded images at **690×254px** for Hot Topic use.

Live site: **https://cfihot.netlify.app**

---

## Repositories

| Purpose | Repo |
|---|---|
| Deployed dist (Netlify source) | `github.com/Ramirikka/cfi-topic` |
| Mirror / backup | `github.com/Ramirikka/dist` |

This repo contains **pre-built static files** — there is no build step.
The `assets/` folder holds the compiled JS and CSS bundles.

---

## Deployment (Netlify)

The site is hosted on Netlify under the project **cfihot**.

### Netlify Build Settings

| Setting | Value |
|---|---|
| Repository | `github.com/Ramirikka/cfi-topic` |
| Branch | `main` |
| Build command | *(empty — no build needed)* |
| Publish directory | `.` (repo root) |

> ⚠️ **Important:** The Publish directory must be set to `.` (a single dot), **not** `dist`.
> The repo root IS the dist — there is no subfolder.

These settings are also enforced via `netlify.toml` in the repo root:

```toml
[build]
  command = "echo 'skip'"
  publish = "."
```

### How to Deploy

Deploys trigger automatically when you push to `main` on `github.com/Ramirikka/cfi-topic`.

To push from local:
```bash
# Push to Netlify source repo
git push cfi-topic main

# Push to mirror
git push origin main
```

To manually trigger a deploy without a code change:
1. Go to **Netlify → cfihot → Deploys**
2. Click **Trigger deploy → Deploy site**

---

## Git Remotes

The local `dist` folder is linked to two remotes:

```bash
# Check remotes
git remote -v

# Expected output:
# cfi-topic   https://github.com/Ramirikka/cfi-topic.git (push)
# origin      https://github.com/Ramirikka/dist.git (push)
```

If you need to re-add the remotes:
```bash
git remote add origin https://github.com/Ramirikka/dist.git
git remote add cfi-topic https://github.com/Ramirikka/cfi-topic.git
```

---

## Gradient Overlay

The preview frame shows a **left-to-right brand gradient** (navy → transparent) over uploaded images.

### Background

The gradient was originally implemented in the React source but was removed from the compiled JS bundle at some point. It was restored as a CSS `::after` pseudo-element injected via a `<style>` tag in `index.html`.

### Why CSS `::after` instead of JS?

React controls the DOM children of `.preview-frame`. Any `<div>` injected by JavaScript gets removed when React re-renders (e.g. when a user uploads an image). CSS pseudo-elements exist outside React's control and persist through all re-renders.

### Gradient Specification

The gradient is defined in `index.html` `<style>`:

```css
.preview-frame::after {
  content: '';
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 2;
  background: linear-gradient(
    90deg,
    rgba(5,   0,  68, 0.80)   0%,   /* #050044 at 80% opacity */
    rgba(9,   0, 122, 0.51)  15%,
    rgba(9,   0, 130, 0.29)  30%,
    rgba(10,  0, 136, 0.13)  45%,
    rgba(10,  0, 140, 0.03)  60%,
    rgba(10,  0, 143, 0.00)  75%,
    rgba(10,  0, 143, 0.00) 100%    /* fully transparent */
  );
}
```

Colors are interpolated between brand colors **`#050044`** (deep navy) and **`#0A008F`** (indigo),
matching the gradient used in the **Newsletter Cover Image** project.

---

## OG / Social Share Preview

The share preview image is configured in `index.html`:

```html
<meta property="og:image" content="https://cfihot.netlify.app/CFIHot.jpg" />
<meta property="twitter:image" content="https://cfihot.netlify.app/CFIHot.jpg" />
```

> ⚠️ OG image URLs must be **absolute** (full `https://` URL).
> Relative paths like `./CFIHot.jpg` are not supported by social platforms.

To test the share preview: https://www.opengraph.xyz/url/https%3A%2F%2Fcfihot.netlify.app%2F

---

## Related Projects

| Project | Description |
|---|---|
| Newsletter Cover Image | 1920×1080 cover generator — source of the gradient logic |
| Hot Topic | This project — 690×254 image generator |

The Newsletter Cover Image project (`/Users/ramirikka/Newsletter Cover Image/dist`) retains the
original gradient implementation in its compiled JS bundle and was used as the reference
to restore the gradient here.

---

## PNG Export

Exported files are named `Hot-Topic-MMDD.png` (e.g. `Hot-Topic-0525.png`) and downloaded automatically.

### Gradient in the exported PNG

The compiled JS draws onto a canvas and calls `canvas.toBlob('image/png')` — CSS `::after` styles
are not visible on a canvas. To bake the gradient into the exported PNG, `index.html` patches
`HTMLCanvasElement.prototype.toBlob` to intercept the 690×254 export and draw the gradient
onto the canvas before encoding.

### Alpha channel fix (CMS compatibility)

Browser canvas always exports as **RGBA PNG** (with an alpha/transparency channel). Some CMSes
reject RGBA PNGs and only accept **RGB PNGs** (no alpha) — the same format Windows Paint produces.

**Fix:** The `toBlob` patch composites the canvas onto a fresh canvas with a solid black background
before encoding. This ensures every pixel is fully opaque, producing an RGB-equivalent PNG that
CMSes accept.

The full patch in `index.html`:

```js
(function () {
  var _toBlob = HTMLCanvasElement.prototype.toBlob;
  HTMLCanvasElement.prototype.toBlob = function (callback, type, quality) {
    if (this.width === 690 && this.height === 254) {
      var flat = document.createElement('canvas');
      flat.width = 690;
      flat.height = 254;
      var ctx = flat.getContext('2d');

      // Solid background — eliminates alpha channel
      ctx.fillStyle = '#000000';
      ctx.fillRect(0, 0, 690, 254);

      // Draw original canvas layers
      ctx.drawImage(this, 0, 0);

      // Draw brand gradient on top
      var grad = ctx.createLinearGradient(0, 0, 690, 0);
      grad.addColorStop(0.00, 'rgba(5,   0,  68, 0.80)');
      grad.addColorStop(0.15, 'rgba(9,   0, 122, 0.51)');
      grad.addColorStop(0.30, 'rgba(9,   0, 130, 0.29)');
      grad.addColorStop(0.45, 'rgba(10,  0, 136, 0.13)');
      grad.addColorStop(0.60, 'rgba(10,  0, 140, 0.03)');
      grad.addColorStop(0.75, 'rgba(10,  0, 143, 0.00)');
      grad.addColorStop(1.00, 'rgba(10,  0, 143, 0.00)');
      ctx.fillStyle = grad;
      ctx.fillRect(0, 0, 690, 254);

      return _toBlob.call(flat, callback, type, quality);
    }
    return _toBlob.call(this, callback, type, quality);
  };
})();
```
