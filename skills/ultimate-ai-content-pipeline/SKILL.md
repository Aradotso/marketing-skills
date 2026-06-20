---
name: ultimate-ai-content-pipeline
description: Automate end-to-end content creation from research, scriptwriting, to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "help me set up the AI content automation pipeline"
  - "how do I generate content with Claude and OpenAI"
  - "automate content research and video creation"
  - "create automated marketing content pipeline"
  - "generate videos from articles using Remotion"
  - "set up AI content research and generation"
  - "build automated content workflow with AI"
  - "configure content pipeline with Claude API"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline leverages Claude 3, OpenAI, and Remotion to transform keywords into fully-formed articles and videos.

## What This Project Does

The Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-crawls news sources** (TechCrunch, a16z, Twitter/X, LinkedIn) for fresh insights within 24 hours
- **Generates multi-format content** using Claude/OpenAI (toplist, POV, case studies, how-to guides)
- **Supports bilingual output** (English & Vietnamese) with customizable tones
- **Renders videos automatically** using Remotion from article content
- **Exports platform-optimized videos** for Reels, TikTok, and Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Application settings
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development

# Remotion (optional, for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── api/             # API routes
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Modules and Usage

### 1. Content Research Module

```typescript
// src/lib/crawler/news-scanner.ts
import { scanLatestNews } from '@/lib/crawler/news-scanner';

interface NewsSource {
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeframe: number; // hours
  keywords: string[];
}

async function researchTopic(keyword: string): Promise<any[]> {
  const sources: NewsSource[] = [
    { platform: 'techcrunch', timeframe: 24, keywords: [keyword] },
    { platform: 'twitter', timeframe: 24, keywords: [keyword] }
  ];
  
  const insights = await scanLatestNews(sources);
  return insights;
}

// Usage
const aiInsights = await researchTopic('artificial intelligence');
console.log(aiInsights);
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any[];
}

async function generateContent(
  keyword: string, 
  config: ContentConfig
): Promise<string> {
  const prompt = buildPrompt(keyword, config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(keyword: string, config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article',
    'pov': 'Write from a personal perspective',
    'case-study': 'Analyze a real-world example',
    'how-to': 'Provide step-by-step instructions'
  };
  
  return `
You are a ${config.tone} content writer. 
Format: ${formatInstructions[config.format]}
Language: ${config.language === 'vi' ? 'Vietnamese' : 'English'}
Topic: ${keyword}

Research data:
${JSON.stringify(config.researchData, null, 2)}

Generate a compelling article that incorporates the research insights.
`;
}
```

### 3. OpenAI Integration Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithGPT(
  keyword: string,
  systemPrompt: string,
  context: any[]
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: systemPrompt
      },
      {
        role: 'user',
        content: `Topic: ${keyword}\n\nContext: ${JSON.stringify(context)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  platform: 'reels' | 'tiktok' | 'shorts';
}

const PLATFORM_SPECS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content
    }
  });
  
  const specs = PLATFORM_SPECS[config.platform];
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      content: config.content
    },
    ...specs
  });
  
  return outputPath;
}

// Usage
const videoPath = await renderContentVideo({
  title: 'AI Marketing Trends 2024',
  content: ['Point 1', 'Point 2', 'Point 3'],
  platform: 'reels'
}, './output/video.mp4');
```

### 5. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { scanLatestNews } from '@/lib/crawler/news-scanner';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/render-video';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(
  config: PipelineConfig
): Promise<{
  article: string;
  videoPath?: string;
  research: any[];
}> {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await scanLatestNews([
    { platform: 'techcrunch', timeframe: 24, keywords: [config.keyword] }
  ]);
  
  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const article = await generateContent(config.keyword, {
    format: config.format,
    tone: config.tone,
    language: config.language,
    researchData: research
  });
  
  let videoPath: string | undefined;
  
  // Step 3: Generate video (optional)
  if (config.generateVideo && config.platform) {
    console.log('🎬 Rendering video...');
    const contentPoints = article.split('\n\n').slice(0, 5);
    videoPath = await renderContentVideo({
      title: config.keyword,
      content: contentPoints,
      platform: config.platform
    }, `./output/${config.keyword}-${Date.now()}.mp4`);
  }
  
  return { article, videoPath, research };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI in Marketing 2024',
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  generateVideo: true,
  platform: 'reels'
});

console.log('Article:', result.article);
console.log('Video:', result.videoPath);
```

## API Routes

### Create Content Endpoint

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      tone: body.tone || 'friendly',
      language: body.language || 'en',
      generateVideo: body.generateVideo || false,
      platform: body.platform
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error: any) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

### Usage from Client

```typescript
// Example client-side usage
async function createContent() {
  const response = await fetch('/api/content/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword: 'AI Marketing Automation',
      format: 'how-to',
      tone: 'expert',
      language: 'en',
      generateVideo: true,
      platform: 'reels'
    })
  });
  
  const result = await response.json();
  return result;
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run video rendering only
npm run remotion
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'friendly',
        language: 'en',
        generateVideo: false
      })
    )
  );
  
  return results;
}
```

### Multi-language Content

```typescript
async function generateBilingualContent(keyword: string) {
  const [english, vietnamese] = await Promise.all([
    generateContent(keyword, {
      format: 'pov',
      tone: 'expert',
      language: 'en',
      researchData: []
    }),
    generateContent(keyword, {
      format: 'pov',
      tone: 'expert',
      language: 'vi',
      researchData: []
    })
  ]);
  
  return { english, vietnamese };
}
```

### Custom Video Templates

```typescript
// remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  content: string[];
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <h1 style={{ color: '#fff', opacity: frame / 30 }}>
        {title}
      </h1>
      {content.map((item, i) => (
        <p key={i} style={{ color: '#fff' }}>{item}</p>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrent renders
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent renders

async function renderMultipleVideos(configs: VideoConfig[]) {
  return Promise.all(
    configs.map(config => 
      limit(() => renderContentVideo(config, `./output/${config.title}.mp4`))
    )
  );
}
```

### Missing Environment Variables

```typescript
// Validate required environment variables on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

## Advanced Configuration

### Custom Content Formatters

```typescript
// src/lib/content/formatters.ts
export const formatters = {
  toplist: (items: string[]) => {
    return items.map((item, i) => `${i + 1}. ${item}`).join('\n\n');
  },
  
  casestudy: (data: any) => {
    return `
## Background
${data.background}

## Challenge
${data.challenge}

## Solution
${data.solution}

## Results
${data.results}
    `.trim();
  }
};
```

### Platform-Specific Video Specs

```typescript
export const VIDEO_SPECS = {
  reels: { width: 1080, height: 1920, fps: 30, duration: 60 },
  tiktok: { width: 1080, height: 1920, fps: 30, duration: 180 },
  shorts: { width: 1080, height: 1920, fps: 30, duration: 60 },
  youtube: { width: 1920, height: 1080, fps: 60, duration: 600 }
};
```

This skill provides comprehensive guidance for AI agents to effectively use the Ultimate AI Content Pipeline for automated marketing content creation.
