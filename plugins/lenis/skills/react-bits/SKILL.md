---
name: react-bits
description: React Bits (reactbits.dev) — 215+ free animated React/Next.js components, text animations, backgrounds and micro-interactions installed via the shadcn or jsrepo CLI. Use when the user mentions React Bits, or wants an animated hero, text effect (split text, blur text, shiny text, decrypted text), animated/WebGL background (aurora, particles, waves), cursor effect, or fancy interactive component in a React or Next.js project.
license: MIT
---

# React Bits

React Bits is a free, open source library of animated React components. Components are **copied into the project as source** (like shadcn/ui), not imported from an npm package. Every component comes in 4 flavours:

| Suffix | Language | Styling |
|---|---|---|
| `JS-CSS` | JavaScript | plain CSS (default) |
| `JS-TW` | JavaScript | Tailwind |
| `TS-CSS` | TypeScript | plain CSS |
| `TS-TW` | TypeScript | Tailwind |

## Workflow

1. **Pick the component.** Read `catalog.md` (in this skill's folder) to find candidates by category: Text Animations, Animations, Components, Micro, Backgrounds. Each entry lists the **CLI identifier** in PascalCase (e.g. `SplitText`). Suggest 2–3 options when the request is vague.
2. **Pick the flavour from the project.** `tsconfig.json` present → `TS`, else `JS`. Tailwind configured (`tailwind.config.*`, `@import "tailwindcss"` in CSS, or `tailwindcss` in package.json) → `TW`, else `CSS`. Don't ask the user if the project makes it obvious.
3. **Install with the CLI** (run from the project root):

```bash
npx shadcn@latest add https://reactbits.dev/r/<Component>-<LANG>-<STYLE>
# e.g.
npx shadcn@latest add https://reactbits.dev/r/SplitText-TS-TW
```

If the project already has `components.json` with the React Bits registry, the short form works: `npx shadcn@latest add @react-bits/SplitText-TS-TW`.

Alternative (no shadcn setup needed):

```bash
npx jsrepo@latest add https://reactbits.dev/r/SplitText-TS-TW
```

If shadcn prompts to initialise (no `components.json`), either accept the defaults or use jsrepo instead.

4. **Check dependencies.** The CLI usually installs them, but verify. Common ones: `gsap` (many text/scroll animations), `motion` (Framer Motion), `three` / `@react-three/fiber` (3D), `ogl` (WebGL backgrounds). Install anything missing.
5. **Read the installed file** before using it: confirm the export name, props and defaults, then render it with sensible props. Don't guess props from memory — the source is in the project now.
6. **Next.js App Router:** these components use hooks/browser APIs, so the file (or the component that renders it) needs `'use client'` at the top. WebGL/three backgrounds may also need `dynamic(() => import(...), { ssr: false })`.

## Usage tips

- Backgrounds are usually absolutely positioned and fill their parent: give the parent `position: relative` and an explicit height (e.g. `h-screen`), and layer content above with `z-index`.
- Components are owned source — edit them freely to match the design (colours, timing, sizes).
- GSAP-based components work with the GSAP and Lenis skills: if the site uses Lenis smooth scroll with ScrollTrigger, sync them as the Lenis skill describes.
- Respect `prefers-reduced-motion` for heavy effects where practical.
- Keep heavy WebGL backgrounds to one per page for performance.

## MCP option (per project)

For live search of the registry, the user can set up the shadcn MCP server in a project. Add to `components.json`:

```json
{
  "registries": {
    "@react-bits": "https://reactbits.dev/r/{name}.json"
  }
}
```

then run `npx shadcn@latest mcp init --client claude` and restart Claude Code. Docs: https://reactbits.dev/get-started/mcp

## Pro

React Bits Pro (paid) has page blocks, app UI, templates and agent skills. Only mention it if the user asks for full sections/templates; default to the free components.

## Do Not

- ❌ `npm install react-bits` — there is no such package for this library; use the CLI URLs above.
- ❌ Mix flavours (e.g. TS-TW component in a JS + plain CSS project).
- ❌ Invent component names — use identifiers from `catalog.md`.
- ❌ Forget `'use client'` in Next.js App Router.
