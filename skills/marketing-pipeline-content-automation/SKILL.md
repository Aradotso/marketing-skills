---
name: marketing-pipeline-content-automation
description: Automated AI content pipeline for research, script generation, and video creation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with this pipeline
  - set up the marketing content automation system
  - generate content from research to video automatically
  - use Claude and OpenAI for content generation
  - create automated social media content workflow
  - render videos from text content using Remotion
  - configure the AI content pipeline
  - build automated content research and publishing
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a complete content automation system that handles research, scriptwriting, and video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

The marketing-pipeline-share project provides:

- **Auto-scan Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh content within 24h
- **AI Content Generation**: Creates multiple content formats (toplist, POV, case studies, how-tos) in multiple languages
- **Video Rendering**: Automatically generates infographics and short-form videos from written content
- **Multi-platform Export**: Outputs optimized content for Reels, TikTok, Shorts

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
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=your_database_connection_string

# Optional: Content Publishing
FACEBOOK_PAGE_TOKEN=your_fb_page_token
LINKEDIN_API_KEY=your_linkedin_key
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run render
```

## Core Architecture

The system follows this flow:

1. **Research Phase**: Crawl and aggregate content
2. **Content Generation**: AI processes and creates articles
3. **Video Rendering**: Remotion converts text to video
4. **Publishing**: Auto-post to social platforms

## Content Research Module

### Crawling News Sources

```typescript
// src/services/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  selector: string;
  category: string;
}

export async function crawlSources(sources: ResearchSource[]) {
  const results = await Promise.all(
    sources.map(async (source) => {
      try {
        const response = await axios.get(source.url, {
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
            'X-RapidAPI-Host': 'news-api.rapidapi.com'
          }
        });
        
        return {
          category: source.category,
          data: response.data,
          timestamp: new Date()
        };
      } catch (error) {
        console.error(`Failed to crawl ${source.url}:`, error);
        return null;
      }
    })
  );
  
  return results.filter(Boolean);
}

// Usage
const sources = [
  { url: 'https://api.example.com/techcrunch', selector: 'article', category: 'tech' },
  { url: 'https://api.example.com/a16z', selector: 'post', category: 'startup' }
];

const crawledData = await crawlSources(sources);
```

### Extracting Insights

```typescript
// src/services/research/analyzer.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function extractInsights(crawledContent: string[]) {
  const prompt = `Analyze the following news articles and extract:
1. Key trends and patterns
2. Important statistics and data points
3. Actionable insights for content creators

Articles:
${crawledContent.join('\n\n')}`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

## AI Content Generation

### Multi-Format Content Creation

```typescript
// src/services/content/generator.ts
import OpenAI from 'openai';
import Anthropic from '@anthropic-ai/sdk';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  insights: string;
}

export async function generateContent(request: ContentRequest) {
  const formatPrompts = {
    'toplist': `Create a top 10 list article about ${request.topic}`,
    'pov': `Write a point-of-view article analyzing ${request.topic}`,
    'case-study': `Create a detailed case study about ${request.topic}`,
    'how-to': `Write a step-by-step how-to guide for ${request.topic}`
  };

  const toneInstructions = {
    'professional': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Use light humor and engaging storytelling'
  };

  const systemPrompt = `You are an expert content creator. ${toneInstructions[request.tone]}. 
Write in ${request.language === 'vi' ? 'Vietnamese' : 'English'}.
Use these insights: ${request.insights}`;

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: formatPrompts[request.format] }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### Dual-Language Generation

```typescript
// src/services/content/multilingual.ts
export async function generateBilingualContent(topic: string, insights: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      topic,
      format: 'toplist',
      language: 'en',
      tone: 'professional',
      insights
    }),
    generateContent({
      topic,
      format: 'toplist',
      language: 'vi',
      tone: 'professional',
      insights
    })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

## Video Rendering with Remotion

### Video Composition Setup

