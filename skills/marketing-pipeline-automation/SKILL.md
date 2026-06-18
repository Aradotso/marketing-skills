```markdown
---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI
  - generate marketing videos from scripts
  - crawl and research news for content
  - create multilingual marketing content
  - build automated content pipeline
  - generate videos with Remotion and AI
  - scrape trending topics for content
  - automate social media content workflow
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a complete automation system that handles content research, scriptwriting, and video generation using AI models (Claude 3, OpenAI) and Remotion for video rendering.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter, LinkedIn for trending topics
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case studies, how-to) using Claude/OpenAI
3. **Multilingual Output**: Generates content in both English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders videos and infographics from written content using Remotion
5. **Multi-Platform Export**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Key Architecture

The project is structured as a Next.js application with TypeScript:

```
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # News crawling endpoints
│   │   ├── generate/      # Content generation
│   │   └── render/        # Video rendering
│   └── page.tsx           # Main UI
├── lib/                   # Core utilities
│   ├── ai/               # AI integrations (OpenAI, Claude)
│   ├── crawler/          # Web scraping logic
│   └── remotion/         # Video generation
├── components/           # React components
└── remotion/            # Remotion video templates
```

## Core API Endpoints

### 1. Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  // Crawl news from specified sources
  const results = await crawlNews({
    keyword,
    sources: sources || ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });
  
  return NextResponse.json({ data: results });
}
```

Usage from frontend:

```typescript
const research = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI marketing tools',
    sources: ['techcrunch', 'twitter']
  })
});

const { data } = await research.json();
```

### 2. Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { Anthropic } from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function POST(req: NextRequest) {
  const { research, format, language, tone, provider } = await req.json();
  
  const prompt = buildContentPrompt({ research, format, language, tone });
  
  let content;
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }],
    });
    content = message.content[0].text;
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{ role: 'user', content: prompt }],
    });
    content = completion.choices[0].message.content;
  }
  
  return NextResponse.json({ content });
}
```

### 3. Video Rendering Endpoint

```typescript
// app/api/render/route.ts
import { renderMedia } from '@remotion/renderer';
import { bundle } from '@remotion/bundler';

export async function POST(req: NextRequest) {
  const { content, template, platform } = await req.json();
  
  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
  });
  
  // Render video
  const { outputPath } = await renderMedia({
    composition: {
      id: template || 'ContentVideo',
      durationInFrames: 300,
      fps: 30,
      width: platform === 'shorts' ? 1080 : 1920,
      height: platform === 'shorts' ? 1920 : 1080,
    },
    serveUrl: bundled,
    codec: 'h264',
    inputProps: { content },
  });
  
  return NextResponse.json({ videoUrl: outputPath });
}
```

## Common Usage Patterns

### Full Pipeline Automation

```typescript
// lib/pipeline/automate.ts
interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone: 'professional' | 'friendly' | 'humorous';
  generateVideo: boolean;
  platforms?: ('reels' | 'tiktok' | 'shorts')[];
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  const researchData = await fetch('/api/research', {
    method: 'POST',
    body: JSON.stringify({ keyword: config.keyword }),
  }).then(r => r.json());
  
  // Step 2: Generate content for each language
  const contents = await Promise.all(
    config.languages.map(async (lang) => {
      const response = await fetch('/api/generate', {
        method: 'POST',
        body: JSON.stringify({
          research: researchData.data,
          format: config.format,
          language: lang,
          tone: config.tone,
          provider: 'claude',
        }),
      });
      const { content } = await response.json();
      return { language: lang, content };
    })
  );
  
  // Step 3: Generate videos if requested
  if (config.generateVideo) {
    const videos = await Promise.all(
      contents.flatMap(({ language, content }) =>
        (config.platforms || ['reels']).map(async (platform) => {
          const response = await fetch('/api/render', {
            method: 'POST',
            body: JSON.stringify({ content, platform }),
          });
          const { videoUrl } = await response.json();
          return { language, platform, videoUrl };
        })
      )
    );
    return { contents, videos };
  }
  
  return { contents };
}
```

### Custom Content Prompt Builder

```typescript
// lib/ai/prompt-builder.ts
interface PromptConfig {
  research: any[];
  format: string;
  language: string;
  tone: string;
}

