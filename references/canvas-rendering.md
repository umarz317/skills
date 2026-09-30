# Canvas rendering starter

Read this only when implementing a custom browser renderer. Preserve an existing working pipeline. These examples come from the original skill and have not been runtime-tested in this revision; adapt and test them before relying on them for production.

## Contents

- Minimal scene and explicit time interface
- Browser-to-FFmpeg renderer
- Analytic springs and reusable motion

## Minimal scene


Create `index.html` in the project root. This original starter covers its entire duration and exposes metadata for the renderer. Replace the simple demonstration with the approved shot list.

```html
<!doctype html>
<meta charset="utf-8">
<style>html,body{margin:0;background:#182025}canvas{display:block}</style>
<canvas id="stage" width="720" height="1280"></canvas>
<script>
const canvas = document.querySelector('canvas');
const ctx = canvas.getContext('2d');
window.video = { width: 720, height: 1280, duration: 4 };
window.ready = document.fonts.ready.then(() => true);
window.seek = (seconds) => {
  const t = Math.max(0, Math.min(window.video.duration, seconds));
  const phase = t / window.video.duration;
  const x = 360 + Math.sin(phase * Math.PI * 2) * 180;
  ctx.setTransform(1, 0, 0, 1, 0, 0);
  ctx.globalAlpha = 1;
  ctx.fillStyle = '#182025';
  ctx.fillRect(0, 0, 720, 1280);
  ctx.fillStyle = '#e2b46d';
  ctx.beginPath(); ctx.arc(x, 620, 60, 0, 2 * Math.PI); ctx.fill();
  ctx.fillStyle = '#f2eee5';
  ctx.font = 'bold 52px sans-serif';
  ctx.textAlign = 'center';
  ctx.fillText('Time becomes motion', 360, 420);
};
window.seek(0);
if (!new URLSearchParams(location.search).has('render')) {
  const start = performance.now();
  function preview(now) {
    window.seek(((now - start) / 1000) % window.video.duration);
    requestAnimationFrame(preview);
  }
  requestAnimationFrame(preview);
}
</script>
```

### Browser-to-FFmpeg renderer

Create `scripts/render.mjs`. The renderer averages consecutive subframes, limits the output to the intended frame count, handles pipe backpressure, and rejects FFmpeg failures. Keep width and height even for yuv420p. Supply an output path to keep previews and final renders separate.

```js
import { chromium } from 'playwright';
import { spawn } from 'node:child_process';
import { once } from 'node:events';
import { mkdir } from 'node:fs/promises';
import { resolve, dirname } from 'node:path';
import { pathToFileURL } from 'node:url';

function option(name, fallback) {
  const i = process.argv.indexOf(`--${name}`);
  return i < 0 ? fallback : process.argv[i + 1];
}
const fps = Number(option('fps', 60));
const samples = Number(option('samples', 4));
if (!Number.isInteger(fps) || fps < 1 ||
    !Number.isInteger(samples) || samples < 1 || samples > 16) {
  throw new Error('Use a positive integer fps and 1–16 samples.');
}
const output = resolve(option('out', 'out/silent.mp4'));
await mkdir(dirname(output), { recursive: true });
const browser = await chromium.launch();
let encoder;
try {
  const page = await browser.newPage();
  const url = pathToFileURL(resolve('index.html'));
  url.searchParams.set('render', '1');
  await page.goto(url.href);
  await page.evaluate(async () => {
    await document.fonts.ready;
    await window.ready;
  });
  const meta = await page.evaluate(() => window.video);
  if (!meta || !Number.isFinite(meta.duration) || meta.duration <= 0 ||
      !Number.isInteger(meta.width) || !Number.isInteger(meta.height) ||
      meta.width <= 0 || meta.height <= 0 || meta.width % 2 || meta.height % 2) {
    throw new Error('Invalid video metadata: positive duration and even dimensions required.');
  }
  await page.setViewportSize({ width: meta.width, height: meta.height });
  const frames = Math.round(meta.duration * fps);
  if (frames < 1) throw new Error('Duration is shorter than one output frame.');
  const filters = samples === 1 ? [] : [
    '-vf', `tmix=frames=${samples},select=eq(mod(n\\,${samples})\\,${samples - 1}),setpts=N/(${fps}*TB)`
  ];
  encoder = spawn('ffmpeg', [
    '-y', '-f', 'image2pipe', '-framerate', String(fps * samples),
    '-i', 'pipe:0', ...filters, '-r', String(fps),
    '-frames:v', String(frames), '-an', '-c:v', 'libx264',
    '-crf', '16', '-pix_fmt', 'yuv420p', '-movflags', '+faststart', output
  ], { stdio: ['pipe', 'inherit', 'inherit'] });
  let pipeError;
  encoder.stdin.on('error', error => { pipeError = error; });
  // Resolve errors as values so an early process failure cannot cause an
  // unhandled rejection while the browser is still rendering.
  const finished = new Promise(resolveDone => {
    encoder.once('error', error => resolveDone({ error }));
    encoder.once('close', code => resolveDone({ code }));
  });
  for (let n = 0; n < frames * samples; n++) {
    if (pipeError) throw pipeError;
    await page.evaluate(t => window.seek(t), n / (fps * samples));
    const png = await page.locator('#stage').screenshot({ animations: 'disabled' });
    if (!encoder.stdin.write(png)) await once(encoder.stdin, 'drain');
  }
  encoder.stdin.end();
  const result = await finished;
  if (result.error) throw result.error;
  if (result.code !== 0) throw new Error(`FFmpeg failed: ${result.code}`);
  console.log(output);
} finally {
  if (encoder && encoder.exitCode === null) encoder.kill();
  await browser.close();
}
```

