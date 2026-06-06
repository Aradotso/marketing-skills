---
name: marketing-pipeline-share-automation
description: AI-powered content pipeline that automates research, scriptwriting, and video generation for marketing content
triggers:
  - how do I automate content creation with marketing pipeline
  - set up AI content pipeline for social media
  - generate videos from content automatically
  - crawl news sources for marketing content
  - create multilingual marketing content with AI
  - automate content research and video generation
  - build automated content workflow with Claude
  - configure marketing pipeline automation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use the Marketing Pipeline Share system - an all-in-one automated content pipeline that handles research (crawling news sources), scriptwriting (using Claude/OpenAI), and video generation (using Remotion). The system can automatically scan sources like TechCrunch, a16z, Twitter, and LinkedIn for fresh content, generate articles in multiple formats and languages, and render them as videos.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes data from major news sources in the last 24 hours
2. **AI Content Generation**: Creates content in various formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
3. **Multilingual Output**: Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically converts written content into videos/infographics using Remotion
5. **Multi-Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

Create a `.env.local` file in the project root:

```env
# AI Service APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Video rendering (if separate)
npm run render
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Core utilities
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── crawlers/          # Content crawlers
│   └── video/             # Video generation
├── public/                # Static assets
└── remotion/              # Remotion video compositions
```

## Core API Usage

### Content Research & Crawling

```typescript
// lib/crawlers/news-scraper.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsArticle {
  title: string;
  url: string;
  source: string;
  publishedAt: string;
  content: string;
}

export async function scrapeRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    const response = await rapidAPI.searchNews({
      query: keyword,
      source,
      timeRange: '24h',
    });
    
    articles.push(...response.articles);
  }
  
  return articles;
}

// Usage example
const articles = await scrapeRecentNews('AI marketing', ['techcrunch', 'x']);
console.log(`Found ${articles.length} recent articles`);
```

### AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = `
You are a professional content creator. Based on the following research data, create a ${request.format} article about "${request.keyword}".

Language: ${request.language}
Tone: ${request.tone}

Research Data:
${request.researchData}

Requirements:
- Make it engaging and data-backed
- Include specific insights and examples
- Optimize for social media sharing
- Length: 800-1200 words
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt,
    }],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Usage example
const content = await generateContent({
  keyword: 'AI in Marketing 2024',
  format: 'toplist',
  language: 'en',
  tone: 'professional',
  researchData: articlesData,
});
```

### Video Generation with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  backgroundColor: string;
}> = ({ title, points, backgroundColor }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <div style={{
        padding: 60,
        opacity,
        transform: `translateY(${Math.max(0, 50 - frame)}px)`,
      }}>
        <h1 style={{ fontSize: 72, fontWeight: 'bold', marginBottom: 40 }}>
          {title}
        </h1>
        <ul style={{ fontSize: 36, lineHeight: 1.6 }}>
          {points.map((point, idx) => (
            <li key={idx} style={{
              opacity: frame > fps * (idx + 1) ? 1 : 0,
              transition: 'opacity 0.5s',
            }}>
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderOptions {
  compositionId: string;
  inputProps: Record<string, any>;
  outputPath: string;
  format?: 'mp4' | 'gif';
}

export async function renderVideo(options: RenderOptions): Promise<string> {
  const bundled = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: options.compositionId,
    inputProps: options.inputProps,
  });
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: options.outputPath,
    inputProps: options.inputProps,
  });
  
  return options.outputPath;
}

// Usage example
const videoPath = await renderVideo({
  compositionId: 'ContentVideo',
  inputProps: {
    title: 'Top 5 AI Marketing Trends',
    points: ['Trend 1', 'Trend 2', 'Trend 3'],
    backgroundColor: '#1a1a1a',
  },
  outputPath: 'output/video.mp4',
});
```

