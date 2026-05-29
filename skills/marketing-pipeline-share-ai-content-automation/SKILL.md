---
name: marketing-pipeline-share-ai-content-automation
description: Automated content pipeline for research, scriptwriting, and video generation using Claude/OpenAI APIs
triggers:
  - automate content creation from research to video
  - generate AI-powered marketing content pipeline
  - create automated blog posts with video rendering
  - build content automation system with Claude
  - set up AI content research and generation
  - generate videos from text content automatically
  - create multi-language marketing content with AI
  - automate content scraping and writing workflow
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that transforms a single keyword into complete content packages including research, written articles, and rendered videos. It leverages Claude 3, OpenAI APIs, and Remotion for end-to-end content generation.

**Key capabilities:**
- Automated web scraping from TechCrunch, a16z, Twitter/X, LinkedIn
- Multi-format content generation (listicles, POV, case studies, how-tos)
- Bilingual output (English/Vietnamese)
- Automatic video/infographic rendering via Remotion
- Next.js-based UI for content management

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if using persistent storage)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Core Architecture

### 1. Research Module (Auto-Scan)

The research pipeline automatically crawls and aggregates content:

```typescript
// lib/research/scraper.ts
import { AnthropicClient } from '@anthropic-ai/sdk';

interface ResearchSource {
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  url: string;
  timeRange: '24h' | '7d' | '30d';
}

export async function scanResearch(
  keyword: string,
  sources: ResearchSource[]
): Promise<ResearchData[]> {
  const results = await Promise.all(
    sources.map(async (source) => {
      const crawledData = await fetchFromSource(source, keyword);
      return parseAndExtractInsights(crawledData);
    })
  );
  
  return results.flat();
}

async function fetchFromSource(
  source: ResearchSource,
  keyword: string
): Promise<RawData> {
  // Use RapidAPI or custom scrapers
  const response = await fetch(`https://api.rapidapi.com/search`, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
    },
    body: JSON.stringify({
      query: keyword,
      source: source.platform,
      timeRange: source.timeRange,
    }),
  });
  
  return response.json();
}
```

### 2. Content Generation with Claude/OpenAI

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: ResearchData[];
}

export async function generateContent(
  config: ContentConfig
): Promise<GeneratedContent> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!,
  });
  
  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });
  
  return parseContentResponse(message.content);
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = getFormatTemplate(config.format);
  const toneGuidelines = getToneGuidelines(config.tone);
  const researchContext = summarizeResearch(config.researchData);
  
  return `
You are a ${config.tone} content writer creating a ${config.format} article in ${config.language}.

Research Context:
${researchContext}

Format Requirements:
${formatInstructions}

Tone Guidelines:
${toneGuidelines}

Generate a complete article with:
1. Compelling headline
2. Structured body content
3. Data-backed insights from research
4. Call-to-action

Output as JSON with structure: { title, sections, cta, metadata }
  `.trim();
}
```

### 3. Multi-Language Support

```typescript
// lib/content/translator.ts
export async function generateBilingualContent(
  keyword: string,
  researchData: ResearchData[]
): Promise<{ en: Content; vi: Content }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData,
    }),
    generateContent({
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData,
    }),
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: GeneratedContent;
  platform: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const composition = selectCompositionByPlatform(config.platform);
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(
      process.cwd(),
      'src/remotion/index.ts'
    ),
  });
  
  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${config.content.id}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.content.title,
      sections: config.content.sections,
      duration: config.duration,
    },
  });
  
  return outputPath;
}

function selectCompositionByPlatform(platform: string) {
  const aspectRatios = {
    reels: { width: 1080, height: 1920 }, // 9:16
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  
  return {
    id: 'ContentVideo',
    ...aspectRatios[platform],
    fps: 30,
    durationInFrames: 900, // 30 seconds at 30fps
  };
}
```

