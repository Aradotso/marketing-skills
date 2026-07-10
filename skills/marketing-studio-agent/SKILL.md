---
name: marketing-studio-agent
description: Agent-driven marketing studio that generates complete launch asset suites (logo reveals, demos, launch videos, social clips, OG assets) from one /marketing command
triggers:
  - generate marketing assets for my product
  - create a launch video suite
  - render logo reveal and product demo
  - build social media clips for launch
  - generate OG images and animated loops
  - create complete marketing package
  - film product demo with camera effects
  - make launch announcement videos
---

# Marketing Studio Agent

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Marketing Studio is an agent-driven video rendering engine for Claude Code. Run `/marketing` from your product repo to generate a complete launch asset suite: 3D logo reveal, screen-captured product demo, hero launch video with voiceover/music, per-platform social clips, and OG assets. All rendering happens in a central engine repo using Remotion, Blender, Playwright, and ElevenLabs.

## Installation

```bash
# Clone the engine repo
git clone https://github.com/ucsandman/marketing-studio.git
cd marketing-studio

# Install dependencies
cd studio && npm install && cd ..

# Configure (optional: Blender path, ElevenLabs API key)
cp .env.example .env

# Install skills into Claude Code
node scripts/install-skills.mjs

# Verify toolchain
python launch.py --check
```

The install script copies skills to `~/.claude/skills` and links them to your cloned engine path.

## Core Workflow

### The `/marketing` Command

From your product repo:

```bash
claude
> /marketing
```

This runs the full pipeline:
1. **Brand onboarding** – Derives brand tokens from your design system (or prompts for them)
2. **Logo reveal** – 3D Blender animation composited in Remotion
3. **Product demo** – Playwright captures your running app with camera zooms
4. **Launch video** – 30-90s hero video combining demo, logo, copy
5. **Audio track** – ElevenLabs voiceover and music scored to video
6. **Social clips** – Platform-specific cuts (X, LinkedIn, TikTok)
7. **OG assets** – Static image, animated loop, README GIF

### Individual Skills

Run any asset type standalone:

```bash
> /logo-reveal
> /product-demo
> /launch-video
> /audio-track
> /social-clip
> /og-assets
```

## Brand Configuration

Brands are defined in `brands/<id>.json`:

```json
{
  "id": "myproduct",
  "name": "MyProduct",
  "tagline": "Ship faster, break less",
  "colors": {
    "primary": "#3b82f6",
    "secondary": "#8b5cf6",
    "accent": "#f59e0b",
    "background": "#000000",
    "foreground": "#ffffff",
    "muted": "#6b7280",
    "success": "#10b981",
    "warning": "#f59e0b",
    "error": "#ef4444",
    "text": "#f9fafb",
    "textMuted": "#9ca3af",
    "border": "#374151",
    "surface": "#111827"
  },
  "fonts": {
    "heading": "Inter",
    "body": "Inter",
    "mono": "JetBrains Mono"
  },
  "voice": {
    "tone": "confident, technical, no fluff",
    "avoid": ["AI slop", "game-changer", "revolutionary"]
  }
}
```

Register the brand in `studio/src/lib/brand.ts`:

```typescript
import { z } from 'zod';

export const BrandSchema = z.object({
  id: z.string(),
  name: z.string(),
  tagline: z.string(),
  colors: z.object({
    primary: z.string(),
    secondary: z.string(),
    accent: z.string(),
    // ... all 13 colors
  }),
  fonts: z.object({
    heading: z.string(),
    body: z.string(),
    mono: z.string(),
  }),
  voice: z.object({
    tone: z.string(),
    avoid: z.array(z.string()),
  }),
});

export type Brand = z.infer<typeof BrandSchema>;

export function getBrand(brandId: string): Brand {
  const brandData = require(`../../brands/${brandId}.json`);
  return BrandSchema.parse(brandData);
}
```

Add a logo mark component in `studio/src/brands/<id>Mark.tsx`:

```typescript
import React from 'react';

export const MyProductMark: React.FC<{
  size?: number;
  color?: string;
}> = ({ size = 100, color = '#3b82f6' }) => {
  return (
    <svg width={size} height={size} viewBox="0 0 100 100">
      <circle cx="50" cy="50" r="40" fill={color} />
    </svg>
  );
};
```

## Manual Rendering

### Remotion Commands

Render any composition directly:

