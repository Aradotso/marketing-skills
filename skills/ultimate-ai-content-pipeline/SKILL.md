---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I use the AI content pipeline to generate articles
  - set up automated content research and video generation
  - create marketing content with Claude and OpenAI integration
  - automate social media content creation with AI
  - generate videos from articles using Remotion
  - configure the content pipeline for multi-language posts
  - research and write trending content automatically
  - build an AI-powered content factory
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles research, script generation, article writing, and video rendering. Leverages Claude 3, OpenAI, web scraping, and Remotion to create a complete content production pipeline.

## Overview

This project automates the entire content creation workflow:

1. **Research Phase**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending topics
2. **Content Generation**: Uses Claude/OpenAI to write articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Creation**: Renders infographics and short-form videos via Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── research/    # Content research & scraping
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Topic Discovery

```typescript
import { researchTrends } from '@/lib/research/trends';

async function findTrendingTopics(keyword: string) {
  const research = await researchTrends({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });

  return {
    topics: research.topics,
    insights: research.insights,
    dataPoints: research.statistics
  };
}

// Usage
const trends = await findTrendingTopics('AI automation');
console.log(trends.topics); // Array of trending topics with scores
```

### 2. Content Generation with Claude

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateArticle(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const prompt = `Write a ${format} article about ${topic}. Include:
- Engaging headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure`;

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

  return message.content[0].text;
}

// Usage
const article = await generateArticle('AI in Marketing', 'how-to');
```

### 3. Multi-Language Content Generation

```typescript
import { translateContent } from '@/lib/ai/translator';

async function generateBilingualContent(topic: string) {
  // Generate English version
  const englishContent = await generateArticle(topic, 'toplist');
  
  // Auto-translate to Vietnamese
  const vietnameseContent = await translateContent(englishContent, {
    from: 'en',
    to: 'vi',
    tone: 'professional'
  });

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpack } from '@remotion/cli';

async function renderContentVideo(article: {
  title: string;
  keyPoints: string[];
  stats: { label: string; value: string }[];
}) {
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: webpack
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'InfoGraphic',
    inputProps: {
      title: article.title,
      keyPoints: article.keyPoints,
      stats: article.stats
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${article.title.replace(/\s+/g, '-')}.mp4`,
    inputProps: composition.defaultProps
  });
}

// Usage
await renderContentVideo({
  title: 'AI Marketing Trends 2024',
  keyPoints: ['Point 1', 'Point 2', 'Point 3'],
  stats: [
    { label: 'ROI Increase', value: '300%' },
    { label: 'Time Saved', value: '90%' }
  ]
});
```

### 5. Complete Pipeline Example

```typescript
import { runContentPipeline } from '@/lib/pipeline';

async function createCompleteContent(keyword: string) {
  const pipeline = await runContentPipeline({
    keyword,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    includeVideo: true,
    videoFormats: ['reels', 'tiktok', 'shorts']
  });

  return {
    research: pipeline.research,
    articles: pipeline.articles,
    videos: pipeline.videos,
    metadata: {
      createdAt: new Date(),
      aiModels: ['claude-3-5-sonnet', 'gpt-4'],
      sources: pipeline.sources
    }
  };
}

// Usage
const content = await createCompleteContent('ChatGPT plugins');
/*
Returns:
{
  research: { topics: [...], insights: [...], stats: [...] },
  articles: {
    en: { toplist: '...', howTo: '...' },
    vi: { toplist: '...', howTo: '...' }
  },
  videos: {
    reels: 'out/video-reels.mp4',
    tiktok: 'out/video-tiktok.mp4',
    shorts: 'out/video-shorts.mp4'
  }
}
*/
```

### 6. API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { generateArticle } from '@/lib/content/generator';

export async function POST(request: Request) {
  const { topic, format, language } = await request.json();

  try {
    const content = await generateArticle(topic, format);
    
    return NextResponse.json({
      success: true,
      content,
      language
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### 7. Research Scraper Configuration

```typescript
import { setupScraper } from '@/lib/research/scraper';

const scraper = setupScraper({
  sources: [
    {
      name: 'techcrunch',
      url: 'https://techcrunch.com/category/artificial-intelligence/',
      selector: '.post-block',
      rateLimit: 1000 // ms between requests
    },
    {
      name: 'a16z',
      url: 'https://a16z.com/posts/',
      selector: 'article',
      rateLimit: 1500
    }
  ],
  filters: {
    minWordCount: 500,
    maxAge: '24h',
    relevanceThreshold: 0.7
  }
});

const articles = await scraper.fetch('AI automation');
```

## Common Workflows

### Daily Content Automation

```typescript
import { schedulePipeline } from '@/lib/scheduler';

// Schedule daily content generation
schedulePipeline({
  cron: '0 9 * * *', // 9 AM daily
  keywords: ['AI', 'Marketing Tech', 'Automation'],
  autoPost: true,
  platforms: ['facebook', 'linkedin', 'twitter']
});
```

### Batch Content Generation

```typescript
async function batchGenerate(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(topic => 
      runContentPipeline({
        keyword: topic,
        formats: ['toplist'],
        languages: ['en', 'vi'],
        includeVideo: false
      })
    )
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

## Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion videos locally
npm run remotion:preview

# Type checking
npm run type-check
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callWithRetry(apiCall: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await apiCall();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

### Claude Context Length

```typescript
// Split long content into chunks
function chunkContent(text: string, maxTokens = 3000) {
  const chunks = [];
  const paragraphs = text.split('\n\n');
  let currentChunk = '';
  
  for (const para of paragraphs) {
    if ((currentChunk + para).length > maxTokens * 4) {
      chunks.push(currentChunk);
      currentChunk = para;
    } else {
      currentChunk += '\n\n' + para;
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

### Scraping Blocks

```typescript
// Use rotating user agents and delays
const userAgents = [
  'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...',
  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)...'
];

async function scrapeWithRotation(url: string) {
  const randomUA = userAgents[Math.floor(Math.random() * userAgents.length)];
  
  await new Promise(r => setTimeout(r, 1000 + Math.random() * 2000));
  
  return fetch(url, {
    headers: { 'User-Agent': randomUA }
  });
}
```

## Best Practices

1. **Rate Limiting**: Always respect API rate limits, especially for Claude and OpenAI
2. **Caching**: Cache research results to avoid redundant API calls
3. **Error Handling**: Implement comprehensive error handling for each pipeline stage
4. **Content Quality**: Review AI-generated content before auto-posting
5. **Video Optimization**: Use appropriate video codecs and resolutions for target platforms
