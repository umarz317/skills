# Renderer setup

Read this only when the project needs setup. Inspect installed tools and dependencies first; preserve working versions and lockfiles. Keep setup in the user's project rather than the installed skill folder.

## Choose one route

- Use the existing renderer when it meets the brief.
- A small custom animation can use Canvas, Playwright, and FFmpeg; see [canvas-rendering.md](canvas-rendering.md).
- A React video project can use [Remotion](https://www.remotion.dev/docs).
- An HTML-oriented video project can use [HyperFrames](https://github.com/heygen-com/hyperframes).

Check the selected framework's current documentation before scaffolding or running framework-specific commands. Install only the route needed for this production. Agent-skill installation alone does not create a runnable video project.

## Custom Canvas route

Needs a supported Node.js runtime, FFmpeg, and Playwright's Chromium browser. Use the platform's package manager for missing prerequisites and preserve any project version requirements.

For a new project, initialize a package only if one is absent. The browser dependencies can be installed with:

```bash
npm install --save-dev playwright
npx playwright install chromium
```

[Playwright browser setup](https://playwright.dev/docs/browsers) documents platform requirements. Create the output directory before running the browser check below.

```bash
mkdir -p out
node --input-type=module <<'JS'
import { chromium } from 'playwright';
const browser = await chromium.launch({ headless: true });
try {
  const page = await browser.newPage();
  await page.setContent('<h1>Motion studio ready</h1>');
  await page.screenshot({ path: 'out/setup-check.png' });
} finally {
  await browser.close();
}
JS
```

Inspect the image. Then render a minimal, inexpensive animation with the selected pipeline and use ffprobe to verify the encoded duration, dimensions, and frame rate. For a custom renderer, also check repeated and out-of-order seeks; see [review.md](review.md). A browser screenshot alone does not verify MP4 export.

## Optional audio analysis

Use this only when the audio examples are needed:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install numpy librosa soundfile
```

Add local environments, generated outputs, dependency folders, and credential files to appropriate project ignore rules. Preserve existing entries. Verify imports and inspect actual analysis or sound output before treating audio setup as ready.
