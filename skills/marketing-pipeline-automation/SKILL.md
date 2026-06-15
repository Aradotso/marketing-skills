---
name: marketing-pipeline-automation
description: AI-powered content automation system from research to video generation with Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate videos from articles automatically
  - research and write marketing content with AI
  - create multilingual content with Claude and OpenAI
  - build automated content workflow
  - scrape news and generate blog posts
  - set up AI content generation pipeline
  - automate video rendering from text content
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a complete content automation system that handles research, scriptwriting, article generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What It Does

The marketing pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
3. **Multilingual Output**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Multi-platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # News scraping modules
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Scraping

```typescript
import { scrapeLatestNews } from '@/lib/scraper/news-aggregator';

interface NewsSource {
  url: string;
  selector: string;
  source: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
}

async function gatherResearch(topic: string): Promise<Article[]> {
  const sources: NewsSource[] = [
    { url: 'https://techcrunch.com', selector: '.post-block', source: 'techcrunch' },
    { url: 'https://a16z.com/articles', selector: '.article', source: 'a16z' }
  ];
  
  const articles = await scrapeLatestNews({
    topic,
    sources,
    timeframe: '24h',
    maxResults: 20
  });
  
  return articles;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: string;
}

async function generateArticle(config: ContentConfig): Promise<string> {
  const prompt = `
You are a professional content writer. Create a ${config.format} article in ${config.language} with a ${config.tone} tone.

Research data:
${config.researchData}

Generate a comprehensive article with:
- Engaging headline
- Introduction with hook
- Main content sections with data-backed insights
- Conclusion with call-to-action
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  topic: string, 
  researchData: string
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content creator.'
      },
      {
        role: 'user',
        content: `Create an engaging article about ${topic} using this research: ${researchData}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  aspectRatio: '16:9' | '9:16' | '1:1';
  platform: 'reels' | 'tiktok' | 'shorts' | 'youtube';
}

async function renderContentVideo(config: VideoConfig): Promise<string> {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      aspectRatio: config.aspectRatio,
    },
  });

  // Render video
  const outputPath = path.resolve(`./output/video-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return outputPath;
}
```

## Complete Pipeline Example

```typescript
import { scrapeLatestNews } from '@/lib/scraper/news-aggregator';
import { generateArticle } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/remotion-renderer';

interface PipelineResult {
  article: string;
  videoPath: string;
  metadata: {
    topic: string;
    language: string;
    sources: number;
  };
}

export async function runContentPipeline(
  topic: string,
  language: 'en' | 'vi' = 'en'
): Promise<PipelineResult> {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research data...');
    const articles = await scrapeLatestNews({
      topic,
      timeframe: '24h',
      maxResults: 15
    });

    const researchData = articles
      .map(a => `${a.title}: ${a.summary}`)
      .join('\n\n');

    // Step 2: Generate Article
    console.log('✍️ Generating article...');
    const article = await generateArticle({
      format: 'toplist',
      tone: 'expert',
      language,
      researchData
    });

    // Step 3: Extract key points for video
    const keyPoints = article
      .split('\n')
      .filter(line => line.match(/^\d+\.|^-/))
      .slice(0, 5);

    // Step 4: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: topic,
      content: keyPoints,
      aspectRatio: '9:16',
      platform: 'reels'
    });

    return {
      article,
      videoPath,
      metadata: {
        topic,
        language,
        sources: articles.length
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## Common Patterns

### Multi-Language Generation

```typescript
async function generateMultilingualContent(topic: string) {
  const [englishArticle, vietnameseArticle] = await Promise.all([
    generateArticle({
      format: 'how-to',
      tone: 'friendly',
      language: 'en',
      researchData: research
    }),
    generateArticle({
      format: 'how-to',
      tone: 'friendly',
      language: 'vi',
      researchData: research
    })
  ]);

  return { en: englishArticle, vi: vietnameseArticle };
}
```

### Batch Processing

```typescript
async function processBatchTopics(topics: string[]) {
  const results = [];
  
  for (const topic of topics) {
    const result = await runContentPipeline(topic);
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Video Templates

```typescript
// remotion/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  content: string[];
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <h1 style={{ opacity, color: 'white', padding: 40 }}>
        {title}
      </h1>
      {content.map((item, i) => (
        <p key={i} style={{ color: 'white', padding: 20 }}>
          {item}
        </p>
      ))}
    </AbsoluteFill>
  );
};
```

## CLI Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render a single video
npm run render -- --id=ContentVideo --output=output.mp4

# Type checking
npm run type-check

# Lint code
npm run lint
```

## API Routes

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: Request) {
  const { topic, language, format } = await request.json();

  try {
    const result = await runContentPipeline(topic, language);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Usage from Frontend

```typescript
// components/ContentGenerator.tsx
async function handleGenerate(topic: string) {
  const response = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      topic,
      language: 'en',
      format: 'toplist'
    })
  });

  const result = await response.json();
  return result.data;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion settings
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  },
  // Reduce memory usage
  concurrency: 2,
  everyNthFrame: 2 // Lower quality for faster rendering
});
```

### Scraping Errors

```typescript
// Add error handling and fallbacks
async function safeScrape(url: string) {
  try {
    return await scrapeLatestNews({ url });
  } catch (error) {
    console.warn(`Scraping failed for ${url}:`, error);
    return []; // Return empty array as fallback
  }
}
```

### TypeScript Type Safety

```typescript
// Define strict types for content
interface ArticleMetadata {
  title: string;
  author?: string;
  publishDate: Date;
  sources: string[];
  language: 'en' | 'vi';
  format: ContentFormat;
}

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';

function validateArticle(data: unknown): ArticleMetadata {
  // Runtime validation
  if (!data || typeof data !== 'object') {
    throw new Error('Invalid article data');
  }
  // Add more validation logic
  return data as ArticleMetadata;
}
```
