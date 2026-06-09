---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for Vietnamese and English marketing content
triggers:
  - how do I automate content generation with AI
  - set up automated marketing content pipeline
  - generate videos from articles automatically
  - research and write blog posts with Claude AI
  - create multilingual content with OpenAI
  - automate TikTok and Reels video production
  - scrape news and generate social media content
  - build AI content pipeline with Remotion
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive system that automates content creation from research through video generation. The pipeline crawls news sources, generates multilingual content (English/Vietnamese), and produces video content using Remotion.

## What This Project Does

The Marketing Pipeline automates:

1. **Research Phase**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for recent news
2. **Content Generation**: Uses Claude 3/OpenAI to write articles in multiple formats (toplist, POV, case study, how-to)
3. **Localization**: Generates parallel English and Vietnamese versions
4. **Video Production**: Converts articles into short-form videos using Remotion
5. **Multi-Platform Publishing**: Exports content optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
node >= 18.x
npm or yarn
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment variables
cp .env.example .env
```

### Environment Configuration

Create `.env` file with:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Remotion Config
REMOTION_TIMEOUT=120000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Core Components Architecture

### 1. Research Module (`/src/research`)

Crawls and aggregates content from multiple sources:

```typescript
// src/research/aggregator.ts
import { TechCrunchScraper } from './scrapers/techcrunch';
import { TwitterScraper } from './scrapers/twitter';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
}

export async function aggregateResearch(
  keyword: string,
  hours: number = 24
): Promise<ResearchResult[]> {
  const sources = [
    new TechCrunchScraper(),
    new TwitterScraper(process.env.TWITTER_BEARER_TOKEN),
  ];

  const results = await Promise.all(
    sources.map(source => source.search(keyword, hours))
  );

  return results.flat().sort((a, b) => 
    b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}
```

### 2. Content Generation (`/src/generators`)

Uses AI to create formatted content:

```typescript
// src/generators/article.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ArticleConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'casual' | 'humorous';
  language: 'en' | 'vi';
  researchData: ResearchResult[];
}

export async function generateArticle(
  config: ArticleConfig
): Promise<string> {
  const systemPrompt = buildSystemPrompt(config);
  const userPrompt = buildUserPrompt(config);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
    system: systemPrompt,
  });

  return message.content[0].text;
}

function buildSystemPrompt(config: ArticleConfig): string {
  const toneMap = {
    expert: 'professional and authoritative',
    casual: 'friendly and conversational',
    humorous: 'witty and entertaining',
  };

  return `You are a ${toneMap[config.tone]} content writer specializing in ${config.format} articles. Write in ${config.language === 'vi' ? 'Vietnamese' : 'English'}.`;
}

function buildUserPrompt(config: ArticleConfig): string {
  const researchContext = config.researchData
    .map(r => `- ${r.title} (${r.source}): ${r.summary}`)
    .join('\n');

  return `Write a ${config.format} article about "${config.keyword}".

Recent research:
${researchContext}

Requirements:
- Include data and statistics from research
- ${config.format === 'toplist' ? 'Create numbered list with explanations' : ''}
- ${config.format === 'case-study' ? 'Include real examples and outcomes' : ''}
- Length: 1500-2000 words
- SEO optimized with headers`;
}
```

### 3. OpenAI Alternative

```typescript
// src/generators/article-openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateArticleOpenAI(
  config: ArticleConfig
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: buildSystemPrompt(config),
      },
      {
        role: 'user',
        content: buildUserPrompt(config),
      },
    ],
    temperature: 0.7,
    max_tokens: 4096,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation (`/src/video`)

Remotion integration for video rendering:

```typescript
// src/video/composition.tsx
import { Composition } from 'remotion';
import { ArticleVideo } from './ArticleVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ArticleVideo"
        component={ArticleVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Sample Title',
          points: [],
          bgColor: '#1a1a1a',
        }}
      />
    </>
  );
};
```

```typescript
// src/video/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ArticleVideoProps {
  title: string;
  points: string[];
  bgColor: string;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  points,
  bgColor,
}) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        padding: 60,
        color: 'white',
        opacity: titleOpacity,
      }}>
        <h1 style={{ fontSize: 64, marginBottom: 40 }}>
          {title}
        </h1>
        {points.map((point, idx) => {
          const pointFrame = 60 + idx * 60;
          const pointOpacity = interpolate(
            frame,
            [pointFrame, pointFrame + 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <p
              key={idx}
              style={{
                fontSize: 32,
                marginBottom: 20,
                opacity: pointOpacity,
              }}
            >
              {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Render Video API

```typescript
// src/api/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderArticleVideo(
  articleContent: string,
  outputPath: string
): Promise<string> {
  // Parse article into video props
  const props = parseArticleForVideo(articleContent);

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/video/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: props,
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: props,
  });

  return outputPath;
}