```bash
cd studio

# Logo reveal
npx remotion render LogoReveal ../out/myproduct/logo.mp4 \
  --props='{"brandId":"myproduct","cta":"Watch the launch"}'

# Product demo
npx remotion render ProductDemo ../out/myproduct/demo.mp4 \
  --props='{"brandId":"myproduct","captureVideo":"../assets/captures/demo.mp4"}'

# Launch video
npx remotion render LaunchVideo ../out/myproduct/launch.mp4 \
  --props='{"brandId":"myproduct","demoVideo":"../out/myproduct/demo.mp4","script":["Hook","Problem","Solution"]}'

# Social clip
npx remotion render SocialClip ../out/myproduct/social-x.mp4 \
  --props='{"brandId":"myproduct","platform":"x","feature":"Real-time sync"}'

# Animated OG
npx remotion render AnimatedOG ../out/myproduct/og.mp4 \
  --props='{"brandId":"myproduct"}'
```

### Feeder Scripts

#### Capture Product Demo

```bash
cd feeders/capture
node capture.mjs \
  --url http://localhost:3000 \
  --output ../../assets/captures/demo.mp4 \
  --brand myproduct \
  --script zoom-dashboard,click-feature,pan-results
```

#### Render Blender Logo

```bash
cd feeders/blender
python render_logo.py \
  --brand myproduct \
  --output ../../assets/blender/logo-frames/ \
  --style draw-on
```

#### Generate Audio

```bash
cd feeders/audio
node generate.mjs \
  --script "Ship your product in days, not months" \
  --voice-id "ELEVENLABS_VOICE_ID" \
  --output ../../assets/audio/voiceover.mp3

node generate.mjs \
  --type music \
  --mood energetic \
  --duration 60 \
  --output ../../assets/audio/music.mp3
```

Set `ELEVENLABS_API_KEY` in `.env`.

## Props Builders

Generate render props via builder scripts:

```bash
# Build launch video props
node scripts/build-launch-props.mjs \
  --brand myproduct \
  --demo-video ./assets/captures/demo.mp4 \
  --script "Hook line" "Problem statement" "Solution demo" \
  --output ./props/launch-myproduct.json

# Build social clip props
node scripts/build-social-props.mjs \
  --brand myproduct \
  --platform x \
  --feature "Real-time collaboration" \
  --output ./props/social-x-myproduct.json
```

Props files are JSON:

```json
{
  "brandId": "myproduct",
  "demoVideo": "./assets/captures/demo.mp4",
  "logoRevealVideo": "./out/myproduct/logo.mp4",
  "script": [
    {"type": "hook", "text": "Ship in days, not months", "duration": 3},
    {"type": "problem", "text": "Marketing assets take weeks", "duration": 4},
    {"type": "solution", "text": "One command, full suite", "duration": 5}
  ],
  "voiceover": "./assets/audio/voiceover.mp3",
  "music": "./assets/audio/music.mp3"
}
```

## Environment Variables

```bash
# .env
BLENDER_PATH=/Applications/Blender.app/Contents/MacOS/Blender
ELEVENLABS_API_KEY=your_api_key_here
COMFYUI_URL=http://localhost:8188
```

All are optional; feeders degrade gracefully without them.

## Verification & Testing

```bash
# Full toolchain health check
python launch.py --check

# Render frame 0 of every composition (smoke test)
node scripts/smoke.mjs

# Brand schema validation
cd studio && npm test

# Launch Remotion Studio for interactive preview
python launch.py
```

## Key Patterns

### Resume on Failure

The pipeline writes a run manifest to `.marketing-run.json`:

```json
{
  "brand": "myproduct",
  "timestamp": "2026-07-10T12:00:00Z",
  "completed": ["logo-reveal", "product-demo"],
  "current": "launch-video",
  "props": {
    "logoRevealVideo": "./out/myproduct/logo.mp4",
    "demoVideo": "./out/myproduct/demo.mp4"
  }
}
```

Re-running `/marketing` resumes from the last incomplete step.

### Camera Zoom Math

Product demo captures use measured easing for professional zooms:

```typescript
// studio/src/lib/camera.ts
export function calculateZoom(
  frame: number,
  duration: number,
  startScale: number = 1,
  endScale: number = 1.2
): number {
  const progress = frame / duration;
  const eased = 1 - Math.pow(1 - progress, 3); // cubic ease-out
  return startScale + (endScale - startScale) * eased;
}
```

### Seamless Loop Rules

Animated OG loops must start and end on identical frames:

