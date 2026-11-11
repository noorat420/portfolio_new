# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.) working on this repository.
Read this file fully before making changes.

## Project overview

Single-page personal portfolio for **Noora T, Full Stack Developer** (Kerala, India).

- Frontend only. No backend, no database, no routing library.
- Goal: a premium, cinematic dark site that shows real, deployed work and makes it easy to get in touch.
- Full specification: `docs/Noora_T_Portfolio_Project_Spec.pdf`. If this file and the spec disagree, the spec wins; flag the conflict.

## Tech stack

| Area | Tool |
|---|---|
| Build | Vite + React 18 (JavaScript, `.jsx`) |
| Styling | Tailwind CSS v4 (`@tailwindcss/vite`) |
| Animation | Framer Motion |
| Smooth scroll | Lenis |
| Icons | react-icons (Lucide `lu` + Simple Icons `si` sets only) |
| Fonts | `@fontsource-variable/manrope`, `@fontsource/jetbrains-mono` |
| Contact form | Web3Forms (fetch POST, no SDK) |
| Hosting | Vercel |

Do not add other dependencies (UI kits, CSS-in-JS, state libraries, router, jQuery, GSAP) without asking first.

## Commands

```bash
npm install          # install dependencies
npm run dev          # start dev server (http://localhost:5173)
npm run build        # production build to dist/
npm run preview      # serve the production build locally
npm run lint         # ESLint
npm run format       # Prettier
```

Before finishing any task, run `npm run lint` and `npm run build`; both must pass with no errors.

## Project structure

```
public/
  favicon.svg, apple-touch-icon.png, og-image.png
  Noora_T_Resume.pdf
  hero-loop.mp4, hero-poster.webp
  projects/              project screenshots (.webp)
src/
  assets/                profile.webp, noise.png
  components/            one component per file, PascalCase.jsx
  data/                  profile.js, projects.js, skills.js, experience.js
  hooks/                 useActiveSection.js, useLenis.js
  App.jsx                section order lives here
  main.jsx               font imports, root render
  index.css              Tailwind import + @theme tokens + utilities
docs/                    project spec and references
```

### Page order (App.jsx)

