---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I set up the marketing content pipeline
  - help me create automated content with AI research
  - generate video content from articles automatically
  - configure the content automation system
  - use Claude and OpenAI for content generation
  - automate social media content creation workflow
  - set up Remotion video rendering for marketing
  - create multi-language content with AI pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end content automation system that transforms keywords into fully-formatted articles and videos. It combines AI research (scraping news from TechCrunch, Twitter, LinkedIn), content generation (Claude 3, OpenAI), and video rendering (Remotion) into a single pipeline.

**Key capabilities:**
- Auto-research trending topics from real-time sources
- Generate multi-format content (toplist, POV, case studies, how-to)
- Bilingual support (English/Vietnamese)
- Automatic video/infographic rendering
- Direct scheduling to social platforms

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Content research endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   └── (routes)/          # Page components
├── lib/                   # Core utilities
│   ├── ai/               # AI integrations (Claude, OpenAI)
│   ├── crawler/          # Web scraping modules
│   └── video/            # Remotion video logic
├── remotion/             # Video templates
└── types/                # TypeScript definitions
```

## Core API Endpoints

### 1. Research Content

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/crawler/research';

export async function POST(req: NextRequest) {
  const { keyword, sources = ['techcrunch', 'twitter', 'a16z'] } = await req.json();
  
  try {
    const insights = await researchTopic({
      keyword,
      sources,
      timeframe: '24h',
      maxResults: 20
    });
    
    return NextResponse.json({ 
      success: true, 
      data: insights 
    });
  } catch (error) {
    return NextResponse.json({ 
      success: false, 
      error: error.message 
    }, { status: 500 });
  }
}
```

### 2. Generate Content

```typescript
// app/api/generate/route.ts
import { generateContent } from '@/lib/ai/content-generator';
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { 
    research, 
    format = 'toplist', 
    language = 'en',
    tone = 'professional' 
  } = await req.json();
  
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    research,
    format,
    language,
    tone
  });
  
  return NextResponse.json({ content });
}
```

### 3. Render Video

```typescript
// app/api/render/route.ts
import { renderVideo } from '@/lib/video/renderer';
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { content, template = 'social-post' } = await req.json();
  
  const videoUrl = await renderVideo({
    composition: template,
    inputProps: {
      title: content.title,
      body: content.body,
      stats: content.stats,
      duration: 30
    },
    codec: 'h264',
    outputFormat: 'mp4'
  });
  
  return NextResponse.json({ videoUrl });
}
```

## Content Generation Patterns

### Pattern 1: Full Pipeline Automation

```typescript
// lib/pipeline/auto-content.ts
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';

export async function autoGenerateContent(keyword: string) {
  // Step 1: Research
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  // Step 2: Generate bilingual content
  const [enContent, viContent] = await Promise.all([
    generateContent({
      provider: 'claude',
      research,
      format: 'toplist',
      language: 'en',
      tone: 'professional'
    }),
    generateContent({
      provider: 'openai',
      research,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly'
    })
  ]);
  
  // Step 3: Render videos
  const [enVideo, viVideo] = await Promise.all([
    renderVideo({
      composition: 'toplist-template',
      inputProps: { content: enContent }
    }),
    renderVideo({
      composition: 'toplist-template',
      inputProps: { content: viContent }
    })
  ]);
  
  return {
    english: { content: enContent, video: enVideo },
    vietnamese: { content: viContent, video: viVideo }
  };
}
```

### Pattern 2: Custom Format Generator

```typescript
// lib/ai/format-templates.ts
export const contentFormats = {
  toplist: {
    systemPrompt: `You are a content creator specializing in top lists.
    Create engaging numbered lists with clear explanations.`,
    structure: ['intro', 'items', 'conclusion']
  },
  
  pov: {
    systemPrompt: `You are a thought leader sharing unique perspectives.
    Write in first person with strong opinions backed by data.`,
    structure: ['hook', 'perspective', 'evidence', 'call-to-action']
  },
  
  caseStudy: {
    systemPrompt: `You are analyzing real-world business cases.
    Include specific metrics, challenges, and outcomes.`,
    structure: ['background', 'challenge', 'solution', 'results']
  }
};

// Usage
import Anthropic from '@anthropic-ai/sdk';

export async function generateWithFormat(
  format: keyof typeof contentFormats,
  research: any
) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const template = contentFormats[format];
  
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    system: template.systemPrompt,
    messages: [{
      role: 'user',
      content: `Research data: ${JSON.stringify(research)}\n\nCreate content following this structure: ${template.structure.join(' -> ')}`
    }]
  });
  
  return message.content[0].text;
}
```