```bash
node scripts/render.mjs --fps 30 --samples 1 --out out/preview.mp4
node scripts/render.mjs --fps 60 --samples 4 --out out/silent.mp4
```

Four samples approximate a full-frame shutter. Adjust the sample times if a shorter shutter is required. Load all image assets through `window.ready` before capturing; waiting for fonts alone is insufficient.

### Analytic springs and reusable motion

Create `lib/motion.mjs`. These functions assume unit mass and handle underdamped, critically damped, and overdamped springs separately. The overdamped case is handled separately from critical damping.

```js
export const clamp01 = x => Math.max(0, Math.min(1, x));
export function spring(t, stiffness = 180, damping = 24) {
  if (!(stiffness > 0) || !(damping > 0)) throw new Error('Positive spring parameters required');
  if (t <= 0) return 0;
  const w = Math.sqrt(stiffness), z = damping / (2 * w);
  if (Math.abs(z - 1) < 1e-6) return 1 - (1 + w * t) * Math.exp(-w * t);
  if (z < 1) {
    const q = w * Math.sqrt(1 - z * z);
    return 1 - Math.exp(-z * w * t) *
      (Math.cos(q * t) + z * w / q * Math.sin(q * t));
  }
  const root = Math.sqrt(z * z - 1);
  const a = -w * (z - root), b = -w * (z + root);
  return 1 + (b * Math.exp(a * t) - a * Math.exp(b * t)) / (a - b);
}
export function track(t, keys, stiffness = 180, damping = 24) {
  if (!keys.length) throw new Error('Track needs a starting value');
  let value = keys[0][1];
  for (let i = 1; i < keys.length; i++) {
    value += (keys[i][1] - keys[i - 1][1]) *
      spring(t - keys[i][0], stiffness, damping);
  }
  return value;
}
export function stretchingIndicator(t, keys, width = 100) {
  const a = track(t, keys, 300, 30), b = track(t, keys, 150, 24);
  return { x: Math.min(a, b), width: Math.abs(a - b) + width };
}
export function contentOpacity(t, enter, leave) {
  return Math.min(clamp01((t - enter - 0.08) / 0.14),
    clamp01((leave - 0.1 - t) / 0.12));
}
export const wrapTime = (t, duration) => ((t % duration) + duration) % duration;
```

Keep track keys chronologically ordered. Treat the first key's value as the baseline. Wrapping time only repeats a timeline: design matching boundary position, velocity, opacity, and content to make the repetition seamless. Check scaled text for raster blur, especially with compositing hints such as `will-change`.