## Complete Workflow Example

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeRecentNews } from '@/lib/crawlers/news-scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone } = await request.json();
    
    // Step 1: Research - Crawl recent news
    console.log('Step 1: Crawling news sources...');
    const articles = await scrapeRecentNews(keyword, [
      'techcrunch',
      'a16z',
      'x',
      'linkedin',
    ]);
    
    const researchData = articles
      .map(a => `${a.title}\n${a.content}\nSource: ${a.source}`)
      .join('\n\n---\n\n');
    
    // Step 2: Generate content with AI
    console.log('Step 2: Generating content with AI...');
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      researchData,
    });
    
    // Step 3: Extract key points for video
    const points = extractKeyPoints(content);
    
    // Step 4: Render video
    console.log('Step 3: Rendering video...');
    const videoPath = await renderVideo({
      compositionId: 'ContentVideo',
      inputProps: {
        title: keyword,
        points: points.slice(0, 5),
        backgroundColor: '#0f172a',
      },
      outputPath: `public/videos/${Date.now()}.mp4`,
    });
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl: videoPath.replace('public', ''),
      articlesFound: articles.length,
    });
    
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - can be enhanced with AI
  const lines = content.split('\n');
  return lines
    .filter(line => line.match(/^[0-9\-\*]/))
    .map(line => line.replace(/^[0-9\-\*.\s]+/, '').trim())
    .filter(line => line.length > 10);
}
```

## API Routes

### Generate Content

```typescript
// POST /api/generate-content
{
  "keyword": "AI Marketing Automation",
  "format": "toplist",
  "language": "en",
  "tone": "professional"
}

// Response
{
  "success": true,
  "content": "...",
  "videoUrl": "/videos/1234567890.mp4",
  "articlesFound": 15
}
```

### Crawl Sources

```typescript
// POST /api/crawl
{
  "keyword": "AI trends",
  "sources": ["techcrunch", "a16z"],
  "timeRange": "24h"
}

// Response
{
  "articles": [
    {
      "title": "...",
      "url": "...",
      "source": "techcrunch",
      "publishedAt": "2024-01-15T10:00:00Z"
    }
  ]
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const articles = await scrapeRecentNews(keyword);
      const content = await generateContent({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'professional',
        researchData: formatArticles(articles),
      });
      
      return { keyword, content, articles: articles.length };
    })
  );
  
  return results;
}
```

### Scheduled Content Pipeline

```typescript
// lib/scheduler/content-pipeline.ts
import cron from 'node-cron';

export function startContentPipeline() {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    const keywords = ['AI marketing', 'content automation', 'social media'];
    
    for (const keyword of keywords) {
      try {
        await generateAndPublishContent(keyword);
      } catch (error) {
        console.error(`Failed for keyword: ${keyword}`, error);
      }
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
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
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    this.processing = false;
    this.process();
  }
}

const limiter = new RateLimiter();
const result = await limiter.add(() => anthropic.messages.create({...}));
```

### Video Rendering Memory Issues

```typescript
// Render in chunks for long videos
export async function renderLongVideo(sections: VideoSection[]) {
  const chunks = [];
  
  for (let i = 0; i < sections.length; i++) {
    const chunkPath = `output/chunk-${i}.mp4`;
    await renderVideo({
      compositionId: 'ContentVideo',
      inputProps: sections[i],
      outputPath: chunkPath,
    });
    chunks.push(chunkPath);
  }
  
  // Concatenate chunks using ffmpeg
  await concatenateVideos(chunks, 'output/final.mp4');
  
  return 'output/final.mp4';
}
```

### Error Handling

```typescript
export async function safeContentGeneration(request: ContentRequest) {
  const maxRetries = 3;
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request);
    } catch (error) {
      lastError = error;
      console.error(`Attempt ${i + 1} failed:`, error);
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  
  throw new Error(`Failed after ${maxRetries} attempts: ${lastError}`);
}
```

## Performance Optimization

```typescript
// Cache research data
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function getCachedNews(keyword: string) {
  const cached = await redis.get(`news:${keyword}`);
  
  if (cached) {
    return JSON.parse(cached);
  }
  
  const articles = await scrapeRecentNews(keyword);
  await redis.setex(`news:${keyword}`, 3600, JSON.stringify(articles));
  
  return articles;
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Share automation system, enabling AI agents to help developers implement automated content workflows with research, AI generation, and video rendering capabilities.
