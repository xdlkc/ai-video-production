# ai-video-production

English | [简体中文](./README.zh-CN.md)

> Turn a single idea into a finished AI short film using a multi-model pipeline: **Claude as Director + Codex as Illustrator + Seedance as Animator**.

This repository packages a [`SKILL.md`](./SKILL.md) skill for Claude Code, Codex, Cursor, Gemini CLI, and other Agent Skills-compatible agents. Its single goal: **produce a finished AI video you are happy with at minimum spend and minimum re-roll rate.**

## Why This Pipeline Exists

The real cost in AI video production is not model subscriptions — it is **blind re-rolling**. Editing a line of prompt text costs nothing, regenerating a still frame costs almost nothing, whereas **video generation is billed per second**. The pipeline enforces a strict cost-escalation ladder:

```text
Errors caught in text          ->  never leak into images
Errors caught in images        ->  never burn video generation credits
Errors fixable in video code   ->  never get re-shot (trim / patch in ffmpeg or Python)
Mandatory re-shoots            ->  revise prompt or start/end frames first, re-shoot only that shot
```

## Role Split

The user only talks to one interface — the **Director** — while models and tools split responsibilities:

| Role | Responsibility |
| --- | --- |
| **User** | Picks direction, confirms visual style, approves spend; reviews actual motion and audio. |
| **Director** (Claude Code / main agent) | Writes story, shot list, and all prompts; invokes Codex & Seedance; extracts and zooms frames for inspection; runs code-based post-production. |
| **Illustrator** (Codex) | Generates character/prop sheets, style frames, and exact start/end keyframes via `codex exec --skip-git-repo-check -s read-only`. |
| **Animator** (Seedance) | Animates start/end keyframe pairs into video clips via Volcengine Ark (`doubao-seedance-2-5-260628` or `doubao-seedance-2-0-mini-260615`). |
| **Tools** (`ffmpeg` / Python) | Concatenation, motion-blurred transitions, color matching, upscaling, and beat-synced cuts. |

For maximum shot pass rates, [`SKILL.md`](./SKILL.md) also defines an optional **4-Thread Mode** where the Director spawns three read-only subagents (**Reviewer**, **Supervisor**, and **Proxy**) alongside the **Worker** to catch mistakes and enforce user preferences within a fixed budget before spending money.

## Eight-Step Pipeline

1. **Idea -> Locked Story**: Realistic causal checks, one-sentence theme, strong opening hook, sequential human reactions, and avoiding known AI video weaknesses.
2. **Story -> Shot List & Color Sheet**: Shot-by-shot camera angles, motion motivation, 38%/62% composition, 1920x1080 color-block storyboards, and grayscale contrast verification.
3. **Style & Character/Prop Sheets**: Lock art direction and fixed character/prop anchors before generating formal shots.
4. **Start/End Keyframes**: Draft frame specs -> review -> generate the denser frame first (usually end frame) -> derive the paired frame for camera lock -> crop to matching 16:9 dimensions -> run the 8-point zoomed inspection checklist.
5. **7-Segment Video Prompts + Pre-Submission Review**: Structured prompt (`Style`, `Camera`, `Composition & Palette`, `Characters`, `Second-by-Second`, `Audio`, `Constraints`) checked against [`references/pitfalls.md`](./references/pitfalls.md) and approved by a fresh-context **Reviewer agent** before any paid API call.
6. **Video Generation**: Generate at 720p (upscale later), test the hardest shot first, keep all old takes, and check audio tracks before keeping them.
7. **Code-Only Post-Production**: Handle BT.601 -> lossless intermediate -> BT.709 color management, segment color alignment, 8–16 slice motion-blur transitions, lanczos 1080p upscaling, and beat-syncing from the original audio track via `ffmpeg` and Python.
8. **Blind Full-Film Audit & Delivery**: A fresh-context agent decodes frames across every shot and cut boundary without prior conclusions, distinguishes blockers from minor issues, and flags motion/audio checks for the user.

Project artifacts are organized into a dedicated folder layout: `01-story/`, `02-shotlist/`, `03-style/`, `04-frames/`, `05-prompts/`, `06-clips/`, `07-final/`.

## Installation

Install using [`npx skills`](https://github.com/vercel-labs/skills) to symlink the skill across detected agents (Claude Code, Codex, Cursor, Gemini CLI, etc.):

```bash
# Install into the current project (default project scope)
npx skills add xdlkc/ai-video-production -y

# Install globally across projects
npx skills add xdlkc/ai-video-production -g -y

# Install this skill and symlink to all detected agents
npx skills add xdlkc/ai-video-production --skill ai-video-production --all
```

Run once without persistent installation:

```bash
npx skills use xdlkc/ai-video-production@ai-video-production
```

Manage installed skills:

```bash
npx skills list
npx skills update -y
npx skills remove ai-video-production -y
```

You can also point any agent directly to [`SKILL.md`](./SKILL.md) without installing.

### Prerequisites

- **Codex CLI** (image generation)
- **`ffmpeg`** and **Python** (frame extraction, color/layout checks, post-production)
- **`ARK_API_KEY`** environment variable (Volcengine Ark Seedance API; never printed or committed)

## Repository & References

```text
.
├── SKILL.md                    # Core pipeline, 8 non-negotiable rules, 4-thread mode, checklist
├── references/
│   ├── prompts.md              # 9 copy-paste prompt templates (director kickoff through 4-thread)
│   └── pitfalls.md             # Model failure modes vs. verified prompt phrasings & post fixes
├── README.md
├── README.zh-CN.md
└── LICENSE                     # MIT License
```

`SKILL.md` stays concise while long templates and troubleshooting tables live in [`references/prompts.md`](./references/prompts.md) and [`references/pitfalls.md`](./references/pitfalls.md), loaded on demand to avoid bloating context.

## Known Boundaries

- The director agent **cannot perceive true video motion or hear audio** — final pacing, motion feel, and soundtrack must always be verified by the user.
- Timing inside generation prompts is approximate; final rhythm is achieved via code-based trimming and time-remapping in post-production.
- Known model weaknesses (crossing characters, small handheld props, text, mirrors/reflections, lighting changes mid-shot, simultaneous multi-person actions) are cataloged with workarounds in [`references/pitfalls.md`](./references/pitfalls.md).

## License

[MIT](./LICENSE)
