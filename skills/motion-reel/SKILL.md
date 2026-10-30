---
name: motion-reel
description: "Produces code-rendered motion films (launch films, product ads, UI motion reels, showreels, data stories, typography pieces, music videos) where the agent writes a seekable program, renders it frame by frame in a headless browser, encodes with ffmpeg, then watches its own frames and fixes the worst defects. Covers the render rig and house rules, the production brief, borrowing a visual grammar from a reference, beat-by-beat state lists, closed-form springs, sound on the same clock, the contact sheet critique loop, approval gates for long runs, and multi-format export. Use this skill whenever the user asks for a launch video, product film, promo, showreel, motion graphics video, explainer, or music video made with code or an AI agent, says \"one prompt video\", or types /motion-reel, even when they only paste a product URL and a length. Always consult it before writing the brief, choosing references, animating, adding sound, or exporting."
---

# Motion Reel

A viral "one prompt" video is never one prompt. The prompt is roughly a tenth of the result. The harness around it does the rest.

**brief → reference → shot list → seekable render → contact sheet critique → fix the worst 3 → export.**

This skill has three parts. **Part A** is the setup and pre-production: rig, brief, reference, studio folder, gates. **Part B** is the motion system: the frame function, state lists, springs, and sound. **Part C** is critique and export. Work through A before writing any animation code, and never ship a render that has not been through C2.

Distilled from seeco's ([@seeconvm](https://x.com/seeconvm)) X article *"Opus 5.5 Motion Design: What the Viral 'One-Prompt' Videos Are Hiding"* ([source](https://x.com/seeconvm/status/2106096193235259564)). Full credit at the bottom.

## Scope

Apply to product films, UI motion, data stories, typography, launch explainers, showreels, and music videos rendered from code. When a rule here conflicts with a framework default, this file wins. When the user's explicit prompt conflicts with a rule, the user wins.

**Where code loses.** Photoreal scenes, complex characters, and organic movement still want footage or a video model. Say so, and offer generate-then-trace (C4) instead of forcing it into code.

**Companion skill:** `design-to-motion` in this repo, when the job is animating one static design into a showcase loop rather than making a film with a story, sound, and a CTA.

## The one-line run

Once Part A exists, a new film is one command:

```
/motion-reel for [URL], 20s, vertical, reference ./refs/frame.png
```

Parse it into: product URL, duration, format, reference. Ask only for what is missing, in one batch, then follow the parts below.

---

# PART A — Setup and pre-production

## A1. The rig

The chat app can write the animation. Only an agent with a shell can render it, listen to it, and look at its own frames. That feedback loop is the whole gap between the viral clips and the "it's a bit mid" replies, so this skill needs a shell agent (Claude Code in the source article).

```bash
brew install node ffmpeg python          # apt on Linux
pip install numpy librosa soundfile
mkdir motion-studio && cd motion-studio && npm init -y
npm i -D playwright && npx playwright install chromium
npx skills add remotion-dev/skills       # optional, framework route
npx skills add heygen-com/hyperframes    # optional, framework route
```

- **Default route:** one `index.html`, Playwright capturing frame by frame, ffmpeg encoding. Left alone, the model picks this zero-dependency route and skips Remotion and HyperFrames even when they're installed, so **name a framework if you want one**.
- **Effort:** use the highest reasoning effort the model offers for a new film (every viral one-shot ran on xhigh or max). Medium is for small fixes. Max when the first 3 seconds have to carry a launch.
- **Library choice:** spell out the look and the limits and let the model pick the technique.

## A2. House rules file

