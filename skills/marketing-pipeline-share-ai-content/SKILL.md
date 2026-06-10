---
name: marketing-pipeline-share-ai-content
description: Automated content pipeline that researches, writes scripts, and generates videos using Claude/OpenAI and Remotion
triggers:
  - automate my content creation workflow
  - set up AI content research and video generation
  - generate blog posts from trending topics automatically
  - create videos from written content using Remotion
  - build an automated marketing content pipeline
  - research and write content with Claude AI
  - generate multi-format content with AI
  - automate content from research to video
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, a complete content automation system that researches trending topics, generates multi-format content in multiple languages, and automatically renders videos using AI (Claude 3, OpenAI) and Remotion.

## What It Does

The system provides an end-to-end content pipeline:

1. **Auto-Scan Research** - Crawls recent content from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation** - Creates content in various formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
3. **Multi-language Output** - Generates Vietnamese and English versions simultaneously
4. **Video/Image Rendering** - Automatically creates infographics and short videos using Remotion
5. **Platform Optimization** - Exports video in formats optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom configurations
NEXT_PUBLIC_API_URL=http://localhost:3000
```

Never commit API keys to version control. Use environment variables.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content scraping & analysis
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── .env.local          # Environment variables
```

## Key API Patterns

### 1. Content Research

```typescript
import { researchTopic } from '@/lib/research/scraper';

async function fetchLatestContent(topic: string) {
  const research = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends
  };
}

// Usage
const aiContent = await fetchLatestContent('artificial intelligence marketing');
console.log(aiContent.insights); // Deep insights extracted by AI
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateContent(
  research: ResearchData,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Based on this research data: ${JSON.stringify(research)}
      
Create a ${format} article in ${language} language.
Tone: Professional yet engaging
Include: Data-backed insights, trending topics, actionable takeaways`
    }]
  });
  
  return message.content[0].text;
}

// Usage
const article = await generateContent(
  aiContent,
  'toplist',
  'vi'
);
```

### 3. Dual Language Generation

```typescript
interface MultiLangContent {
  en: string;
  vi: string;
  metadata: {
    format: string;
    tone: string;
    wordCount: Record<'en' | 'vi', number>;
  };
}

async function generateDualLanguage(
  research: ResearchData,
  format: ContentFormat
): Promise<MultiLangContent> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, format, 'en'),
    generateContent(research, format, 'vi')
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
    metadata: {
      format,
      tone: 'professional',
      wordCount: {
        en: englishContent.split(' ').length,
        vi: vietnameseContent.split(' ').length
      }
    }
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';

async function generateVideo(content: string, format: 'reels' | 'tiktok' | 'shorts') {
  const compositionId = 'ContentVideo';
  
  // Aspect ratios for different platforms
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride
  });
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: compositionId,
    inputProps: {
      content,
      platform: format
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${format}-${Date.now()}.mp4`,
    inputProps: {
      content,
      ...dimensions[format]
    }
  });
}

// Usage
await generateVideo(article, 'reels');
```

### 5. Complete Pipeline

```typescript
interface ContentPipelineConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoFormats?: ('reels' | 'tiktok' | 'shorts')[];
}

async function runContentPipeline(config: ContentPipelineConfig) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword: config.topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });
  
  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateDualLanguage(research, config.format);
  
  // Step 3: Generate Videos (optional)
  if (config.generateVideo && config.videoFormats) {
    console.log('🎬 Rendering videos...');
    for (const format of config.videoFormats) {
      await generateVideo(content.en, format);
    }
  }
  
  return {
    research,
    content,
    timestamp: new Date().toISOString()
  };
}

// Full automation example
const result = await runContentPipeline({
  topic: 'AI in marketing automation 2026',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormats: ['reels', 'tiktok']
});
```

## Next.js API Routes

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scraper';

export async function POST(request: NextRequest) {
  const { topic, sources, timeRange } = await request.json();
  
  try {
    const research = await researchTopic({
      keyword: topic,
      sources: sources || ['techcrunch', 'a16z', 'twitter'],
      timeRange: timeRange || '24h',
      language: 'en'
    });
    
    return NextResponse.json(research);
  } catch (error) {
    return NextResponse.json(
      { error: 'Research failed' },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateDualLanguage } from '@/lib/ai/generate';

export async function POST(request: NextRequest) {
  const { research, format } = await request.json();
  
  try {
    const content = await generateDualLanguage(research, format);
    
    return NextResponse.json(content);
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Common Workflows

### Workflow 1: Quick Article Generation

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/generate';

async function quickArticle(topic: string) {
  const research = await researchTopic({ keyword: topic });
  const article = await generateContent(research, 'how-to', 'en');
  return article;
}
```

### Workflow 2: Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run every day at 6 AM
cron.schedule('0 6 * * *', async () => {
  const topics = [
    'AI marketing trends',
    'social media automation',
    'content strategy 2026'
  ];
  
  for (const topic of topics) {
    await runContentPipeline({
      topic,
      format: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true,
      videoFormats: ['reels']
    });
  }
});
```

### Workflow 3: Batch Processing

```typescript
async function batchGenerate(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(topic => runContentPipeline({
      topic,
      format: 'pov',
      languages: ['en'],
      generateVideo: false
    }))
  );
  
  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');
  
  return { successful, failed };
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Type checking
npm run type-check

# Remotion studio (for video editing)
npm run remotion:studio

# Render single video
npm run remotion:render
```

## TypeScript Types

```typescript
// Core types for the pipeline
interface ResearchData {
  articles: Article[];
  insights: string[];
  trends: Trend[];
  sources: string[];
  timestamp: string;
}

interface Article {
  title: string;
  url: string;
  source: string;
  publishedAt: string;
  summary: string;
}

interface ContentFormat {
  type: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  targetAudience: string;
}

interface VideoConfig {
  platform: 'reels' | 'tiktok' | 'shorts';
  duration: number;
  aspectRatio: string;
  fps: number;
}
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
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries reached');
}

// Usage
const content = await retryWithBackoff(() => 
  generateContent(research, 'toplist', 'en')
);
```

### Video Rendering Memory Issues

```typescript
// For large videos, use concurrency control
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent renders

async function renderMultipleVideos(contents: string[]) {
  const tasks = contents.map((content, i) => 
    limit(() => generateVideo(content, 'reels'))
  );
  
  return Promise.all(tasks);
}
```

### Claude API Errors

```typescript
// Handle Claude-specific errors
async function safeGenerateContent(research: ResearchData) {
  try {
    return await generateContent(research, 'toplist', 'en');
  } catch (error: any) {
    if (error.status === 429) {
      console.error('Rate limit hit, waiting...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeGenerateContent(research);
    }
    
    if (error.status === 529) {
      console.error('Claude overloaded, falling back to OpenAI');
      return generateWithOpenAI(research);
    }
    
    throw error;
  }
}
```

## Best Practices

1. **Always validate research data** before passing to AI generation
2. **Cache research results** to avoid redundant API calls
3. **Use streaming** for real-time content generation feedback
4. **Implement proper error handling** for each pipeline stage
5. **Monitor API usage** to stay within rate limits
6. **Version control prompts** separately for easy A/B testing
7. **Store generated content** with metadata for analytics

This skill enables full automation of content marketing workflows from research to video production.
