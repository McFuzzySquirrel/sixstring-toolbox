---
name: showcase-capture
description: Capture screenshots, a captioned GIF and a WebM tour of a web project from a YAML or TypeScript config. Use when the user wants a visual tour, portfolio screenshots, a README gallery, or an automated showcase for a web app or static site.
---

# Showcase capture

Use this skill when someone wants screenshots, a GIF or a WebM walkthrough of a web project. The tool runs Playwright against a local or live site, follows a declarative scenario, and writes a README and gallery.

## What it produces

- `screens/*.png`
- `README.md` with the screenshots and captions
- `index.html` gallery
- `showcase.json` manifest
- optional `*.gif`
- optional `*.webm`

## Install

From the skill directory:

```bash
node scripts/install.mjs
```

From the npm package:

```bash
npx showcase-capture install
```

Or manually:

```bash
npm install
npm run build
npx playwright install chromium
```

Check the environment:

```bash
npx showcase-capture doctor
```

## Workflow

1. Inspect the project.
   - Framework and dev command.
   - Build output directory for static sites.
   - Routes and key screens.
   - Authentication flow and test user.
   - Seed or reset mechanism, usually localStorage or cookies.
   - The 5 to 8 screens that best explain the product.
   - One end-to-end flow for the GIF and WebM.

2. Write `showcase.config.yaml`.
   - Use a static, command or live URL server.
   - Use roles and accessible names before CSS selectors.
   - Add a `reset` block so every scene starts from a known state.
   - Use `mask` for emails, tokens, names and payment details.
   - Keep the GIF flow short: 20 to 40 seconds.

3. Validate the scenario. This checks the scenes and the GIF flow.

```bash
npx showcase-capture check
```

4. Capture.

```bash
npx showcase-capture capture
```

5. Review the output.
   - Open `README.md` and `index.html`.
   - Check the GIF and WebM.
   - Fix missing selectors, bad crops or timing.
   - Re-run with `--only <scene>` while iterating.

6. Commit the config and the generated output.

## Config example

```yaml
name: LaunchPad
baseUrl: http://localhost:5173
outDir: docs/showcase

server:
  mode: command
  command: npm run dev
  url: http://localhost:5173

reset:
  localStorage:
    - launchpad.demo.v2

context:
  locale: en-ZA
  timezoneId: Africa/Johannesburg
  fixedTime: "2026-11-09T09:00:00+02:00"
  reducedMotion: reduce

viewports:
  desktop: { width: 1440, height: 900 }
  mobile: { width: 390, height: 844 }

scenes:
  - name: desktop
    viewport: desktop
    steps:
      - goto: /
      - waitFor: h1
      - screenshot: 01-home

      - goto: /login
      - waitFor: form
      - click:
          role: button
          name: Continue as test user
      - waitForUrl: "#/app/dashboard"
      - screenshot: 06-dashboard

  - name: mobile
    viewport: mobile
    steps:
      - goto: /login
      - waitFor: form
      - click:
          role: button
          name: Continue as test user
      - waitForUrl: "#/app/dashboard"
      - screenshot: 21-mobile-dashboard

gif:
  viewport: { width: 1024, height: 640 }
  size: { width: 960, height: 600 }
  fps: 8
  steps:
    - goto: /
    - caption: [1, Discover, "Learn, match and book in one place"]
    - wait: 600
    - scroll: 660
    - goto: /learn
    - caption: [1, "Explore six practical modules", "Built around real freelance questions"]

video:
  enabled: true
  size: { width: 1280, height: 720 }
```

## Server modes

Static site:

```yaml
server:
  mode: static
  dir: dist
  spa: true
```

Dev server:

```yaml
server:
  mode: command
  command: npm run dev
  url: http://localhost:5173
```

Live URL:

```yaml
server:
  mode: none
baseUrl: https://example.com
```

Optional build step before serving:

```yaml
build: npm run build
```

## Overlay

The GIF overlay is optional. Use it to match the project brand or move the caption away from important content.

```yaml
overlay:
  background: "rgba(8, 20, 38, 0.93)"
  accent: "#5367e8"
  font: "Inter, system-ui, sans-serif"
  offsetX: 24
  offsetY: 24
```

Set `overlay.enabled: false` to record the GIF without captions or cursor.

## Step reference

- `goto: /path`
- `waitFor: h1` or `waitFor: { role: heading, name: Dashboard }`
- `click: ".selector"` or `click: { role: button, name: Continue }`
- `fill: { selector: "#email", value: "test@example.com" }`
- `press: Enter`
- `hover: ".card"`
- `waitForUrl: "#/app/dashboard"` or `waitForUrl: { pattern: "/app/", flags: "i" }`
- `wait: 500`
- `scroll: 600` or `scroll: { selector: ".footer" }`
- `screenshot: 01-home`
- `screenshot: { name: 01-home, fullPage: true }`
- `viewport: mobile`
- `caption: [1, "Title", "Subtitle"]` (GIF flow only)
- `reset: true`
- `eval: "localStorage.setItem('key', 'value')"`
- `run: function` (TypeScript or JavaScript config only)

## TypeScript config

Use a TypeScript config when the scenario needs custom logic:

```ts
import { defineShowcase } from 'showcase-capture'

export default defineShowcase({
  name: 'My project',
  outDir: 'docs/showcase',
  server: { mode: 'static', dir: 'dist' },
  scenes: [
    {
      name: 'desktop',
      steps: [
        { goto: '/' },
        {
          run: async ({ page }) => {
            await page.evaluate(() => localStorage.setItem('theme', 'light'))
          },
        },
        { waitFor: 'h1' },
        { screenshot: '01-home' },
      ],
    },
  ],
})
```

## Privacy

- Do not capture admin screens, secrets, tokens or payment details.
- Use `mask` for emails, names and account data.
- Use a test account with fictional data.
- Check the GIF and WebM before publishing.
- The tool does not upload anything.

## Troubleshooting

- `Could not launch Chromium`: run `npx playwright install chromium` or set `CHROME_PATH`.
- `Server did not respond`: check the command, port and URL. Use `reuseExisting: true` if the dev server is already running.
- `waitFor` times out: prefer `getByRole` selectors, check the page title, or add a `wait`.
- GIF is too long: shorten the steps, lower `fps`, or reduce the viewport.
- WebM is missing: check that `video.enabled` is true and a `gif` flow exists.
- Screenshots are wrong size: set the viewport in the scene or config.
