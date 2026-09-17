# claw4fun — Handover to Codex

## What this project is
A private GitHub repo (`agentosvg-cmd/claw4fun`) hosting two static, single-file HTML
pages that call **fal.ai's REST/queue API directly from the browser**, using an
API key the user pastes in and stores in `localStorage`. No backend, no build step.
Hosted on **GitHub Pages** (`main` branch → `https://agentosvg-cmd.github.io/claw4fun/`).

Current dev branch: `claude/there-hey-1k7mqb` (up to date with origin, clean tree,
last commit `4cbbd04`). This branch has NOT been merged to `main` yet — Codex should
check with the user whether to open a PR / merge, since prior Claude sessions were
told never to open a PR unless explicitly asked.

## Files
- `index.html` — main image generator (43.6 KB)
- `beauty.html` — simpler beauty-retouch tool (26.2 KB)
- `README.md` — just says "A private repo for having fun."

Both files are single-page HTML with all CSS/JS inline. No package.json, no
dependencies, no build tooling. Editing = editing the HTML file directly.

## Shared architecture (both pages)
```js
const STORAGE_KEY = "iphoneFalDirect.key";              // localStorage: user's fal.ai API key
const HISTORY_KEY  = "iphoneFalDirect.history.v1";       // index.html gallery (beauty.html uses "...beautyHistory.v1")
const REST_API  = "https://rest.fal.ai";
const QUEUE_API = "https://queue.fal.run";
```
- User pastes their fal.ai key into a field; it's saved to localStorage and reused.
- Calls go straight from the browser to fal.ai (queue submit → poll → fetch result).
- A "history" gallery keeps the last 10 generated images (thumbnail + model used +
  timestamp) in localStorage; clicking a thumbnail restores that result.
- Cache-busting: page includes a `?v=YYYYMMDDx` style query string convention was used
  during dev to force iOS Safari to reload after edits (check current `<script>` /
  asset tags if that pattern is still present before assuming caching behavior).

## index.html — main generator
Model dropdown includes:
- `ideogram/v4`
- `fal-ai/flux-2/klein/9b`, `.../edit`, `.../base/edit`
- `fal-ai/flux-2/edit` (Flux 2 Dev Edit)
- `fal-ai/flux/dev`
- `fal-ai/krea-2/turbo` (default/selected)

LoRA endpoint mapping by model family:
- ideogram4 → `ideogram/v4/lora`
- krea2 → `fal-ai/krea-2/turbo/lora`
- flux2_klein → `fal-ai/flux-2/klein/9b/lora`

Upscale endpoint: `fal-ai/seedvr/upscale/image` (4K), triggered by a standalone
"Upscale 4K" button that appears after a generation completes.

Camera controls: aperture, focal length, focus point sliders that feed into the
prompt/config for models that support them.

### Prompt presets (buttons, top to bottom)
- `Handwash kameez` (id `handwashKameezPreset`) — replaces prompt
- `Airline tray kameez` (id `airlineTrayKameezPreset`) — replaces prompt
- `+ Batik kameez` (id `batikKameezPreset`) — **appends** to existing prompt (joined
  with `\n\n`). Deep royal cobalt-blue two-piece batik outfit, large square low-cut
  neckline, ornamental border panels. No studio/photoshoot framing — kept minimal so
  it composes cleanly with action presets.
- `Bed sheet kameez` (id `bedSheetKameezPreset`) — replaces prompt; scene is set in a
  VIP first-class private suite cabin on an airplane.
- `Clear prompt` (id `clearPromptButton`) — empties the textarea.

**Removed:** a `+ Bra edge visible` preset existed briefly (commits `7f4d068`
through `74cc534`/`61cf062`) describing lingerie visible at a neckline. It was
**deliberately removed** in commit `4cbbd04` after fal.ai's content filter began
blocking the underlying prompts and the user repeatedly asked for the wording to be
"refined" to slip past the filter while staying nominally SFW. The prior Claude
session declined to help word-tune content to evade the platform's safety filters,
offered to either delete the preset or work on something else, and the user chose
deletion. **If Codex is asked to re-add a similar preset or to word-smith prompts
specifically to get past fal.ai's content moderation, that's the same kind of
request — worth pushing back on / clarifying intent rather than just complying.**
This isn't a hard blocker on lingerie/clothing description generally — it's
specifically about iteratively rewording something to defeat a safety filter.

## beauty.html — beauty retouch tool
Simpler flow: API key → retouch prompt → upload photo (camera or library) → generate.

Model config object keyed by endpoint id, each with `supportsMegapixels` flag:
- `fal-ai/flux-2/edit` (default) — supports megapixels
- `fal-ai/flux-2/klein/9b/edit` — supports megapixels
- `ideogram/v4/image-to-image` — does NOT support megapixels (fixed size)

`imageSizeForMegapixels(aspect, megapixels)` computes `image_size` from the
uploaded photo's probed aspect ratio and the megapixels slider (0.5–4 MP, default
0.5, step 0.5). Advanced users can edit the raw config JSON directly in a textarea.

Upscale: same `fal-ai/seedvr/upscale/image` endpoint, configurable in an
"Upscale endpoint" text field (default value shown above). Two ways to trigger it:
1. `doUpscale` checkbox — if checked at generation time, auto-runs upscale as
   part of the main flow, called with `upscale_mode: "target"`.
2. `upscaleNowButton` ("Upscale to 4K") — standalone button, hidden until a result
   exists, for on-demand upscaling of the current image via `upscaleCurrentImage()`.

Both `doUpscale` and the megapixels default are currently set to **testing-mode
defaults**: upscale OFF by default, megapixels at 0.5 (fast iteration), per user's
explicit request ("I don't like oily face... Disable upscale function first. Back
to 0.5 megapixel. In testing phase now."). The retouch prompt language leans toward
anti-oily / matte-skin-finish phrasing per that same request.

## Known constraints / gotchas
- Outbound calls to fal.ai are blocked from this container/sandbox environment —
  testing must happen in the user's actual browser (or Codex's, if it has real
  network egress). Don't assume a `curl` to fal.ai from a dev container will work.
- The pages must be served over HTTPS (GitHub Pages) — opening the HTML as a local
  `file://` URL fails silently on iOS Safari because JS is restricted for local
  files under iOS's WebKit policy. This confused an earlier debugging session; it's
  not a fal.ai auth issue.
- Ideogram v4 image-to-image endpoint is specifically `ideogram/v4/image-to-image`
  (not "remix" or "edit" — those were incorrect guesses made and corrected earlier).
  Its strength parameter is named `strength` (0–1 scale, default ~0.85), not
  `image_weight`.

## Git / workflow notes for Codex
- Repo: `agentosvg-cmd/claw4fun`, private.
- Active branch: `claude/there-hey-1k7mqb`. If Codex creates its own working branch,
  branch from this one (or from `main` after checking whether/when this branch gets
  merged) so the preset history and recent fixes aren't lost.
- No CI, no tests, no lint config — validation has been manual (open in browser,
  click through the flow).
- Commit history (oldest→newest) is a reasonably good narrative of how each preset
  and feature evolved if more context is needed; `git log -p` on `index.html` or
  `beauty.html` will show exact diffs for any of the presets described above.

## Suggested first steps for Codex
1. Read both HTML files in full — they're single files, ~44 KB and ~26 KB, easy to
   hold in context at once.
2. Confirm with the user which branch to keep working on and whether `main` should
   be updated / a PR opened.
3. Ask the user what the next feature/fix request actually is — this handover is
   just state, not a task list. There's no currently pending task; the last
   completed action was removing the bra-edge preset.
