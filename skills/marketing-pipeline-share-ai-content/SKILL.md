---
name: marketing-pipeline-share-ai-content
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automatic article research and video generation
  - generate content from keyword to video automatically
  - build AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - use Remotion to generate videos from articles
  - crawl news and generate content with AI
  - automate social media content creation pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that transforms keywords into finished content and videos. It handles the entire pipeline: research/crawling news sources → AI content generation (multiple formats) → automated video rendering → ready-to-publish content.

**Key capabilities:**
- Auto-crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- Generates articles in multiple formats (toplist, POV, case study, how-to)
- Creates bilingual content (English/Vietnamese)
- Automatically renders videos and infographics via Remotion
- Optimized for social media platforms (Reels, TikTok, Shorts)

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

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Optional: Database for storing generated content
DATABASE_URL=your_database_url
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utilities
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (if separate script)
npm run render
```

## Core API Usage

### 1. Research & Crawling

```typescript
// src/lib/crawler/news-crawler.ts
import axios from 'axios';

interface NewsSource {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
}

export async function crawlRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsSource[]> {
  const results: NewsSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/news/search`,
        {
          params: {
            q: keyword,
            source: source,
            timeframe: '24h'
          },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
            'X-RapidAPI-Host': 'news-api.rapidapi.com'
          }
        }
      );
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Error crawling ${source}:`, error);
    }
  }
  
  return results;
}

// Usage in API route
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const keyword = searchParams.get('keyword') || '';
  
  const news = await crawlRecentNews(keyword);
  return Response.json({ news });
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ToneOfVoice = 'expert' | 'friendly' | 'humorous';

interface GenerateContentParams {
  keyword: string;
  researchData: string;
  format: ContentFormat;
  tone: ToneOfVoice;
  language: 'en' | 'vi';
}

export async function generateContent({
  keyword,
  researchData,
  format,
  tone,
  language
}: GenerateContentParams): Promise<string> {
  const formatPrompts = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write a perspective piece with personal insights and opinions',
    'case-study': 'Develop an in-depth case study with data and analysis',
    'how-to': 'Create a step-by-step tutorial guide'
  };

  const tonePrompts = {
    'expert': 'professional, authoritative, data-driven',
    'friendly': 'conversational, approachable, engaging',
    'humorous': 'witty, entertaining, light-hearted'
  };

  const prompt = `
You are a ${tonePrompts[tone]} content writer.

Task: ${formatPrompts[format]} about "${keyword}"

Research Data:
${researchData}

Language: ${language === 'vi' ? 'Vietnamese' : 'English'}

Create compelling, well-structured content that incorporates the latest research insights.
Include specific data points, examples, and actionable takeaways.
  `.trim();

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
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!,
});

export async function generateContentOpenAI({
  keyword,
  researchData,
  format,
  tone,
  language
}: GenerateContentParams): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer creating ${format} articles in ${language}.`
      },
      {
        role: 'user',
        content: `Create content about "${keyword}" using this research:\n\n${researchData}`
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

interface VideoContent {
  title: string;
  points: string[];
  backgroundImage?: string;
}

export async function renderContentVideo(
  content: VideoContent,
  outputPath: string
): Promise<string> {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });

  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    outputPath
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: content,
  });

  return outputLocation;
}
```