### Pattern 3: Multi-Platform Video Export

```typescript
// lib/video/platform-templates.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

export const platformSpecs = {
  instagram: { width: 1080, height: 1920, fps: 30 },
  youtube: { width: 1920, height: 1080, fps: 60 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  twitter: { width: 1280, height: 720, fps: 30 }
};

export async function renderForPlatforms(
  content: any,
  platforms: (keyof typeof platformSpecs)[]
) {
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  const renders = platforms.map(async (platform) => {
    const specs = platformSpecs[platform];
    const composition = await selectComposition({
      serveUrl: bundled,
      id: 'ContentVideo',
      inputProps: { ...content, ...specs }
    });
    
    const output = `./output/${platform}-${Date.now()}.mp4`;
    
    await renderMedia({
      composition,
      serveUrl: bundled,
      codec: 'h264',
      outputLocation: output,
      inputProps: { ...content, ...specs }
    });
    
    return { platform, url: output };
  });
  
  return Promise.all(renders);
}
```

## Remotion Video Components

```tsx
// remotion/compositions/TopListVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import { z } from 'zod';

export const topListSchema = z.object({
  title: z.string(),
  items: z.array(z.object({
    number: z.number(),
    title: z.string(),
    description: z.string()
  })),
  duration: z.number()
});

export const TopListVideo: React.FC<z.infer<typeof topListSchema>> = ({
  title,
  items,
  duration
}) => {
  const frame = useCurrentFrame();
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence durationInFrames={60}>
        <div style={{ 
          opacity, 
          fontSize: 72, 
          color: 'white',
          textAlign: 'center',
          padding: 40
        }}>
          {title}
        </div>
      </Sequence>
      
      {items.map((item, index) => (
        <Sequence
          key={index}
          from={60 + (index * 90)}
          durationInFrames={90}
        >
          <div style={{ 
            color: 'white', 
            padding: 40,
            fontSize: 48
          }}>
            <div style={{ fontSize: 96, fontWeight: 'bold' }}>
              {item.number}
            </div>
            <div style={{ marginTop: 20 }}>{item.title}</div>
            <div style={{ 
              fontSize: 32, 
              opacity: 0.8,
              marginTop: 20
            }}>
              {item.description}
            </div>
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## CLI Usage (if applicable)

```bash
# Research a topic
npm run research -- --keyword "AI trends" --sources techcrunch,twitter

# Generate content
npm run generate -- --input research.json --format toplist --lang en

# Render video
npm run render -- --content content.json --template social-post

# Full pipeline
npm run pipeline -- --keyword "blockchain" --formats toplist,pov --platforms instagram,youtube
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Promise<any> = Promise.resolve();
  
  async throttle(fn: () => Promise<any>, delay: number = 1000) {
    this.queue = this.queue.then(async () => {
      const result = await fn();
      await new Promise(resolve => setTimeout(resolve, delay));
      return result;
    });
    return this.queue;
  }
}

// Usage with Claude
const limiter = new RateLimiter();

export async function generateWithRateLimit(content: any) {
  return limiter.throttle(async () => {
    return await generateContent(content);
  }, 2000); // 2 second delay between requests
}
```

### Video Rendering Timeouts

```typescript
// lib/video/renderer.ts
export async function renderWithRetry(
  props: RenderProps,
  maxRetries: number = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await renderVideo({
        ...props,
        timeoutInMilliseconds: 120000 // 2 minutes
      });
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}`);
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
}
```

### Memory Issues with Large Content

```typescript
// lib/utils/chunk-processor.ts
export async function processInChunks<T, R>(
  items: T[],
  processor: (chunk: T[]) => Promise<R[]>,
  chunkSize: number = 5
): Promise<R[]> {
  const results: R[] = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await processor(chunk);
    results.push(...chunkResults);
  }
  
  return results;
}
```

## Best Practices

1. **Always validate research data** before passing to AI generators
2. **Cache research results** to avoid duplicate API calls
3. **Use streaming responses** for long-form content generation
4. **Implement queue systems** for video rendering to prevent memory overflow
5. **Store rendered videos** in cloud storage (S3, Cloudflare R2) instead of local filesystem
6. **Monitor API usage** across all providers to stay within budget
