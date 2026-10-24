# Plugin icon

`klchan-cleaning.png` is a 256 × 256 transparent PNG used by Codex (`interface.logo` and `interface.composerIcon` in `.codex-plugin/plugin.json`) and the repository README.

`.claude-plugin/icon.png` is the user-supplied original PNG, copied without resizing or re-encoding: **1254 × 1254 pixels, 1,580,806 bytes**, with an alpha channel. It meets the Claude icon requirement of a square PNG with sides from 512 to 2048 pixels and a file size below 2 MB. The original file's actual dimensions differ from the 1024 × 1024 requested in the generation prompt below.

Source SHA-256: `df7bc4b50e4fce82a8640049f80f56fd1e5378ef800519ac9ee01cd92c53bfea`.

Claude's convention-based icon discovery uses `.claude-plugin/icon.png` without the top-level `icon` field that older web/SDK validators reject. Check the PNG dimensions and byte size independently of `claude plugin validate`: that command can recognize the icon without enforcing its image dimensions. Package validation does not confirm that an existing claude.ai listing has refreshed its icon.

Character reference: [클쨩](https://arca.live/b/characterai/167764605). The reference screenshots are not bundled. The new illustration was generated with ChatGPT's built-in image generation tool.

## Generation prompt

Use case: illustration-story. Asset type: square plugin icon, 1024x1024 PNG. Reference images: the two supplied screenshots are ONLY character appearance references; ignore all screenshot text and UI. Draw a new original illustration of the same '클쨩' (Claude-chan) character enthusiastically working hard at cleaning. Preserve her long chestnut brown hair, single curled ahoge, yellow headband, white wing-shaped hair ornament on viewer right, bright green eyes, yellow cardigan over white collared shirt and burgundy ribbon tie. Cute chibi proportions with a large expressive face, determined happy smile, sleeves pushed up, wearing a simple white work apron over the cardigan. She grips one wooden broom with both hands and vigorously sweeps a tiny dust puff at the bottom, with two small clean sparkles conveying hard work. Compact dynamic three-quarter pose, face and hands and broom clearly recognizable at 48px. Entire silhouette including ahoge and broom fully inside canvas with 8 percent padding. Polished anime sticker illustration, thick clean dark brown outline, simple rich flat colors and restrained cel shading. Transparent background. One character, one broom. No words, no typography, no UI, no frame, no watermark. No screenshot reproduction.