export function buildContentPrompt(config: PromptConfig): string {
  const { research, format, language, tone } = config;
  
  const formatInstructions = {
    'toplist': 'Create a numbered list format with clear rankings',
    'pov': 'Write from a unique perspective or opinion',
    'case-study': 'Analyze with data, examples, and takeaways',
    'how-to': 'Provide step-by-step actionable guidance',
  };
  
  const toneInstructions = {
    'professional': 'Use formal, authoritative language',
    'friendly': 'Be conversational and approachable',
    'humorous': 'Include wit and engaging storytelling',
  };
  
  return `
You are a ${tone} content creator writing in ${language}.

Research Data:
${JSON.stringify(research, null, 2)}

Task: ${formatInstructions[format]}
Tone: ${toneInstructions[tone]}

Requirements:
- Use data and insights from the research
- Include specific examples and numbers
- Write ${language === 'vi' ? 'in Vietnamese' : 'in English'}
- Make it engaging and shareable
- Add relevant hashtags at the end

Generate the complete article now:
  `.trim();
}
```

### Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: {
    title: string;
    points: string[];
    hashtags: string[];
  };
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      <div style={{ opacity }}>
        <h1 style={{ color: 'white', fontSize: 72, marginBottom: 40 }}>
          {content.title}
        </h1>
        
        {content.points.map((point, i) => {
          const pointFrame = fps * (i + 1);
          const pointOpacity = frame > pointFrame ? 1 : 0;
          
          return (
            <p
              key={i}
              style={{
                color: 'white',
                fontSize: 36,
                marginBottom: 20,
                opacity: pointOpacity,
                transition: 'opacity 0.5s',
              }}
            >
              {point}
            </p>
          );
        })}
        
        <div style={{ marginTop: 40, color: '#888', fontSize: 24 }}>
          {content.hashtags.join(' ')}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### Web Crawler Implementation

```typescript
// lib/crawler/news-crawler.ts
interface CrawlConfig {
  keyword: string;
  sources: string[];
  timeframe: string;
}

export async function crawlNews(config: CrawlConfig) {
  const { keyword, sources, timeframe } = config;
  
  const results = await Promise.all(
    sources.map(async (source) => {
      switch (source) {
        case 'techcrunch':
          return crawlTechCrunch(keyword, timeframe);
        case 'twitter':
          return crawlTwitter(keyword, timeframe);
        case 'a16z':
          return crawlA16z(keyword, timeframe);
        default:
          return [];
      }
    })
  );
  
  return results.flat();
}

async function crawlTechCrunch(keyword: string, timeframe: string) {
  // Use RapidAPI or direct scraping
  const response = await fetch('https://api.rapidapi.com/techcrunch/search', {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
    },
    method: 'POST',
    body: JSON.stringify({ query: keyword, timeframe }),
  });
  
  const data = await response.json();
  return data.articles.map((article: any) => ({
    title: article.title,
    url: article.url,
    summary: article.excerpt,
    publishedAt: article.date,
    source: 'TechCrunch',
  }));
}
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Access at http://localhost:3000
```

## Building for Production

```bash
# Build the application
npm run build

# Start production server
npm start
```

## Remotion CLI Commands

```bash
# Preview Remotion compositions
npx remotion preview

# Render a specific video
npx remotion render ContentVideo output.mp4 --props='{"content": {...}}'

# Bundle for serverless rendering
npx remotion bundle
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors with AI providers:

```typescript
// lib/ai/rate-limiter.ts
import pRetry from 'p-retry';

export async function callWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  return pRetry(fn, {
    retries: maxRetries,
    onFailedAttempt: (error) => {
      console.log(
        `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
      );
    },
  });
}

// Usage
const content = await callWithRetry(() =>
  anthropic.messages.create({...})
);
```

### Memory Issues with Video Rendering

For large video renders, increase Node.js memory:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### CORS Issues with API Routes

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const response = NextResponse.next();
  
  response.headers.set('Access-Control-Allow-Origin', '*');
  response.headers.set('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');
  response.headers.set('Access-Control-Allow-Headers', 'Content-Type');
  
  return response;
}
```

### Missing Environment Variables

Always validate env vars at startup:

```typescript
// lib/config/validate-env.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY',
];

export function validateEnv() {
  const missing = requiredEnvVars.filter((key) => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

## Best Practices

1. **Caching Research Data**: Store crawled data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for heavy video processing
3. **Content Versioning**: Save generated content with metadata for iteration
4. **Error Handling**: Always wrap AI calls with try-catch and proper logging
5. **Cost Monitoring**: Track API usage to manage costs with Claude/OpenAI

## Integration Examples

### Scheduling Content Posts

```typescript
// lib/scheduler/auto-post.ts
import { schedule } from 'node-cron';

export function scheduleContentGeneration() {
  // Run daily at 6 AM
  schedule('0 6 * * *', async () => {
    const result = await runContentPipeline({
      keyword: 'trending AI tools',
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'professional',
      generateVideo: true,
      platforms: ['reels', 'tiktok'],
    });
    
    // Auto-post to social media
    await postToSocialMedia(result);
  });
}
```

This skill provides comprehensive coverage of the Marketing Pipeline automation system, enabling AI agents to help developers implement automated content workflows with research, generation, and video rendering capabilities.
```
