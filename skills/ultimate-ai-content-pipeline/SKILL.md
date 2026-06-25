---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, scriptwriting, and video generation with Claude/OpenAI
triggers:
  - how do I generate content with the marketing pipeline
  - create automated video content from research
  - set up the AI content automation system
  - generate posts with Claude and Remotion
  - automate content research and video creation
  - use the marketing pipeline to create videos
  - how to configure the content generation pipeline
  - create AI-powered content with auto-research
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a complete TypeScript-based automation system that researches trends, generates content scripts in multiple formats, and renders videos automatically using Claude/OpenAI and Remotion.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, X (Twitter), LinkedIn within the last 24 hours
2. **AI Content Generation**: Creates posts in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
3. **Multi-Language Support**: Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically renders infographics and short-form videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

```env
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for research crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_remotion_license_here
```

## Key Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research crawlers
│   │   ├── video/       # Remotion video rendering
│   │   └── content/     # Content generation logic
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/             # Static assets
```

## Core Usage Patterns

### 1. Content Research API

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Research a topic from multiple sources
const researchData = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  maxResults: 10
});

console.log(researchData);
// {
//   articles: [...],
//   insights: [...],
//   trends: [...],
//   statistics: [...]
// }
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(research: any, format: string) {
  const prompt = `Based on this research data: ${JSON.stringify(research)}
  
Create a ${format} format post in both English and Vietnamese.
Tone: Professional yet engaging
Include data-backed insights and actionable takeaways.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].text;
}

// Usage
const content = await generateContent(researchData, 'toplist');
```

### 3. Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(research: any, style: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in data-driven marketing posts.'
      },
      {
        role: 'user',
        content: `Create a ${style} post based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(contentData: any) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: contentData.title,
      content: contentData.body,
      insights: contentData.insights,
    },
  });

  // Render video
  const outputLocation = path.join(process.cwd(), 'public/videos', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: contentData.title,
      content: contentData.body,
      insights: contentData.insights,
    },
  });

  return outputLocation;
}
```

### 5. Complete Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/remotion';

async function runContentPipeline(keyword: string, format: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeRange: '24h',
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent(research, format);

    // Step 3: Parse content structure
    const structuredContent = JSON.parse(content);

    // Step 4: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo(structuredContent);

    return {
      success: true,
      content: structuredContent,
      videoPath,
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline('AI automation trends', 'toplist');
```

## API Routes

### Research API Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();

  try {
    const data = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeRange: timeRange || '24h',
    });

    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation API Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function POST(request: NextRequest) {
  const { research, format, language, tone } = await request.json();

  const prompt = `Create a ${format} format post in ${language}.
Tone: ${tone}
Research data: ${JSON.stringify(research)}

Return as JSON with structure:
{
  "title": "...",
  "body": "...",
  "insights": ["..."],
  "cta": "..."
}`;

  try {
    const message = await anthropic.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }],
    });

    const content = message.content[0].text;
    return NextResponse.json({ success: true, content: JSON.parse(content) });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering API Endpoint

```typescript
// src/app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/remotion';

export async function POST(request: NextRequest) {
  const { content } = await request.json();

  try {
    const videoPath = await renderContentVideo(content);
    
    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(videoPath)}`,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## CLI Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video directly (if CLI script exists)
npm run render -- --keyword "AI trends" --format toplist

# Type checking
npm run type-check

# Linting
npm run lint
```

## Content Format Types

```typescript
type ContentFormat = 
  | 'toplist'      // Top 5/10 lists with rankings
  | 'pov'          // Point of view / opinion piece
  | 'case-study'   // In-depth analysis with examples
  | 'how-to'       // Step-by-step guides
  | 'news-digest'  // News roundup format
  | 'comparison';  // Side-by-side comparisons

type ContentTone =
  | 'professional'  // Expert, formal
  | 'friendly'      // Approachable, casual
  | 'humorous'      // Light, entertaining
  | 'inspirational' // Motivational, uplifting
  | 'analytical';   // Data-driven, technical
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateBilingualContent(research: any, format: string) {
  const languages = ['English', 'Vietnamese'];
  const results = {};

  for (const lang of languages) {
    const content = await generateContent(research, format, {
      language: lang,
      tone: 'professional',
    });
    results[lang.toLowerCase()] = content;
  }

  return results;
}
```

### Batch Processing

```typescript
async function processBatch(keywords: string[], format: string) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await researchTopic({ keyword });
      const content = await generateContent(research, format);
      return { keyword, content };
    })
  );

  return results;
}
```

### Error Handling and Retry Logic

```typescript
async function generateWithRetry(
  research: any,
  format: string,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(research, format);
    } catch (error) {
      console.error(`Attempt ${i + 1} failed:`, error);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pQueue from 'p-queue';

const queue = new pQueue({ concurrency: 2, interval: 1000 });

async function queuedGeneration(items: any[]) {
  return Promise.all(
    items.map(item => queue.add(() => generateContent(item.research, item.format)))
  );
}
```

### Video Rendering Memory Issues

```typescript
// Use imageFormat to reduce memory usage
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  imageFormat: 'jpeg', // Use JPEG instead of PNG
  scale: 0.8, // Reduce resolution slightly
  outputLocation,
});
```

### Research Crawler Timeouts

```typescript
// Add timeout handling
async function researchWithTimeout(keyword: string, timeout = 30000) {
  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Research timeout')), timeout)
  );

  return Promise.race([
    researchTopic({ keyword }),
    timeoutPromise,
  ]);
}
```

### Missing Environment Variables

```typescript
// Validate environment on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

## Advanced Configuration

### Custom Research Sources

```typescript
// src/lib/research/sources.ts
export const customSources = {
  techcrunch: {
    url: 'https://techcrunch.com/feed/',
    parser: 'rss',
  },
  twitter: {
    url: 'https://api.twitter.com/2/tweets/search/recent',
    parser: 'json',
    headers: { Authorization: `Bearer ${process.env.TWITTER_BEARER_TOKEN}` },
  },
  custom: {
    url: 'https://your-custom-source.com/api',
    parser: 'json',
  },
};
```

### Video Template Customization

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
  insights: string[];
}> = ({ title, content, insights }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ padding: 60, color: 'white' }}>
        <h1 style={{ fontSize: 48, marginBottom: 40 }}>{title}</h1>
        <div style={{ fontSize: 24, lineHeight: 1.6 }}>{content}</div>
        <ul style={{ marginTop: 40 }}>
          {insights.map((insight, i) => (
            <li key={i} style={{ marginBottom: 20 }}>
              {insight}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

This skill provides AI coding agents with comprehensive knowledge to help developers implement automated content pipelines using research crawling, AI generation, and video rendering capabilities.
