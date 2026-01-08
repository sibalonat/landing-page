# Copilot Instructions for National Gallery Landing Page

## Project Overview

This is an Astro-based landing page for the National Gallery of Arts of Albania (Galeria Kombëtare e Arteve), featuring a minimalist design with animated logo transitions and a "coming soon" message. The site uses GSAP for choreographed animations and Tailwind CSS v4 for styling.

## Tech Stack & Configuration

- **Framework**: Astro 5 with React integration enabled
- **Styling**: Tailwind CSS v4 (configured via Vite plugin in [astro.config.mjs](astro.config.mjs))
- **Animation**: GSAP (with TextPlugin) for complex timeline sequences
- **TypeScript**: Strict mode with React JSX transform
- **Dev server**: `npm run dev` runs on `localhost:4321`

## Architecture & Key Patterns

### Layout-Slot Architecture
The project uses Astro's slot pattern for composable layouts. [Layout.astro](src/layouts/Layout.astro) defines two named slots:
- `<slot name="logos" />` - Injected logo content (hero section)
- `<slot />` - Default page content (below hero)

Pages inject content into slots:
```astro
<Layout title="..." description="...">
  <div slot="logos"><!-- Logo markup --></div>
  <!-- Default slot content -->
</Layout>
```

### Dual Animation System
Animations run in **two separate GSAP timelines** that must coordinate:

1. **Page-level** ([index.astro](src/pages/index.astro#L115-L175)): Controls logo crossfade and repositioning
   - English → Albanian text transition
   - GKA logo container movement to top-left
   - MTKS logo fade-in

2. **Layout-level** ([Layout.astro](src/layouts/Layout.astro#L95-L170)): Controls "coming soon" typing and contact reveal
   - Triggered at `delay: 4.2` to sync after logo animations
   - Typewriter effect for "Coming soon"
   - Staggered contact info appearance

**Critical timing**: Layout animations start at 4.2s delay to allow page animations (≈3.5s) to complete.

### Responsive Animation Strategy
Both timelines check `window.innerWidth` to adjust:
- Mobile (`< 640px`): Reduced animation distances, faster durations, no rotation effects
- Tablet (`640-1024px`): Medium-scale animations with specific logo positioning
- Desktop (`≥ 1024px`): Full animation effects with elastic easing

Example from [index.astro](src/pages/index.astro#L138-L147):
```javascript
tl.to("#gka-logo-container", {
  left: isMobile ? "3rem" : isTablet ? "0.75rem" : "1rem",
  scale: isMobile ? 1 : isTablet ? 0.7 : 0.8,
  duration: isMobile ? 0.8 : 1,
  // ...
});
```

### Asset Organization
Assets are organized by institution:
- `src/assets/gka/` - National Gallery assets (logo, text SVGs, OG image)
- `src/assets/mtks/` - Ministry of Tourism logo
- Custom fonts in `src/styles/fonts/` (Accunthisa, Kufam) preloaded in layout head

## Styling Conventions

### Custom Font System
Three font families with specific usage:
- **Kufam**: Body text (uppercase headings like "Coming soon")
- **Accunthisa**: Email/contact text
- **Mulish** (Google Font): Logo text matching original branding

Defined in [global.css](src/styles/global.css#L5-L15) with `@font-face` and preloaded in [Layout.astro](src/layouts/Layout.astro#L26-L38).

### Tailwind v4 Patterns
Using Tailwind v4 via `@tailwindcss/vite` plugin:
- Responsive spacing: `inset-4 sm:inset-8 md:inset-12`
- Golden ratio layout: `aspect-[1.618]` container at 65vh
- Breakpoint-specific sizing: `w-20 sm:w-24 md:w-35 lg:w-57.5`

Note: Arbitrary values use bracket notation (`w-[1.8rem]`)

## SEO & Meta Implementation

### Structured Data
JSON-LD schema in [index.astro](src/pages/index.astro#L14-L58) defines Museum type with:
- Opening hours (Tue-Sun, 10:00-18:00)
- Contact info and social media
- Address data for Tirana location

Injected via `slot="head"` with `is:inline` directive.

### Meta Tags
Layout accepts props: `title`, `description`, `email` with sensible defaults.

## Common Development Tasks

### Adding New Animations
1. Identify which timeline (page or layout) based on element location
2. Check if adjustment affects timing of dependent animations
3. Add responsive breakpoints for mobile/tablet if animation involves movement
4. Test that `delay` values maintain coordination between timelines

### Modifying Logo Behavior
Logo container (`#gka-logo-container`) uses absolute positioning with responsive offsets. When changing:
- Update responsive `left`/`top` values for all breakpoints
- Maintain `shrink-0` on red-g.svg to prevent squashing
- Albanian text (`#text-al`) starts at `opacity: 0` - don't remove

### Working with Custom Fonts
Fonts are preloaded and defined separately:
1. Add `.woff2` to `src/styles/fonts/`
2. Define `@font-face` in [global.css](src/styles/global.css)
3. Import and preload in [Layout.astro](src/layouts/Layout.astro) head
4. Apply via Tailwind utility class (e.g., `font-kufam`)

## Build & Deployment

- **Build**: `npm run build` → outputs to `./dist/`
- **Preview**: `npm run preview` to test production build locally
- **Static output**: No SSR/SSG configuration - pure static site

## Project-Specific Notes

- This is a **temporary landing page** - explains minimal content structure
- Bilingual support (EN/AL) is animation-based, not i18n framework
- No component directory - all code in pages/layouts due to single-page scope
- GSAP TextPlugin must be explicitly registered before use
- Dark mode classes present but not actively styled (future consideration)
