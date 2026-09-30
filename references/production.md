# Production direction

Read this when planning a new production or making substantial creative changes.

## Brief and assets

Establish purpose, audience, main message, duration, aspect ratios, visual references, brand assets, and audio needs from the user's input. For product work, inventory approved screenshots, logos, and recordings before designing shots. Do not invent features, testimonials, or customer data. Distinguish illustrative interfaces from real product screens.

When the user delegates creative decisions, choose coherent defaults and state the consequential assumptions briefly. Match the planning effort to the scope: a short logo animation may need a few timing notes; a narrative film may need a storyboard and animatic.

## Visual story

Record the composition, palette, typography, texture, pacing, and transition principles needed to keep the film consistent. Create a timed shot list with on-screen content, motion, and sound cues. Make each shot advance the story, allow enough reading time, and give the opening and ending a clear purpose.

For UI animation, map the actual states and transitions. Identify which objects persist, morph, enter, or leave. Make cursor motion, clicks, and resulting state changes agree. Keep text readable while containers change shape.

Share the direction and timeline when the user's workflow calls for a checkpoint. A reference article's workflow does not add a mandatory approval step to the user's task.

## Motion

Choose movement from the brief and references. Give objects a consistent sense of weight and acceleration; avoid repeating generic fade-ins or arbitrary decorative movement. Use restrained overshoot where legibility matters.

For spring-based transitions, use a response evaluable at any timestamp. Successive target changes should preserve continuity; summing time-shifted responses to target deltas is one option. Handle underdamped, critical, and overdamped cases explicitly when implementing springs.

For a loop, match boundary position, velocity, opacity, and content. Wrapping time repeats a timeline but does not make the boundary seamless. Inspect the end-to-start transition in playback.

## Longer or character-led films

Establish character proportions, palette, expressions, and visual identity before producing all scenes. Work through a timed storyboard, representative stills or a rough edit, a representative animation section, and then the full film. Reuse approved assets and motion primitives for consistency.

For authorized parallel work, establish shared timing, typography, palette, asset ownership, and scene interfaces before splitting chapters. Integrate the chapters into one timeline and inspect their transitions. Use external image, video, or voice generation only when it serves the brief and is authorized.

## Formats and project organization

Share the timeline across requested formats, but recompose text, UI, and focal points for each aspect ratio. Check safe margins and readability separately; cropping a widescreen master often breaks a vertical composition.

Adapt to the existing project. Useful locations include `assets/` for approved inputs, `src/` for scenes, `scripts/` for render utilities, `out/` for exports, and brief notes in `docs/`. Create shared motion helpers only when scenes actually reuse them.

Frame rate, render sampling, resolution, and loudness should follow the destination and brief. Start with an inexpensive preview. Higher temporal sampling can improve motion blur but adds render cost; apply it where useful.
