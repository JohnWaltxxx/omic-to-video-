# Seedance job manifest

Keep panel segmentation, prompts, provider settings, and review status in a machine-readable file. This file must not contain API keys or access tokens.

\`\`\`json
{
  "version": 2,
  "source": {
    "name": "halloween-sheep.png",
    "width": 1024,
    "height": 1536,
    "readingDirection": "ttb-ltr"
  },
  "output": {
    "width": 1080,
    "height": 1920,
    "fps": 30,
    "format": "mp4"
  },
  "panels": [
    {
      "id": "page-01-panel-01",
      "order": 1,
      "source": "panels/page-01-panel-01.png",
      "crop": {"x": 0.018, "y": 0.004, "width": 0.965, "height": 0.244},
      "durationSeconds": 4,
      "promptFile": "seedance/prompts/page-01-panel-01.txt",
      "references": [],
      "mode": "independent-panel",
      "provider": {
        "name": "seedance",
        "model": "provider-configured",
        "jobId": null,
        "status": "planned"
      },
      "review": {
        "status": "not-reviewed",
        "issues": []
      }
    }
  ]
}
\`\`\`

## Status values

- \`planned\`: prompt and crop are ready;
- \`submitted\`: provider accepted the job;
- \`completed\`: clip was downloaded;
- \`needs-regeneration\`: clip failed a quality gate;
- \`approved\`: clip is ready for assembly.

## Prompt file format

Each prompt file should contain:

\`\`\`text
SOURCE LOCK:
Use the input comic panel as the exact visual reference. Preserve the same characters, faces, clothing, colors, setting, composition, linework, and illustration style.

ACTION:
Describe the primary movement in one or two sentences.

SECONDARY MOTION:
Describe no more than three supporting motions.

CONTINUITY:
Keep the panel framing, speech balloons, captions, and Chinese lettering unchanged and legible.

NEGATIVE CONSTRAINTS:
No new characters, no redesign, no scene change, no extra limbs, no deformed hands, no text changes, no random cuts, no generated subtitles.
\`\`\`
