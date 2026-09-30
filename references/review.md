# Export review

Read this when verifying a preview or final export. Match inspection to the work: check changed scenes and adjacent transitions for a revision; cover the whole timeline for a finished production. Inspect encoded output, not just the source preview.

## Sampling and metadata


These commands create a sample overview, a close sequence of frames, a phone-size sheet, and a doubled loop. They assume `out/final.mp4` exists.

```bash
ffmpeg -i out/final.mp4 -vf "fps=2,scale=270:-1,tile=6x5" -frames:v 1 out/contact.png
ffmpeg -ss 1 -i out/final.mp4 -vf "scale=320:-1,tile=12x1" -frames:v 1 out/action-strip.png
ffmpeg -i out/final.mp4 -vf "fps=1,scale=360:-1,tile=5x3" -frames:v 1 out/phone.png
ffmpeg -stream_loop 1 -i out/final.mp4 -c copy out/loop-check.mp4
ffprobe -v error -show_entries stream=codec_type,codec_name,width,height,r_frame_rate,sample_rate,channels -show_entries format=duration -of json out/final.mp4
ffmpeg -i out/final.mp4 -vn -af loudnorm=I=-14:TP=-1:LRA=11:print_format=json -f null -
```

A 30-cell sheet sampled at 2 fps covers only 15 seconds. For longer films, generate additional sheets using explicit time ranges; do not assume one sheet covers the whole film. Adjust the action-strip start time to the motion under review.

Test determinism on decoded pixels at selected timestamps, including a backward seek: capture at `t=1`, seek to `t=3`, then capture at `t=1` again. Compare the two RGBA arrays. Also compare a fresh browser session. Hashing an encoded MP4 can be affected by encoding metadata and does not isolate the frame being tested. For the starter Canvas, pixel data can be read with:

```js
const pixels = await page.evaluate(() => {
  const c = document.querySelector('#stage');
  return Array.from(c.getContext('2d').getImageData(0, 0, c.width, c.height).data);
});
```

Record concrete timestamped defects, fixes, and remaining limitations when a review log is useful. Inspect composition, text legibility, motion, pacing, brand fidelity, audio alignment, and loop continuity. Render affected sections after a change. Continue until the brief is met and material defects are resolved within the agreed constraints; neither a pass count nor a score proves quality. Report unavailable listening or playback checks explicitly.

