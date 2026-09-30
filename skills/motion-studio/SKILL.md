---
name: motion-studio
description: Create and refine code-rendered videos, motion graphics, product demos, animated explainers, and showreels with synchronized audio and reproducible exports. Use for video deliverables, rather than ordinary website animation.
---

# Motion Studio

Turn an idea into a finished video. Include source files the user can edit and clear steps to render the video again. Work in the user's project, follow its instructions, and use its existing tools. Match the work to what the user asks for.

## Choose the work needed

- **New video:** establish the brief, visual direction, and timed shot list; build a representative preview before the full render.
- **Existing video or project:** inspect the timeline, assets, and renderer; change the requested scenes or settings and verify the affected output.
- **Critique:** inspect the supplied video and report concrete findings with timestamps. Modify it only if requested.
- **Export or reformat:** preserve the approved content; recompose for each requested aspect ratio and verify the encoded files.

Reuse supplied decisions. Ask only for gaps that materially affect the result: purpose, duration, format, assets, visual direction, or audio. Honor requested approval checkpoints; otherwise proceed within the user's brief.

## Production essentials

1. Use authentic product assets and verified claims. Identify illustrative UI clearly. Read [production.md](references/production.md) for planning, motion direction, and longer films.
2. Preserve the working stack. For a new project, choose the simplest renderer that fits the brief; read [setup.md](references/setup.md) only when setup is needed.
3. Derive animation from absolute time and fixed inputs. Load fonts and assets before capture, seed randomness, and support backward seeks. Read [canvas-rendering.md](references/canvas-rendering.md) for the optional Canvas renderer and spring examples; these are starter code requiring project-specific validation.
4. Build a cheap preview, inspect it, and fix material defects before an expensive export. Keep every part of the timeline intentional and recompose text and focal points for each format.
5. If audio is requested, align meaningful actions with measured cues and keep voice intelligible. Read [audio.md](references/audio.md) for beat detection, sound effects, and mixing. Detected beats do not establish downbeats; effects alone do not make a score.
6. Inspect the encoded video, representative frames, transitions, and available audio playback. Check loop boundaries when looping is requested. Read [review.md](references/review.md) for commands and evidence. Revise until the brief is met within the agreed constraints; use observable defects rather than arbitrary scores or pass counts.

## Delivery

Deliver the requested videos, editable source and required assets, plus the commands and dependencies needed to reproduce them. Include a poster, contact sheets, or review log when useful or requested. Verify duration, dimensions, frame rate, and expected audio streams. State any playback, listening, or other checks that could not be performed.

Read [briefs.md](references/briefs.md) only when a reusable creative brief would help. Optional tools and services should serve the current task; this skill does not authorize paid services, publication, or delegation.