### 5. Remotion Composition Example

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  sections: Array<{ heading: string; text: string }>;
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  sections,
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ color: 'white', fontSize: 64, textAlign: 'center' }}>
          {title}
        </h1>
        {sections.map((section, idx) => (
          <SectionSlide key={idx} section={section} frame={frame} />
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scanResearch } from '@/lib/research/scraper';
import { generateContent } from '@/lib/content/generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, includeVideo } = await req.json();
    
    // Step 1: Research
    const researchData = await scanResearch(keyword, [
      { platform: 'techcrunch', url: '', timeRange: '24h' },
      { platform: 'twitter', url: '', timeRange: '24h' },
    ]);
    
    // Step 2: Generate Content
    const content = await generateContent({
      format,
      language,
      tone: 'expert',
      researchData,
    });
    
    // Step 3: Render Video (optional)
    let videoUrl = null;
    if (includeVideo) {
      const videoPath = await renderContentVideo({
        content,
        platform: 'reels',
        duration: 30,
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl,
    });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Complete Pipeline Workflow

```typescript
// lib/pipeline/orchestrator.ts
export async function runFullPipeline(
  keyword: string
): Promise<PipelineResult> {
  console.log(`Starting pipeline for keyword: ${keyword}`);
  
  // 1. Research Phase
  const research = await scanResearch(keyword, [
    { platform: 'techcrunch', url: '', timeRange: '24h' },
    { platform: 'a16z', url: '', timeRange: '7d' },
    { platform: 'twitter', url: '', timeRange: '24h' },
  ]);
  
  console.log(`Collected ${research.length} research items`);
  
  // 2. Content Generation (Bilingual)
  const { en, vi } = await generateBilingualContent(keyword, research);
  
  // 3. Video Rendering
  const videos = await Promise.all([
    renderContentVideo({ content: en, platform: 'reels', duration: 30 }),
    renderContentVideo({ content: vi, platform: 'tiktok', duration: 30 }),
  ]);
  
  return {
    research,
    content: { en, vi },
    videos,
    createdAt: new Date(),
  };
}
```

### Format-Specific Templates

```typescript
// lib/content/formats.ts
export function getFormatTemplate(format: ContentFormat): string {
  const templates = {
    'toplist': `
      Create a numbered list article (5-10 items).
      Each item should have:
      - Bold headline
      - 2-3 paragraphs explanation
      - Data point or example
    `,
    'pov': `
      Write from a unique perspective with:
      - Personal angle/thesis
      - Supporting arguments
      - Counter-arguments addressed
      - Strong conclusion
    `,
    'case-study': `
      Structure as:
      - Background/Challenge
      - Solution/Approach
      - Results (with metrics)
      - Lessons Learned
    `,
    'how-to': `
      Step-by-step guide format:
      - Introduction (why it matters)
      - Prerequisites
      - Numbered steps with details
      - Tips and troubleshooting
    `,
  };
  
  return templates[format];
}
```

## CLI Commands (if applicable)

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos in batch
npm run render-videos

# Run research scraper standalone
npm run research -- --keyword "AI automation" --sources techcrunch,twitter
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function batchWithRateLimit<T>(
  items: T[],
  processor: (item: T) => Promise<any>
): Promise<any[]> {
  return Promise.all(
    items.map((item) => limit(() => processor(item)))
  );
}
```

### Video Rendering Errors

```typescript
// Ensure ffmpeg is installed
// Linux: apt-get install ffmpeg
// Mac: brew install ffmpeg
// Windows: Download from ffmpeg.org

// Increase memory for large renders
// In package.json:
{
  "scripts": {
    "render-videos": "NODE_OPTIONS='--max-old-space-size=4096' tsx scripts/render.ts"
  }
}
```

### Claude API Timeouts

```typescript
// lib/content/generator.ts
const message = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  timeout: 60000, // 60 seconds
  messages: [{ role: 'user', content: prompt }],
});
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Queue video rendering** for background processing
3. **Implement retry logic** for API failures
4. **Validate content** before rendering videos
5. **Use streaming responses** for real-time feedback
6. **Monitor API usage** to stay within quotas
