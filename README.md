# Nival - Cyberspace Identity

Nival is a single-page cyberspace identity experience. It presents a visual identity studio with a fixed, cinematic hero, a cursor-following spotlight reveal, an avatar gallery, lightweight interactions, and supporting Create, Pricing, About, and Contact sections.

The page is intentionally delivered as one standalone HTML file. There is no framework, build step, package manager, local asset, or backend dependency.

## Quick Start

Open [index.html](index.html) directly in a browser.

For a local HTTP server, run one of these commands from the `Nival` folder:

```bash
python -m http.server 8000
```

Then open `http://localhost:8000`.

The remote images and Google Font require an internet connection. Opening the file directly still works for the HTML, CSS, and JavaScript, but browser privacy or network settings may affect remote assets.

## Technology

- Semantic HTML
- Inline CSS
- Vanilla JavaScript in an IIFE
- CSS Grid and absolute positioning
- CSS transforms and container queries
- Web Animations API for the intro sequence
- CSS masks and a small canvas used as the spotlight mask source
- Google Fonts: Inter 400, 500, and 700
- No external JavaScript or CSS libraries

## Page Structure

The document has two visual layers:

1. **Hero stage**
   - A fixed `1536 x 1024` design canvas on desktop and landscape layouts.
   - The stage is scaled with `transform: scale(...)` to fit the viewport.
   - The hero remains in the first viewport while the rest of the page scrolls underneath it.
   - The stage contains the navigation, headline, identity card, avatar gallery, material control, social controls, and android artwork.

2. **Supporting sections**
   - `#create`: identity creation call to action.
   - `#pricing`: Signal, Studio, and Collective plans.
   - `#about`: product positioning and studio contact.
   - `#contact`: email contact anchor inside the About section.

On real portrait phones, the fixed stage is replaced by a natural CSS Grid layout so the content can scroll normally and remain readable.

## Asset Sources

All image elements start with an empty `src` in the markup. JavaScript assigns the URLs at runtime from the `assets` object.

### Hero assets

- Front android:
  `https://d2ol7oe51mr4n9.cloudfront.net/user_38xzZboKViGWJOttwIXH07lWA1P/658f62c6-fdb2-41b9-aa0f-10837f9e72cb.png`
- Back armor variant:
  `https://d8j0ntlcm91z4.cloudfront.net/user_38xzZboKViGWJOttwIXH07lWA1P/hf_20260914_123603_5ac5732a-900b-4919-9d9c-3c454194b0a5.png`
- Material thumbnail:
  `https://d2ol7oe51mr4n9.cloudfront.net/user_38xzZboKViGWJOttwIXH07lWA1P/96ef2bcd-ed69-4b7e-ab83-8a18d2129d99.png`

### Avatar gallery

The five gallery images are stored in `assets.avatars` in display order:

1. `caf08036-c63c-4d0c-9b81-43c5a7a49343.png`
2. `599a5a28-c9f1-44f9-b01b-adb8b22e8c12.png`
3. `fc804805-b23c-40ba-97b0-9ab2ba3efb5d.png`
4. `0cb0c1f0-f84a-47bb-bfa1-43b6c7f1575d.png`
5. `64e651b1-a8ba-44e8-8fbe-e85fcd6e5cf0.png`

All five use the same `d2ol7oe51mr4n9.cloudfront.net/user_38xzZboKViGWJOttwIXH07lWA1P/` base URL.

## Spotlight Reveal

The signature interaction is implemented by two stacked transparent PNGs:

- `.hero-front` is the normal white/silver android.
- `.hero-back` is the dark armor variation.

The pointer position is converted from rendered coordinates to the stage's unscaled coordinate system:

```js
var sx = stack.offsetWidth / rect.width;
var sy = stack.offsetHeight / rect.height;
mouse.x = (event.clientX - rect.left) * sx;
mouse.y = (event.clientY - rect.top) * sy;
```

This conversion is necessary because the stage is CSS-scaled. Without it, the spotlight would drift away from the cursor at fitted viewport sizes.

A requestAnimationFrame loop eases the spotlight position toward the pointer. The current position is written to CSS custom properties on `#hero`:

```css
--spot-x: 300px;
--spot-y: 380px;
```

Both images use radial masks based on those variables:

- `.hero-back` uses a normal mask, so the armor appears inside the spotlight.
- `.hero-front` uses an inverse mask, so the white android disappears inside the same spotlight.

The back layer has the higher z-index. This creates a real replacement effect instead of placing two faces on top of each other. Outside the spotlight, the front image remains visible. Inside it, the back image is the visible layer.

The spotlight uses a CSS radial gradient rather than rebuilding a `canvas.toDataURL()` on every frame. This avoids expensive per-frame image serialization and keeps cursor movement responsive.

## Intro Animation

The blocking script in `<head>` adds the `intro` class unless the user prefers reduced motion. CSS hides the main pieces while the Web Animations API reveals them in sequence:

- Brand, menu, navigation, and join control
- Headline lines and chain mark
- Hero
- Feature list and ghost typography
- Identity card and creator metadata
- Material control, gallery, and social buttons

A safety timeout removes `.intro` so a browser that does not support the animation API cannot leave the page hidden.

## Responsive Layout

`layout()` runs on page load and resize.

- **Desktop:** uses the `1536 x 1024` stage and scales it to the viewport.
- **Landscape or portrait-ish tablet:** uses a taller `640 x 1366` stage with adjusted positions.
- **Portrait phones up to 520px:** removes the stage transform and uses a natural CSS Grid with areas for header, title, artwork, material, gallery, card, and socials.

The hero measurements and spotlight coordinate conversion remain stage-relative. No mobile-specific hardcoded spotlight position is used.

## Interactions

- Navigation links update active states in both desktop and compact menus and scroll to their sections.
- The hamburger opens the compact menu, updates `aria-expanded`, and supports Escape to close.
- Join buttons toggle between `Join us` and `Joined`.
- The play control toggles the identity breathing state and `aria-pressed`.
- The eye control toggles the same identity preview state.
- Avatar buttons update the active avatar and apply hue/saturation variables to the hero front image.
- Shuffle and material-next advance through the five identities.
- Social buttons announce their selected network through the status toast.
- The Start creating link scrolls to `#create`.

## Accessibility

The page includes:

- A labeled main experience region.
- Real buttons and links for interactive controls.
- `aria-label` values for icon-only buttons.
- `aria-expanded` and `aria-hidden` menu state.
- `aria-pressed` on the play control.
- A polite live status toast.
- `:focus-visible` styling for keyboard navigation.
- Decorative images and marks hidden from assistive technology where appropriate.
- Reduced-motion handling for CSS and JavaScript animation.

## Editing Guide

Most changes can be made without touching the spotlight logic:

- Change colors and typography in `:root` and the main CSS blocks.
- Change image sources in the `assets` object.
- Change gallery behavior in `choose(index)`.
- Change section copy and structure in `.site-sections`.
- Change stage fitting rules in `layout()`.
- Change spotlight responsiveness in the smoothing factor inside `loop()`.
- Change spotlight size and softness in the radial-gradient definitions for `.hero-front` and `.hero-back`.

Keep the two spotlight masks synchronized. If the front and back masks use different coordinates or radii, the reveal will show a visible edge or misalignment.

## Known Constraints

- Remote assets are not bundled locally.
- The hero and gallery depend on the exact transparent PNG composition and their current alignment.
- The page is a front-end experience only; pricing and contact content are presentation-level placeholders until connected to a real product flow.
- Google Fonts and remote images may be blocked in offline or privacy-restricted environments.
