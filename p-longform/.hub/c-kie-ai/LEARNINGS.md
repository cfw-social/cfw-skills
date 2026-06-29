# c-kie-ai Learnings

> This file is the self-learning loop for `c-kie-ai`. Before executing this skill, the agent reads this file and applies all accumulated `Active Feedback`. After execution, the agent asks the user for feedback and appends it here.

---

## Active Feedback (apply on every run)

*None yet — add feedback below and it becomes part of this skill's behavior.*

---

## Feedback Log

### 2026-05-16 — Image editing via c-kie-ai
- **kie.ai credit depletion**: Multiple models (qwen/image-edit, qwen/image-to-image, google/nano-banana) returned 402 "Credits insufficient" — account may need top-up before next run.
- **flux-2/flex-image-to-image resolution**: API accepted "hd" as a valid resolution value once (hit 429 rate-limit), then inconsistently rejected it as "not within range". Flux-2 is not reliable for precise editing.
- **fal.ai fallback works for image-to-image**: `fal.run/fal-ai/flux/dev/image-to-image` successfully processed a hand-removal + floating-object edit with public URL input. Use fal.ai when kie.ai is out of credits or when precise image-to-image is needed.
- **Model input format notes**:
  - flux-2/flex-image-to-image: accepts `input_urls` with public HTTP URLs (not base64)
  - gpt-image/1.5-image-to-image: accepts `input_urls` with data URIs, but returned "File type not supported" for both JPEG and PNG base64 on 2026-05-16 — model may be temporarily broken or requires a different prefix format
  - qwen/image-edit and qwen/image-to-image: accept `image_url` with public URL, but require credits

### 2026-05-08 — Initial template
- Skill created. No feedback yet.

