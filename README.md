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
