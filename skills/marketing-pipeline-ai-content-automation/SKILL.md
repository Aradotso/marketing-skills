---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content research and generation
  - generate video from blog posts automatically
  - create AI-powered marketing content pipeline
  - automate content creation with Claude and OpenAI
  - build automated content workflow with video rendering
  - set up AI content automation system
  - use marketing pipeline for content generation
  - automate research to video content creation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end AI-powered content automation system that handles research, scriptwriting, posting, and video generation. This TypeScript/Next.js pipeline crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates content in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes recent content from major sources (24h window)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Automatically converts written content into videos/infographics using Remotion
- **Platform Optimization**: Exports video in formats optimized for Reels, TikTok, Shorts

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
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for research/crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_url

# Remotion for video rendering
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key API Patterns

### 1. Research & Content Crawling

```typescript
// lib/research/crawler.ts
import { fetchNews } from './sources';

interface ResearchConfig {
  keywords: string[];
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeRange: '24h' | '7d' | '30d';
}

async function performResearch(config: ResearchConfig) {
  const results = await Promise.all(
    config.sources.map(source => 
      fetchNews({
        source,
        keywords: config.keywords,
        timeRange: config.timeRange
      })
    )
  );
  
  return results.flat();
}

// Usage
const research = await performResearch({
  keywords: ['AI', 'marketing automation'],
  sources: ['techcrunch', 'a16z'],
  timeRange: '24h'
});
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with rankings',
    'pov': 'Write from a unique perspective or viewpoint',
    'case-study': 'Analyze a specific example with data',
    'how-to': 'Provide step-by-step instructions'
  };
  
  return `
    Topic: ${request.topic}
    Format: ${formatInstructions[request.format]}
    Language: ${request.language}
    Tone: ${request.tone}
    
    Based on this research data:
    ${JSON.stringify(request.researchData, null, 2)}
    
    Generate a comprehensive article following the specified format.
  `;
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} format.`
      },
      {
        role: 'user',
        content: buildPrompt(request)
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
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function renderVideo(config: VideoConfig): Promise<string> {
  const bundled = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      title: config.title
    }
  });
  
  const outputPath = path.join(
    process.cwd(), 
    'public/videos',
    `${Date.now()}-${config.format}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: config.content,
      title: config.title,
      aspectRatio: getAspectRatio(config.format)
    }
  });
  
  return outputPath;
}

function getAspectRatio(format: string): [number, number] {
  const ratios = {
    'reels': [9, 16],
    'tiktok': [9, 16],
    'shorts': [9, 16]
  };
  return ratios[format] || [16, 9];
}
```

### 5. Complete Pipeline Flow

```typescript
// lib/pipeline/executor.ts
import { performResearch } from './research/crawler';
import { generateContent } from './ai/claude-generator';
import { renderVideo } from './video/renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  videoFormats: ('reels' | 'tiktok' | 'shorts')[];
}

async function runPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const researchData = await performResearch({
    keywords: [config.keyword],
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h'
  });
  
  // Step 2: Generate content in multiple languages
  console.log('✍️ Generating content...');
  const contents = await Promise.all(
    config.languages.map(lang => 
      generateContent({
        topic: config.keyword,
        format: config.contentFormat,
        language: lang,
        tone: 'expert',
        researchData
      })
    )
  );
  
  // Step 3: Render videos
  console.log('🎬 Rendering videos...');
  const videos = await Promise.all(
    contents.flatMap(content =>
      config.videoFormats.map(format =>
        renderVideo({
          content,
          title: config.keyword,
          format
        })
      )
    )
  );
  
  return {
    research: researchData,
    contents,
    videos
  };
}

// Usage
const result = await runPipeline({
  keyword: 'AI Marketing Automation',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  videoFormats: ['reels', 'tiktok']
});
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runPipeline } from '@/lib/pipeline/executor';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, languages } = req.body;
    
    const result = await runPipeline({
      keyword,
      contentFormat: format || 'toplist',
      languages: languages || ['en'],
      videoFormats: ['reels']
    });
    
    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline execution failed',
      message: error.message 
    });
  }
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview
npm run remotion:preview

# Render single video
npm run remotion:render
```

## Common Patterns

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { runPipeline } from '../lib/pipeline/executor';

const topics = [
  'AI Marketing Trends 2024',
  'Content Automation Tools',
  'Video Marketing Strategies'
];

async function batchGenerate() {
  for (const topic of topics) {
    console.log(`Processing: ${topic}`);
    
    await runPipeline({
      keyword: topic,
      contentFormat: 'toplist',
      languages: ['en', 'vi'],
      videoFormats: ['reels']
    });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
}

batchGenerate();
```

### Custom Research Sources

```typescript
// lib/research/custom-source.ts
interface CustomSource {
  name: string;
  url: string;
  selector: string;
}

async function crawlCustomSource(source: CustomSource) {
  const response = await fetch(
    `https://api.rapidapi.com/web-scraper`,
    {
      method: 'POST',
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        url: source.url,
        selector: source.selector
      })
    }
  );
  
  return response.json();
}
```

### Video Template Customization

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoProps {
  title: string;
  content: string;
  aspectRatio: [number, number];
}

export const ContentVideo: React.FC<VideoProps> = ({ 
  title, 
  content,
  aspectRatio 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        padding: 60, 
        color: 'white',
        opacity 
      }}>
        <h1 style={{ fontSize: 48, marginBottom: 20 }}>
          {title}
        </h1>
        <p style={{ fontSize: 24, lineHeight: 1.6 }}>
          {content.substring(0, 300)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limiting

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
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
    const fn = this.queue.shift()!;
    await fn();
    
    await new Promise(resolve => setTimeout(resolve, 1000));
    this.processing = false;
    this.process();
  }
}

export const limiter = new RateLimiter();
```

### Video Rendering Memory Issues

```typescript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "remotion:render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### Claude API Timeout Handling

```typescript
async function generateWithRetry(request: ContentRequest, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}`);
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

## Best Practices

1. **Cache Research Data**: Store crawled data to avoid redundant API calls
2. **Queue Video Rendering**: Use job queue (Bull, BullMQ) for video generation
3. **Monitor Costs**: Track AI API usage with logging middleware
4. **Optimize Prompts**: Test and refine prompts for consistent quality
5. **Error Recovery**: Implement checkpoints for long-running pipelines
