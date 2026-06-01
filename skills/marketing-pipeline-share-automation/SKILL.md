---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content research and generation
  - set up AI content pipeline with video rendering
  - create automated marketing content workflow
  - generate social media content with AI
  - build content automation using Claude and OpenAI
  - automate research to video content pipeline
  - use remotion for automated video generation
  - set up multi-language content generation system
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is an end-to-end automated content generation system that handles research, script writing, and video generation. It crawls real-time data from sources like TechCrunch, Twitter, and LinkedIn, generates multi-format content in multiple languages using Claude/OpenAI, and automatically renders videos using Remotion.

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
cp .env.example .env
```

## Environment Configuration

Configure your `.env` file with the following keys:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Content Research & Crawling

```typescript
import { researchTopic } from '@/lib/crawler/research';

// Automatically crawl and analyze content from multiple sources
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });

  return {
    insights: research.insights,
    dataPoints: research.statistics,
    trending: research.trendingTopics
  };
}

// Usage
const data = await gatherResearch('AI marketing automation');
console.log(data.insights); // Key findings from sources
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate content using Claude or OpenAI
async function createContent(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format, // 'toplist', 'pov', 'case-study', 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'professional', 'friendly', 'humorous'
    provider: 'claude', // 'claude' or 'openai'
    includeResearch: true
  });

  return content;
}

// Example: Generate bilingual top list
const article = await createContent(
  'Top 10 AI Tools for Marketers 2026',
  'toplist'
);

console.log(article.en.title);
console.log(article.vi.title);
console.log(article.en.body);
```

### 3. Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, context: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${context}\n\n${prompt}`
      }
    ],
  });

  return message.content[0].text;
}

// Generate script with research context
const script = await generateWithClaude(
  'Write a 60-second video script about this topic',
  researchData.insights.join('\n')
);
```

### 4. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  systemPrompt: string,
  userPrompt: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}

// Example usage
const content = await generateWithGPT(
  'You are a marketing content expert. Create engaging, data-driven content.',
  'Write a case study about AI in content marketing'
);
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpack } from '@remotion/webpack-config';

// Render video from content
async function renderContentVideo(content: any) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: 'modern'
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
  });

  return `out/${content.id}.mp4`;
}
```

### 6. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    languages: ['en', 'vi']
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);
  
  // Step 2: Generate content
  const content = await pipeline.generate({
    research,
    format: 'toplist',
    tone: 'professional'
  });
  
  // Step 3: Create video
  const video = await pipeline.renderVideo(content);
  
  // Step 4: Export results
  return {
    article: content,
    videoPath: video.path,
    thumbnails: video.thumbnails,
    socialPosts: content.socialMedia
  };
}

// Execute pipeline
const result = await runFullPipeline('AI automation tools');
console.log('Content ready:', result.article.en.title);
console.log('Video rendered:', result.videoPath);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextResponse } from 'next/server';
import { researchTopic } from '@/lib/crawler/research';

export async function POST(request: Request) {
  const { keyword } = await request.json();
  
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  return NextResponse.json(research);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: Request) {
  const { topic, format, languages } = await request.json();
  
  const content = await generateContent({
    topic,
    format,
    languages,
    provider: 'claude'
  });
  
  return NextResponse.json(content);
}
```

### Video Rendering Endpoint

```typescript
// app/api/render/route.ts
import { NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: Request) {
  const { content } = await request.json();
  
  const videoPath = await renderContentVideo(content);
  
  return NextResponse.json({ 
    success: true, 
    path: videoPath 
  });
}
```

## Common Patterns

### Multi-Format Content Generation

```typescript
const formats = ['toplist', 'pov', 'case-study', 'how-to'];

async function generateMultiFormat(topic: string) {
  const results = await Promise.all(
    formats.map(format => 
      generateContent({ 
        topic, 
        format,
        languages: ['en', 'vi']
      })
    )
  );
  
  return results;
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run content generation daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  const trendingTopics = await getTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runFullPipeline(topic);
  }
});
```

### Batch Video Rendering

```typescript
async function batchRenderVideos(contents: any[]) {
  const renders = contents.map(content => ({
    id: content.id,
    promise: renderContentVideo(content)
  }));
  
  const results = await Promise.allSettled(
    renders.map(r => r.promise)
  );
  
  return results.map((result, i) => ({
    id: renders[i].id,
    status: result.status,
    path: result.status === 'fulfilled' ? result.value : null
  }));
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run type checking
npm run type-check

# Render Remotion video locally
npm run render

# Bundle Remotion composition
npm run bundle
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(fn: Function, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Process videos in batches to avoid memory exhaustion
async function renderInBatches(contents: any[], batchSize = 3) {
  const batches = [];
  for (let i = 0; i < contents.length; i += batchSize) {
    batches.push(contents.slice(i, i + batchSize));
  }
  
  const results = [];
  for (const batch of batches) {
    const batchResults = await batchRenderVideos(batch);
    results.push(...batchResults);
  }
  
  return results;
}
```

### Research Data Quality

```typescript
// Validate and filter research data
function validateResearch(research: any) {
  return {
    insights: research.insights.filter(
      (insight: string) => insight.length > 50 && insight.length < 500
    ),
    statistics: research.statistics.filter(
      (stat: any) => stat.source && stat.value
    ),
    trending: research.trendingTopics.slice(0, 10)
  };
}
```

### Language Generation Issues

```typescript
// Ensure consistent bilingual output
async function ensureBilingualContent(content: any) {
  if (!content.vi || !content.vi.body) {
    content.vi = await generateContent({
      topic: content.en.title,
      format: content.format,
      languages: ['vi'],
      provider: 'claude'
    });
  }
  return content;
}
```

## Best Practices

1. **Always validate API keys before running pipelines**
2. **Use environment variables for all sensitive credentials**
3. **Implement proper error handling for external API calls**
4. **Cache research data to avoid redundant API calls**
5. **Monitor video rendering resource usage**
6. **Test content quality with sample topics before production**
7. **Use TypeScript strict mode for type safety**
8. **Implement logging for debugging pipeline stages**
