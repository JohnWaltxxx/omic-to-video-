---
name: comic-to-video
description: Turn comic pages or comic panel images into a true animated comic using Seedance image-to-video generation. Use when each comic panel should become an animated shot with moving characters, facial expressions, clothing, effects, or camera motion, then be assembled in reading order into one video. This is not the simple pan-and-zoom treatment of a static comic page.
---

# Comic to Video with Seedance

Turn each comic panel into an independent Seedance-generated video clip, then assemble the clips in story order. The source panel is the visual anchor: preserve its character identity, costume, setting, palette, composition, speech balloons, captions, and drawn style while animating the people and environment inside the panel.

## Core distinction

The default goal is true image-to-video animation, not a virtual camera moving over a still image. Every panel should be sent to Seedance as its own image-to-video job. A camera-only effect may be used as a fallback when Seedance fails or when the user explicitly asks for a motion-comic version.

## Input handling

- Accept one full comic page, multiple comic pages, or a folder of already-cropped panels.
- For a full page, identify every panel rectangle and the reading order before generating any video.
- Preserve the user's intended reading direction. For Chinese comics and ordinary Western layouts, default to top-to-bottom and left-to-right; for manga, use right-to-left and top-to-bottom unless the user says otherwise.
- Never treat a speech balloon or caption as a separate panel unless it is visually framed as one.
- Keep the original upload unchanged. Create working crops in a separate project folder.
- If panel boundaries or reading order are uncertain, stop before generation and present the proposed panel list for correction.

## Project structure

Create a project folder named \`comic-seedance-<input-name>\`:

- \`input/\` — original page images, copied or referenced without modification;
- \`panels/\` — one lossless crop per comic panel;
- \`seedance/panel-manifest.json\` — segmentation, order, duration, and references;
- \`seedance/prompts/\` — one prompt file per panel;
- \`seedance/jobs.json\` — provider-ready job plan, without secrets;
- \`clips/\` — downloaded Seedance results, one clip per panel;
- \`final/\` — concatenated video and optional caption/audio versions;
- \`review/\` — contact sheets, frame grabs, and a QC report.

## Panel generation workflow

1. Detect or manually define panel crops and write the manifest described in [references/seedance-manifest.md](references/seedance-manifest.md).
2. For each panel, inspect the action, characters, gaze direction, environment, props, and likely next beat. Write a short, concrete animation prompt rather than a generic “make it move”.
3. Use the panel crop as the primary input image. If the provider supports reference images, add only references that improve character/style continuity; do not add unrelated artwork.
4. Generate a short clip for each panel, normally 3–6 seconds. Use the shortest duration that lets the action read clearly. Keep the panel's composition stable unless a controlled camera move is part of the intended shot.
5. Download each result into \`clips/\` and record its provider job ID, model/version, prompt, duration, aspect ratio, and status. Never place API keys in the manifest or prompt files.
6. Review every clip for identity drift, extra limbs, broken faces, warped speech balloons, changing clothing, unwanted cuts, and motion that contradicts the panel. Regenerate only the failed panel with a more constrained prompt.
7. Concatenate successful clips in manifest order. Use a short crossfade only when a hard cut would feel abrupt; do not reorder clips to improve visual rhythm if that changes the story.
8. Add voiceover, music, subtitles, or sound effects only when requested. Original comic text should remain visible unless the user explicitly asks for text replacement or removal.

## Prompt construction

Every Seedance prompt should contain these parts, in plain language:

1. **Source lock:** “Use the input comic panel as the exact visual reference. Preserve the same characters, faces, clothing, colors, setting, composition, linework, and illustration style.”
2. **Action:** Describe what moves, in what direction, and with what emotional intent.
3. **Secondary motion:** Add only a few supporting motions such as cape flutter, hair bounce, candle flame, dust, mirror shimmer, rain, or a subtle camera push.
4. **Continuity:** State what must remain fixed, especially character identity, costume, props, panel framing, and Chinese lettering.
5. **Negative constraints:** “No new characters, no redesign, no scene change, no extra limbs, no deformed hands, no text changes, no random cuts, no subtitles added by the model.”

Use visual specificity. For example, for a sheep hero raising its paws: describe the paws lifting, cape flaring once, eyes narrowing with determination, pumpkin light flickering, and a very small forward camera push. Do not ask the model to invent a new story beat that is absent from the panel.

## Text and speech balloons

- Treat existing Chinese text as part of the artwork and request that it remain unchanged and legible.
- Avoid asking Seedance to animate individual Chinese characters or redraw lettering. If a balloon must move, keep its shape and lettering stable and animate only the surrounding character or camera.
- If generated footage repeatedly damages text, create a text-protected composite: animate a clean copy of the panel underneath and overlay the original speech balloons/captions during assembly.

## Character consistency

- Use the same character description and style-lock phrase in every panel prompt.
- When reference-image support is available, use the first clean character panel or a user-approved character reference consistently across the relevant jobs.
- Do not overstuff a job with references. Prefer the current panel plus one or two continuity references over a large unrelated image set.
- If a panel needs a large change in pose, preserve face shape, horn/ear shape, costume, colors, and distinctive props explicitly in the prompt.

## Continuity modes

Choose the least complex mode that meets the story:

- **Independent panel mode (default):** generate every panel separately, then concatenate. Best for ordinary comic pages.
- **Reference continuity mode:** use a consistent character/style reference for all related panels. Best when the same character appears repeatedly.
- **First/last-frame mode:** use when one panel's action must flow directly into the next and the provider supports first/last-frame control.
- **Video extension mode:** use only when a panel's movement must continue across a boundary. Keep the original panel order and check for style drift.

## Quality gates

A clip passes only if:

- the intended action is visible within the first second;
- the main character remains recognizable;
- the comic's style and palette remain coherent;
- no important speech balloon, caption, or sound-effect lettering is destroyed;
- there are no obvious anatomy glitches or unexplained new objects;
- the clip begins and ends in a state that can cut to the neighboring panel;
- the final concatenation preserves the original story order.

If a panel fails, regenerate that panel rather than weakening the quality of the whole sequence. If Seedance cannot preserve a text-heavy panel, use a protected-text composite and document the exception.

## Provider/API behavior

The skill may operate in either of two modes:

- **Seedance UI mode:** prepare numbered panel crops, prompts, durations, references, and a tracking checklist that can be pasted or uploaded into the user's Seedance workflow.
- **Official API mode:** use the currently available official Seedance/Volcano Engine API documentation and the user's configured credentials. Keep endpoint details and authentication out of this skill's static files because they can change.

Never invent an API endpoint, model ID, or parameter name. If the user has not supplied API access, generate the complete job plan and ready-to-paste prompts instead of pretending to submit jobs.

## Final assembly

Use FFmpeg or an equivalent local video tool to concatenate the generated clips. Normalize the clips first if they differ in frame rate, dimensions, pixel format, or audio presence. Preserve the requested output aspect ratio; if the source panels are wide and the output is vertical, use a soft background fill or a protected crop rather than cutting away dialogue.

The final handoff should include:

- the assembled video if generation completed;
- the editable panel manifest;
- all per-panel prompts;
- the list of successful and failed jobs;
- a short QC note identifying any panel that used a fallback or protected-text composite.