Drop these rules in the project root (`CLAUDE.md`, `AGENTS.md`, or your tool's rules file) so they never need repeating:

```markdown
## Render contract
- window.seek(t) paints frame t, nothing else drives the picture
- no CSS transitions, setTimeout or requestAnimationFrame in render mode
- seeded noise only, never Math.random

## Look
- banned: centered title on gradient, everything fading in, corner labels, glow on UI
- one display face, one UI face, one accent
- something new on screen every 2 to 4 seconds

## Sound
- score and SFX synthesized in code unless a track is supplied, -14 LUFS

## Before you show me anything
- render one frame per beat as a contact sheet, score it, fix the worst 3
```

## A3. The brief

A vibe prompt ("an insane product video") leaves every decision open, and the model fills each blank with the safest default it knows. That is where the centered gradient title comes from. When there is something to sell, write a production document before any render task:

```
FILM      one sentence the viewer keeps, audience, duration, formats
ASSETS    real logo, real UI screenshots, brand type and colors, audio
VISUAL    what to keep from the reference, which clichés to avoid
HARD LINE never invent product screens or metrics
DELIVER   storyboard, contact sheet, final video, poster frame, source
```

- **Small additions go far.** The product URL, a demand for real screenshots, logo and assets, and "must have music" are enough for the agent to go to the site and collect everything itself.
- **One session per brand.** A second film in the same session comes out faster because the renderer, audio synth, and export pipeline already exist.
- **Keys** go in `.env` and are referred to by name in the prompt, never pasted.
- **Rig test only.** The famous one-liner is for testing the rig once, then move on. Everyone who sends it gets reels that rhyme with each other ("brief contagion"):

  ```
  make a dynamic 15-second motion graphics video that shows what an incredible motion designer you are, like it's your showreel for a résumé. go all out.
  ```

## A4. The reference

With no visual grammar supplied, the model brings its own, and unrelated reels start looking like siblings. Naming a style works better than describing one, and a reference frame or video hands over pacing, type, and transitions in one go.

Three kinds of reference pull their weight: a single frame of a video you love, a short clip sliced with ffmpeg, or a folder of your own past work (a library nobody else can copy). Where to dig: whatships.com for launch videos, Dribbble motion, competitors' launch films.

```
Reference: ./refs/launch.mp4
1. Extract a frame every 0.5s with ffmpeg and study them
2. Write docs/style_guide.md: palette in hex, type, shot lengths, transitions, camera, grain
3. Write docs/shotlist.md for a [DURATION]s video about [SUBJECT] in that style
4. Take the grammar, never the subjects, logos or copy
5. Show me both files and wait for my OK before any code
```

## A5. Real assets only

The model can invent a camera move. It must never invent your dashboard.

- For a product film, pull the real screens from the live site and put the asset list in front of the user for review.
- If something is missing, **stop and ask** instead of drawing a convincing fake. A missing interface screenshot is a blocker.

## A6. The studio folder

If the only record of the film lives in a chat, you had a lucky session. Keep every decision in files the next run can read:

```
motion-studio/
  brief.md         goal and product facts
  style_guide.md   palette, type, motion, banned defaults
  shotlist.md      beats, states, timestamps, purpose
  assets/          approved logos, screens, sound
  src/             seekable composition and motion code
  reviews/         contact sheets and defect notes
  out/             one export per format
```

For the next launch, swap the brief and the assets and keep everything else.

## A7. Gates

Long and overnight runs need gates. The agent can work for hours between gates, but does not pass one without approval:

```
GATE 1  references and product assets approved
GATE 2  style guide and shot list approved
GATE 3  stills for every shot, critiqued
GATE 4  animatic at 960x540 with placeholder audio
GATE 5  full pass, polish, sound
GATE 6  formats, loop check, poster frame, README
```

An unclear reference gets a question or one test frame, never 900 polished wrong ones.

---

# PART B — Motion system

## B1. The frame function

The model cannot output an MP4. It writes a program, and a headless browser turns it into pictures. The whole program answers one question:

```
seek(t) → paint exactly what the screen shows at second t
```

The browser calls it once per frame (900 times for 15 seconds at 60fps), screenshots each frame, and ffmpeg glues them into a file.

- **Seekability is the one rule you can't bend.** Any frame renders on its own, the same way every time. Frame 240 never needs frames 0 to 239 replayed.
- Nothing runs on a timer: no CSS transitions, `setTimeout`, or `requestAnimationFrame` in render mode. Seeded noise only.
- Payoff: a title that lands too early is one number to change plus a re-render.

## B2. State lists, not effects

"Zoom in, morph, add particles" is a list of activity. A storyboard says what the viewer knows at each second:

```
00-02s  HOOK     one image that earns attention
02-05s  PROBLEM  the friction, shown on screen
05-09s  REVEAL   the product enters through a real interaction
09-13s  PROOF    one screen or number backs the claim
13-16s  CLOSE    a clean final state and a readable CTA
```

- **Every beat has an entry state, an exit state, and a reason to be there.** If the agent can't explain why shot four exists, shot four gets cut.
- **One shape that never cuts.** A single container morphs size, radius, and color from state to state, a cursor drives every change with real clicks, and the final frame matches the first so the clip loops:

  ```
  logo → CTA button → email field (typed) → loader → success check → dashboard → chart draws itself → tooltip → ⌘K palette → toast → logo
  ```

- **The story version:** code a fake UI (for example a video editor with a timeline, waveform, and export button) and stage the whole film inside it. A fake UI works like a film set where every element has a known state.

## B3. Springs

A fixed easing curve slides from A to B. Motion with mass speeds up, overshoots a hair, and settles. Use closed-form springs: they give that feel while staying a pure function of time, so `seek(t)` stays deterministic.

When a value changes target several times (a cursor, a container's width), stack one spring per change instead of restarting it:

```
track(t) = v0 + Σ (v_i − v_(i−1)) × spring(t − t_i)
```

That way frame 812 still renders without simulating frames 0 to 811.

**Each class of object gets its own rule:**

| Object | Behaviour |
|---|---|
| Small UI | snaps, minimal overshoot |
| Panels | settle |
| Camera | glides almost invisibly |
| Headlines | land hard, hold long enough to read |
| Mascots | bounce on purpose |

If everything overshoots, nothing feels precise.

## B4. Sound on the same clock

Sound runs on the same clock as the picture. Score and SFX are synthesized in code unless a track is supplied, mixed to -14 LUFS.

With a supplied track, measure it before you animate:

```bash
python beats.py song.wav > beats.json   # librosa: bpm, beats, downbeats, hits
```

- State changes go on beats, big moments on downbeats, clicks and whooshes on hits.
- **The blindfold test:** close your eyes and listen to the film. If the audio has no shape without the picture, the sound design is unfinished.

---

# PART C — Critique and export

## C1. Build the evidence

The critique loop is the step people skip because the first render feels like progress, and it's the step every viral clip had. Don't watch a full export ten times. Have ffmpeg build the evidence, saved to `reviews/`:

```
contact sheet   2 frames per second, 6 across
strip           12 consecutive frames around the fastest move
phone test      frames scaled to 360 px wide
loop check      the clip played twice back to back
determinism     render frame 300 twice, the hashes must match
```

## C2. The critique

The model reads images, so it inspects what it just rendered. Give it a critique task with teeth:

```
Judge only the rendered frames, ignore what you meant to make
Score 1-10: hook in 2s, phone readability, motion, variety, sound sync, brand accuracy
List the 3 worst defects with timestamp, evidence and a local fix
Patch those seconds, re-render them, repeat until every score is 8+
```

What to hunt for: overlapping text, elements sliding where they should ease, corner labels, blurry scaled type, a dead beat with nothing happening, a stutter at the loop point.

## C3. Formats

Every aspect ratio comes from the same timeline, **never from a crop**.

| Format | Adjust |
|---|---|
| Vertical | bigger type, fewer elements |
| Square | tighter holds |
| Wide | more spatial context |

Deliver one export per format in `out/`, plus the poster frame, the contact sheet, and the source.

## C4. Generate-then-trace (when code alone loses)

For characters and physics: a video model renders base shots, then the agent redraws the whole film in JavaScript on top, so viewers only see the code-drawn layer. For long multi-chapter films, the subagent version has the agent write an `ANIMATION_GUIDE.md` that briefs parallel workers per chapter (see [JohnHeibel/PDoomVideo](https://github.com/JohnHeibel/PDoomVideo)).

---

# Output format

Before writing code (Gate 2), state:

1. **Film** — the one sentence the viewer keeps, audience, duration, formats
2. **Assets** — what was collected, and what is missing (a blocker, not a fake)
3. **Reference** — the source and the grammar taken from it (`style_guide.md`)
4. **Shot list** — each beat with timestamps, entry state, exit state, and purpose
5. **Sound** — synthesized or supplied track, and how beats map to state changes

After rendering, report:

- The critique scores per category and the 3 defects fixed in the last round
- Determinism hash check and loop check results
- Each export with format, dimensions, frame rate, and duration
- Anything invented or not reproduced from real assets, named plainly

---

# Make it yours

Every default lives here. Edit this section and the rest of the skill follows.

| Value | Default |
|---|---|
| Frame rate | 60fps |
| Something new on screen | every 2 to 4 seconds |
| Shots in 15 seconds | 6 to 8 |
| Faces and colour | one display face, one UI face, one accent |
| Banned defaults | centered title on gradient, everything fading in, corner labels, glow on UI |
| Loudness | -14 LUFS |
| Reference frame extraction | one frame every 0.5s |
| Animatic | 960×540 with placeholder audio |
| Contact sheet | 2 frames per second, 6 across |
| Motion strip | 12 consecutive frames around the fastest move |
| Phone test width | 360px |
| Determinism check | frame 300 rendered twice, hashes match |
| Critique pass mark | every score 8+ |
| Defects fixed per round | worst 3 |

---

# Quick checklist

**Setup**
- [ ] Shell agent with Playwright and ffmpeg; framework named if one is wanted
- [ ] House rules file in the project root
- [ ] Highest effort for a new film
- [ ] Studio folder with brief, style guide, shot list, assets, reviews, out

**Pre-production**
- [ ] Production brief written, with a hard line against invented screens or metrics
- [ ] Reference supplied; style guide takes its grammar, never its subjects, logos, or copy
- [ ] Real assets collected and reviewed; missing assets raised as blockers
- [ ] Gates 1 and 2 approved before any code

**Motion and sound**
- [ ] `seek(t)` drives every pixel; no timers, seeded noise only
- [ ] Every beat has entry state, exit state, and purpose
- [ ] Springs stacked per target change; each object class has its own rule
- [ ] State changes on beats, big moments on downbeats; blindfold test passes

**Critique and export**
- [ ] Contact sheet, strip, phone test, loop check, determinism check in `reviews/`
- [ ] Critique scored from rendered frames only; every score 8+
- [ ] Every format from the same timeline, none cropped
- [ ] Poster frame, source, and README delivered

---

# Credits

Distilled from **seeco** ([@seeconvm](https://x.com/seeconvm))'s X article *"Opus 5.5 Motion Design: What the Viral 'One-Prompt' Videos Are Hiding"*, posted October 2, 2026: https://x.com/seeconvm/status/2106096193235259564. The pipeline, the `/motion-reel` command, the house rules, the brief template, the reference prompt, the beat structure, the spring formula, the evidence list, the critique prompt, the studio folder, and the gates are the author's, reproduced here with light edits for a general audience. The article itself credits the community examples it draws on, including @twoclipping, @verbove, @oozn, @mablesjoseph, and @pradeepXkapoor.

Starter repos the article recommends: [JohnHeibel/PDoomVideo](https://github.com/JohnHeibel/PDoomVideo) for the music-video pipeline and [JohnHeibel/ClaudeAnimationBase](https://github.com/JohnHeibel/ClaudeAnimationBase) as a starter.