function parseArticleForVideo(content: string) {
  const lines = content.split('\n').filter(l => l.trim());
  const title = lines[0].replace(/^#\s*/, '');
  const points = lines
    .filter(l => l.match(/^-\s|^\d+\.\s/))
    .map(l => l.replace(/^-\s|^\d+\.\s/, ''))
    .slice(0, 5);

  return { title, points, bgColor: '#1a1a1a' };
}
```

## Complete Pipeline Example

```typescript
// src/pipeline/orchestrator.ts
import { aggregateResearch } from '../research/aggregator';
import { generateArticle } from '../generators/article';
import { renderArticleVideo } from '../api/render';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
}

export async function runContentPipeline(
  config: PipelineConfig
) {
  console.log(`🔍 Researching: ${config.keyword}`);
  
  // Step 1: Research
  const research = await aggregateResearch(config.keyword, 24);
  console.log(`✅ Found ${research.length} sources`);

  const results = [];

  // Step 2: Generate content for each language
  for (const lang of config.languages) {
    console.log(`✍️  Generating ${lang} article...`);
    
    const article = await generateArticle({
      keyword: config.keyword,
      format: config.format,
      tone: 'expert',
      language: lang,
      researchData: research,
    });

    results.push({
      language: lang,
      article,
      videoPath: null,
    });

    // Step 3: Generate video if requested
    if (config.generateVideo) {
      console.log(`🎬 Rendering ${lang} video...`);
      
      const videoPath = `./output/${config.keyword}-${lang}-${Date.now()}.mp4`;
      await renderArticleVideo(article, videoPath);
      
      results[results.length - 1].videoPath = videoPath;
      console.log(`✅ Video saved: ${videoPath}`);
    }
  }

  console.log('🎉 Pipeline complete!');
  return results;
}
```

### Usage in Next.js API Route

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '../../src/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, languages, generateVideo } = req.body;

  try {
    const results = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      languages: languages || ['en', 'vi'],
      generateVideo: generateVideo || false,
    });

    res.status(200).json({ success: true, results });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline failed',
      message: error.message,
    });
  }
}
```

## CLI Commands

### Start Development Server

```bash
npm run dev
```

### Build for Production

```bash
npm run build
npm start
```

### Render Video Directly

```bash
npx remotion render src/video/index.ts ArticleVideo output.mp4 --props='{"title":"Test","points":["Point 1"]}'
```

### Run Research Only

```typescript
// scripts/research.ts
import { aggregateResearch } from '../src/research/aggregator';

const keyword = process.argv[2] || 'AI technology';

aggregateResearch(keyword, 24).then(results => {
  console.log(JSON.stringify(results, null, 2));
});
```

```bash
npm run research "AI marketing tools"
```

## Common Patterns

### Custom Article Format

```typescript
// Add new format to generators
export const formats = {
  'toplist': generateToplist,
  'pov': generatePOV,
  'case-study': generateCaseStudy,
  'how-to': generateHowTo,
  'comparison': generateComparison, // New format
};

async function generateComparison(config: ArticleConfig) {
  // Custom comparison logic
  const prompt = `Compare ${config.keyword} options based on research...`;
  // ... implementation
}
```

### Batch Processing

```typescript
// src/pipeline/batch.ts
export async function batchProcess(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['en', 'vi'],
        generateVideo: true,
      });
      results.push({ keyword, success: true, data: result });
    } catch (error) {
      results.push({ keyword, success: false, error: error.message });
    }
  }
  
  return results;
}
```

### Custom Video Templates

```typescript
// src/video/templates/MinimalVideo.tsx
export const MinimalVideo: React.FC<ArticleVideoProps> = ({ title, points }) => {
  return (
    <AbsoluteFill style={{ backgroundColor: '#fff' }}>
      <div style={{ padding: 80, color: '#000' }}>
        <h1 style={{ fontSize: 72, fontWeight: 'bold' }}>{title}</h1>
        {/* Custom styling */}
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// src/utils/rateLimit.ts
export class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.process();
    });
  }
  
  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    }
    
    this.processing = false;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter();
const article = await limiter.add(() => generateArticle(config));
```

### Video Rendering Timeout

```typescript
// Increase timeout in .env
REMOTION_TIMEOUT=300000

// Or in code
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  inputProps: props,
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Memory Issues with Large Batches

```typescript
// Process in chunks
async function processInChunks<T>(
  items: T[],
  processor: (item: T) => Promise<any>,
  chunkSize: number = 3
) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    await Promise.all(chunk.map(processor));
    
    // Force garbage collection if available
    if (global.gc) global.gc();
  }
}
```

### Missing Environment Variables

```typescript
// src/utils/validateEnv.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY',
];

export function validateEnvironment() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call in main entry point
validateEnvironment();
```

This skill covers the complete content automation pipeline from research through video generation, with practical TypeScript examples for AI agents to help developers implement and customize the system.