`Navbar` → `Hero` + `HeroBand` → `Work` (#work) → `Toolkit` (#toolkit) → `About` (#about) → `Experience` (#experience) → `Contact` (#contact) → `Footer`

## Design system (do not deviate)

### Colors: exactly three

| Token | Hex | Use |
|---|---|---|
| `bg` | `#070B14` | Page background |
| `ink` | `#F4F6FA` | All text; muted text via opacity (`text-ink/70`, `/50`); hairlines `border-ink/10` |
| `accent` | `#4CC9F0` | Section labels, badge dot, the period after headlines, focus rings, hover states |

- Never introduce a new color. Use opacity variants of these three only.
- Primary buttons are solid `ink` with `bg`-colored text.
- No gradients except the soft cyan glow behind the hero visual.

### Typography: exactly two fonts

- **Manrope** (`font-display`): all headlines and body. Headlines: weight 800, tracking `-0.045em`, line-height `0.95`.
- **JetBrains Mono** (`font-mono`): labels, nav links, buttons, tags, index numbers. Always uppercase, tracking `0.25em`, 11 to 12px.

### Signature patterns

- **Section header**: cyan mono label `0X / NAME` → giant title ending in a cyan period → muted subtext right-aligned on desktop. Use `SectionHeader`.
- **Cyan period**: every section title ends with `<span className="text-accent">.</span>`.
- **Accordion rows** (Work, Experience): index | big title | outlined pill `Tag` | plus icon, separated by hairlines. Use `AccordionRow`. One row open at a time.
- **Buttons**: rectangular (2px radius), 56px tall, mono uppercase label + arrow icon. Use `Button` (`variant="primary" | "secondary"`).
- **Structure**: hairline borders and vertical grid lines, not cards with shadows. No `shadow-*` classes.
- **Outlined text**: `.outline-text` utility (transparent fill, 1px stroke).
- **Texture**: 3% film-grain overlay on the page, `pointer-events-none`.

### Layout

- Max width `1440px`, centered. Side padding `px-5` (mobile) to `px-[60px]` (desktop).
- Mobile-first Tailwind: write base styles for 360px, then add `sm:` `md:` `lg:` `xl:`.
- Never allow horizontal scrolling at any width.

## Coding conventions

### Content is data

- All text, links, projects, skills and experience live in `src/data/*.js`.
- Components never hard-code copy that belongs in data. To add a project or skill, edit only the data file.

### Components

- Function components declared as `export default function Name() {}`.
- One component per file in `src/components/`, named `PascalCase.jsx`.
- Props destructured in the signature. Keep components under ~150 lines; extract sub-components if larger.
- Reuse the primitives (`Button`, `Tag`, `SectionHeader`, `RevealLines`, `AccordionRow`, `OutlineWordmark`) instead of re-styling elements.
- Style with Tailwind classes only. Custom CSS goes in `index.css` as a named utility, never inline `style` objects (except dynamic motion values).

### Animation rules

- Use Framer Motion. Animate only `opacity` and `transform` (x, y, scale, rotate).
- Default easing `[0.22, 1, 0.36, 1]`, durations 0.4 to 0.9s. Motion must feel calm, never bouncy or flashy.
- Scroll reveals use `whileInView` with `viewport={{ once: true }}`.
- Every animated component must respect reduced motion via `useReducedMotion()`. When reduced: no Lenis, no marquee, no video loop (poster only), no transforms.
- Hover effects only on pointer devices (`@media (hover: hover)` or Tailwind `hover:` inside a hover-capable context).

### Accessibility (required, not optional)

- One `<h1>` (the hero headline). Sections use `<section id>` with an `<h2>`.
- Semantic landmarks: `header`, `nav`, `main`, `section`, `footer`.
- Every image has meaningful `alt`; decorative elements get `aria-hidden="true"`.
- Icon-only links need `aria-label` (e.g. `"GitHub profile"`).
- Accordions: `button` elements with `aria-expanded` and `aria-controls`.
- Visible focus ring: `focus-visible:outline-2 focus-visible:outline-accent`.
- Mobile menu traps focus and closes on Esc.
- External links: `target="_blank" rel="noopener noreferrer"`.

### Performance

- Images: WebP, max 1600px wide, under 200KB, `loading="lazy"` below the fold.
- Hero video: `muted loop playsInline autoPlay` with a `poster`; show the poster only on mobile or reduced motion.
- Import only the font weights in use.
- Target: initial JS under 200KB gzipped, LCP under 2.5s, Lighthouse 90+ in all categories.

## Contact form

- POST `FormData` to `https://api.web3forms.com/submit` with `access_key` from `import.meta.env.VITE_WEB3FORMS_KEY`.
- Fields: name (required, 2 to 60), email (required, valid), subject (optional, up to 100), message (required, 10 to 1000).
- Include a hidden `botcheck` honeypot checkbox.
- States: `idle` → `sending` → `success` | `error`. The error state shows a `mailto:` fallback.
- Submissions go to `noora_thanari@outlook.com`.

## Environment variables

| Name | Where |
|---|---|
| `VITE_WEB3FORMS_KEY` | `.env` locally; Vercel Project Settings for Production and Preview |

- Never commit `.env`. Keep `.env.example` updated with variable names only.

## Privacy and content rules

- Do not put Noora's phone number anywhere on the site.
- Do not invent projects, employers, metrics, testimonials or certifications. If data is missing, leave a clear `TODO:` in the data file and mention it in your summary.
- Do not copy text, images or branding from the reference template; the design is inspired by it, the content is original.

## Git conventions

- Branch per Jira task: `feature/PORT-12-hero-section`, `fix/PORT-40-mobile-menu-focus`.
- Commit messages: Conventional Commits, referencing the Jira key.
  - `feat(hero): add video background and badge (PORT-12)`
  - `fix(contact): validate email format (PORT-38)`
  - `chore: configure prettier (PORT-6)`
- Keep commits small and focused. Do not commit `dist/`, `node_modules/` or `.env`.

## Definition of done (every task)

- [ ] Matches the spec and this file's design system
- [ ] Works at 360px, 768px, 1024px and 1440px with no horizontal scroll
- [ ] Keyboard accessible, with reduced-motion handled
- [ ] Content comes from `src/data/`
- [ ] `npm run lint` and `npm run build` pass
- [ ] No console errors or warnings
- [ ] Summary of changes and any `TODO:` items reported back

## Deployment

- Vercel auto-deploys `main`; pull requests get preview URLs.
- Build command `npm run build`, output directory `dist`, framework preset Vite.
- Custom domain and HTTPS are configured in Vercel > Settings > Domains.

## When unsure

Ask before: adding a dependency, changing the palette or fonts, changing section order, or restructuring folders.
Otherwise choose the option that keeps the code simple and the design consistent, and note the decision in your summary.