```tsx
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, spring } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  backgroundImage?: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  backgroundImage
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = spring({
    frame: frame - 10,
    fps,
    config: { damping: 100 },
  });

  return (
    <AbsoluteFill style={{
      backgroundColor: '#1a1a1a',
      backgroundImage: backgroundImage ? `url(${backgroundImage})` : undefined,
      backgroundSize: 'cover',
    }}>
      <div style={{
        position: 'absolute',
        top: '10%',
        width: '100%',
        textAlign: 'center',
        opacity: titleOpacity,
      }}>
        <h1 style={{
          fontSize: 60,
          color: 'white',
          fontWeight: 'bold',
          textShadow: '2px 2px 4px rgba(0,0,0,0.8)',
        }}>
          {title}
        </h1>
      </div>

      <div style={{
        position: 'absolute',
        top: '30%',
        width: '100%',
        padding: '0 80px',
      }}>
        {points.map((point, index) => {
          const pointOpacity = spring({
            frame: frame - (60 + index * 30),
            fps,
            config: { damping: 100 },
          });

          return (
            <div
              key={index}
              style={{
                opacity: pointOpacity,
                marginBottom: 30,
                fontSize: 32,
                color: 'white',
                backgroundColor: 'rgba(0,0,0,0.6)',
                padding: 20,
                borderRadius: 10,
              }}
            >
              {index + 1}. {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './ContentVideo';

registerRoot(() => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
});
```

## Complete Pipeline Example

```typescript
// src/app/api/generate-pipeline/route.ts
import { crawlRecentNews } from '@/lib/crawler/news-crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/render-video';

export async function POST(request: Request) {
  const { keyword, format, tone, language } = await request.json();

  try {
    // Step 1: Research
    const news = await crawlRecentNews(keyword);
    const researchData = news
      .map(n => `${n.title}\n${n.content}`)
      .join('\n\n');

    // Step 2: Generate Content
    const content = await generateContent({
      keyword,
      researchData,
      format,
      tone,
      language,
    });

    // Step 3: Extract key points for video
    const lines = content.split('\n').filter(line => 
      line.trim().match(/^\d+\./) || line.trim().startsWith('-')
    );
    const points = lines.slice(0, 5).map(l => 
      l.replace(/^\d+\.\s*|-\s*/, '').trim()
    );

    // Step 4: Generate Video
    const videoPath = await renderContentVideo({
      title: keyword,
      points,
    }, `${Date.now()}.mp4`);

    return Response.json({
      success: true,
      content,
      videoUrl: `/videos/${path.basename(videoPath)}`,
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return Response.json(
      { error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Batch Content Generation

```typescript
// src/lib/generator/batch-generator.ts
export async function generateBatchContent(
  keywords: string[],
  options: Partial<GenerateContentParams>
): Promise<Map<string, string>> {
  const results = new Map<string, string>();

  for (const keyword of keywords) {
    const news = await crawlRecentNews(keyword);
    const content = await generateContent({
      keyword,
      researchData: news.map(n => n.content).join('\n'),
      format: options.format || 'toplist',
      tone: options.tone || 'expert',
      language: options.language || 'en',
    });

    results.set(keyword, content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Bilingual Content Generation

```typescript
export async function generateBilingualContent(
  keyword: string,
  researchData: string,
  format: ContentFormat
): Promise<{ en: string; vi: string }> {
  const [enContent, viContent] = await Promise.all([
    generateContent({
      keyword,
      researchData,
      format,
      tone: 'expert',
      language: 'en',
    }),
    generateContent({
      keyword,
      researchData,
      format,
      tone: 'expert',
      language: 'vi',
    }),
  ]);

  return { en: enContent, vi: viContent };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
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

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift()!;
      await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    }
    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter();
const content = await limiter.add(() => generateContent(params));
```

### Video Rendering Errors

If Remotion fails to render:

1. Ensure all dependencies are installed: `npm install @remotion/bundler @remotion/renderer`
2. Check output directory exists: `mkdir -p public/videos`
3. Verify composition ID matches: Check `remotion/index.ts` composition ID

### Memory Issues with Large Batches

```typescript
// Process in chunks
async function processInChunks<T>(
  items: T[],
  chunkSize: number,
  processor: (chunk: T[]) => Promise<void>
) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    await processor(chunk);
  }
}
```

## Best Practices

1. **Always validate API keys** before starting pipeline
2. **Cache research data** to avoid redundant API calls
3. **Use queue systems** for video rendering (CPU-intensive)
4. **Store generated content** in database with metadata
5. **Implement retry logic** for failed API calls
6. **Monitor API usage** to stay within quotas
