# Motion Studio

Create code-rendered videos and motion graphics with editable source, synchronized audio, and reproducible exports.

Supports product launch videos, UI demonstrations, animated explainers, showreels, and narrative animation. It also handles focused revisions, critiques, exports, and aspect-ratio adaptations in existing projects.

## Install

```bash
npx skills add umarz317/motion-studio --skill motion-studio
```

## Use

Ask your agent to use `motion-studio` and describe the deliverable. Supply duration, audience, formats, authentic assets, visual references, and audio preferences when available.

> Use motion-studio to make a 20-second product launch video from the supplied screenshots and music. Deliver vertical and widescreen versions with editable source and render instructions.

> Use motion-studio to improve the pacing and text legibility of this existing video. Preserve its visual identity and renderer.

> Use motion-studio to critique this explainer. Give timestamped findings without editing it.

The skill preserves a project's working stack. Canvas with Playwright and FFmpeg is an optional route; framework and audio setup are loaded only when needed. Reference code is starter code, not a runtime-validated production tool.

## Package

`SKILL.md` contains the core workflow. Focused references cover production direction, setup, Canvas rendering, audio, review, and creative briefs.

## Attribution

Developed from the original Motion Reel skill. The supporting production approach and reference examples were adapted from [Movez's motion-design article](https://x.com/0xMovez/status/2104216919033192746). Code examples were written for the original guide. Article-specific model claims, prices, and numerical review targets are not requirements of this skill.