```typescript
// src/video/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  points, 
  brandColor 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  const currentPoint = Math.floor(frame / (fps * 2)) % points.length;

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ 
        opacity,
        padding: 60,
        color: '#fff',
        fontFamily: 'Arial, sans-serif'
      }}>
        <h1 style={{ 
          fontSize: 72, 
          marginBottom: 40,
          color: brandColor 
        }}>
          {title}
        </h1>
        
        <div style={{ fontSize: 48, lineHeight: 1.6 }}>
          {points[currentPoint]}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### Register Compositions

```typescript
// src/video/Root.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Top 10 AI Trends',
          points: [
            '1. Generative AI is transforming content creation',
            '2. AI agents are automating workflows',
            '3. Multi-modal AI is becoming mainstream'
          ],
          brandColor: '#FF6B6B'
        }}
      />
    </>
  );
};
```

### Render Video Programmatically

```typescript
// src/services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  contentData: { title: string; points: string[] }
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/video/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: contentData.title,
      points: contentData.points,
      brandColor: '#FF6B6B'
    },
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public', 
    'videos', 
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: contentData.title,
      points: contentData.points,
      brandColor: '#FF6B6B'
    },
  });

  return outputLocation;
}
```

## Complete Pipeline Workflow

```typescript
// src/services/pipeline/executor.ts
import { crawlSources } from '../research/crawler';
import { extractInsights } from '../research/analyzer';
import { generateContent } from '../content/generator';
import { renderContentVideo } from '../video/renderer';

export async function executeContentPipeline(keyword: string) {
  console.log(`Starting pipeline for keyword: ${keyword}`);
  
  // Step 1: Research
  const sources = [
    { url: 'https://api.example.com/news', selector: 'article', category: 'tech' }
  ];
  const crawledData = await crawlSources(sources);
  
  // Step 2: Extract insights
  const articles = crawledData.map(d => JSON.stringify(d?.data));
  const insights = await extractInsights(articles);
  
  // Step 3: Generate content
  const content = await generateContent({
    topic: keyword,
    format: 'toplist',
    language: 'en',
    tone: 'professional',
    insights
  });
  
  // Step 4: Parse content for video
  const points = content?.split('\n').filter(line => line.match(/^\d+\./));
  
  // Step 5: Render video
  const videoPath = await renderContentVideo({
    title: `Top Insights: ${keyword}`,
    points: points || []
  });
  
  return {
    content,
    insights,
    videoPath
  };
}
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { executeContentPipeline } from '@/services/pipeline/executor';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword } = req.body;

  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }

  try {
    const result = await executeContentPipeline(keyword);
    return res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return res.status(500).json({ 
      error: 'Pipeline execution failed',
      details: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

### Video Rendering Endpoint

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderContentVideo } from '@/services/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { title, points } = req.body;

  try {
    const videoPath = await renderContentVideo({ title, points });
    return res.status(200).json({ videoPath });
  } catch (error) {
    console.error('Render error:', error);
    return res.status(500).json({ error: 'Video rendering failed' });
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
// src/utils/batch-generator.ts
export async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await executeContentPipeline(keyword);
      results.push({ keyword, success: true, data: result });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
    } catch (error) {
      results.push({ keyword, success: false, error });
    }
  }
  
  return results;
}
```

### Content Scheduling

```typescript
// src/services/scheduler/publisher.ts
interface ScheduledContent {
  content: string;
  publishAt: Date;
  platform: 'facebook' | 'linkedin' | 'twitter';
}

export async function scheduleContent(item: ScheduledContent) {
  const delay = item.publishAt.getTime() - Date.now();
  
  if (delay > 0) {
    setTimeout(async () => {
      await publishToPlatform(item.platform, item.content);
    }, delay);
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/utils/rate-limiter.ts
export class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
  private processing = false;
  
  constructor(private delayMs: number = 1000) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
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
    const task = this.queue.shift();
    
    if (task) {
      await task();
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
    }
    
    this.processing = false;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter(2000);
await limiter.execute(() => generateContent(request));
```

### Video Rendering Memory Issues

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setScale(0.5); // Reduce quality for faster rendering
Config.setConcurrency(1); // Limit parallel rendering
```

### Error Recovery

```typescript
// src/utils/retry.ts
export async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw new Error('Max retries exceeded');
}

// Usage
const content = await retryWithBackoff(() => 
  generateContent(request)
);
```

## Testing

```typescript
// tests/pipeline.test.ts
import { executeContentPipeline } from '@/services/pipeline/executor';

describe('Content Pipeline', () => {
  it('should generate content from keyword', async () => {
    const result = await executeContentPipeline('AI trends 2024');
    
    expect(result.content).toBeDefined();
    expect(result.insights).toBeDefined();
    expect(result.videoPath).toMatch(/\.mp4$/);
  });
});
```