```typescript
// studio/src/compositions/AnimatedOG.tsx
export const AnimatedOG: React.FC<{brandId: string}> = ({brandId}) => {
  const frame = useCurrentFrame();
  const {durationInFrames} = useVideoConfig();
  const brand = getBrand(brandId);
  
  // Normalize to [0, 1) for seamless loop
  const progress = (frame % durationInFrames) / durationInFrames;
  const rotation = progress * 360;
  
  return (
    <AbsoluteFill style={{backgroundColor: brand.colors.background}}>
      <div style={{transform: `rotate(${rotation}deg)`}}>
        {/* logo mark */}
      </div>
    </AbsoluteFill>
  );
};
```

### Platform-Specific Social Clips

Aspect ratios and safe zones per platform:

```typescript
// studio/src/compositions/SocialClip.tsx
const PLATFORM_SPECS = {
  x: { width: 1280, height: 720, safeZone: 0.1 },
  linkedin: { width: 1280, height: 720, safeZone: 0.1 },
  tiktok: { width: 1080, height: 1920, safeZone: 0.15 },
  instagram: { width: 1080, height: 1920, safeZone: 0.12 },
};

export const SocialClip: React.FC<{
  brandId: string;
  platform: keyof typeof PLATFORM_SPECS;
  feature: string;
}> = ({brandId, platform, feature}) => {
  const spec = PLATFORM_SPECS[platform];
  const safeWidth = spec.width * (1 - 2 * spec.safeZone);
  
  return (
    <AbsoluteFill style={{width: spec.width, height: spec.height}}>
      {/* content */}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### Blender Not Found

```
Error: Blender executable not found at /Applications/Blender.app
```

Set `BLENDER_PATH` in `.env` or skip 3D logo reveals (pipeline falls back to 2D).

### ElevenLabs Rate Limit

```
Error: 429 Too Many Requests
```

Audio generation is optional. Pipeline renders video without voiceover if API fails.

### Playwright Capture Timeout

```
Error: Page did not load within 30s
```

Ensure your dev server is running on the correct port. Capture script waits for `networkidle` state.

### Remotion Memory Error

```
Error: JavaScript heap out of memory
```

Set Node memory limit:

```bash
export NODE_OPTIONS="--max-old-space-size=8192"
npx remotion render ...
```

### Brand Schema Validation Failed

```
Error: Invalid brand JSON
```

Run `cd studio && npm test` to see schema violations. All 13 colors, 3 fonts, and voice rules are required.

## File Structure

```
marketing-studio/
├── brands/              # per-product brand tokens (JSON)
├── studio/              # Remotion project (all compositions)
│   ├── src/
│   │   ├── compositions/
│   │   │   ├── LogoReveal.tsx
│   │   │   ├── ProductDemo.tsx
│   │   │   ├── LaunchVideo.tsx
│   │   │   ├── SocialClip.tsx
│   │   │   └── AnimatedOG.tsx
│   │   ├── brands/      # logo mark components
│   │   └── lib/
│   │       ├── brand.ts
│   │       └── camera.ts
├── feeders/
│   ├── blender/         # headless Blender scenes
│   ├── capture/         # Playwright recorder
│   ├── audio/           # ElevenLabs client
│   └── comfy/           # ComfyUI client (optional)
├── skills/              # Claude Code skills
│   ├── marketing.md
│   ├── logo-reveal.md
│   ├── product-demo.md
│   └── ...
├── scripts/
│   ├── install-skills.mjs
│   ├── build-launch-props.mjs
│   ├── build-social-props.mjs
│   └── smoke.mjs
├── props/               # generated render props (JSON)
├── assets/              # captures, audio, blender frames
├── out/                 # final rendered videos
└── examples/            # real output from shipped products
```

## Best Practices

1. **Run smoke test after brand changes**: `node scripts/smoke.mjs` catches schema errors before full renders
2. **Use props builders**: Never hand-edit props JSON; use `scripts/build-*-props.mjs`
3. **Film demo last in UI work**: Run `/polish` and `/frontend-verify` before `/product-demo`
4. **Score audio after picture lock**: Don't generate voiceover until launch video script is final
5. **Test locally before `/marketing`**: Run individual skills (`/logo-reveal`, `/product-demo`) to debug brand tokens
6. **Keep run manifest**: Don't delete `.marketing-run.json` during active runs (enables resume)

## Related Skills

- `/polish` – Final UI quality pass before demo filming
- `/frontend-verify` – Headless route verification (console errors, failed requests)
- `/de-vibe` – Remove AI-generated fingerprints before shipping
- `/launch` – Generate announcement drafts per channel

All supporting skills install with `scripts/install-skills.mjs`.
