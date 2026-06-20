---
name: ultimate-ai-content-pipeline
description: Full-stack TypeScript content automation system for research, script generation, video creation and auto-posting
triggers:
  - automate content creation with AI research and video generation
  - set up an AI content pipeline with Claude and Remotion
  - build automated marketing content workflow
  - create videos from blog posts automatically
  - integrate content research and auto-posting system
  - generate multi-format content with AI and render videos
  - implement content automation with OpenAI and Claude
  - scrape news and generate social media videos
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What It Does

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content lifecycle:

1. **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Uses Claude 3/OpenAI to create multi-format content (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates Vietnamese and English versions simultaneously
4. **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
5. **Auto-Publishing**: Posts content to social platforms

Built with Next.js, it provides a complete end-to-end solution for content marketers and creators to produce data-backed, trendy content with minimal manual effort.

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

## Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media Integration (optional)
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token_here
LINKEDIN_ACCESS_TOKEN=your_linkedin_token_here

# Remotion Configuration
REMOTION_TIMEOUT=120000
REMOTION_CONCURRENCY=1

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── publisher/   # Social media posting
│   ├── services/        # Business logic services
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── prisma/              # Database schema
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { analyzeInsights } from '@/lib/ai/insight-analyzer';

// Crawl latest news by keyword
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const articles = await crawlNews({
    keyword,
    sources,
    timeRange: '24h',
    limit: 50
  });

  // Use AI to extract insights
  const insights = await analyzeInsights({
    articles,
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-opus-20240229'
  });

  return {
    articles,
    insights,
    stats: {
      totalArticles: articles.length,
      topAuthors: insights.topContributors,
      trendingTopics: insights.trends
    }
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate multi-format content
async function createContent(topic: string, insights: any) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'];
  
  const content = await generateContent({
    topic,
    insights,
    format: 'toplist', // or any format from array
    languages: ['en', 'vi'],
    tone: 'expert', // 'friendly', 'humorous'
    provider: 'claude', // or 'openai'
    model: 'claude-3-sonnet-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return content;
  // Returns: { en: { title, body, metadata }, vi: { title, body, metadata } }
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Render video from content
async function generateVideo(content: any, format: 'reel' | 'tiktok' | 'short') {
  const compositionId = format === 'reel' ? 'InstagramReel' : 'TikTokVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      bgColor: '#1a1a1a',
      accentColor: '#00ff88'
    },
  });

  // Render video
  const outputPath = path.resolve(`./public/videos/${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps,
  });

  return outputPath;
}
```

### 4. Auto-Publishing

```typescript
import { publishToFacebook, publishToLinkedIn } from '@/lib/publisher';

// Publish content to social platforms
async function publishContent(content: any, videoPath: string) {
  const results = await Promise.allSettled([
    publishToFacebook({
      pageId: process.env.FACEBOOK_PAGE_ID,
      accessToken: process.env.FACEBOOK_PAGE_ACCESS_TOKEN,
      message: content.en.body.substring(0, 500),
      videoPath
    }),
    
    publishToLinkedIn({
      accessToken: process.env.LINKEDIN_ACCESS_TOKEN,
      text: content.en.body,
      mediaUrn: videoPath // Requires upload first
    })
  ]);

  return results;
}
```

## Complete Pipeline Example

```typescript
import { researchTopic } from '@/services/research';
import { generateContent } from '@/services/content';
import { generateVideo } from '@/services/video';
import { publishContent } from '@/services/publisher';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic(keyword);
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent(keyword, research.insights);
    
    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo(content.en, 'reel');
    
    // Step 4: Publish
    console.log('📤 Publishing...');
    const published = await publishContent(content, videoPath);
    
    return {
      success: true,
      research,
      content,
      videoPath,
      published
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
runContentPipeline('AI automation trends 2026')
  .then(result => console.log('✅ Pipeline completed:', result))
  .catch(err => console.error('❌ Pipeline failed:', err));
```

## Common Patterns

### Custom Content Templates

```typescript
// Define custom content template
export const customTemplate = {
  name: 'myth-busting',
  structure: {
    intro: 'hook + problem statement',
    myths: ['myth 1', 'myth 2', 'myth 3'],
    reality: 'data-backed truth',
    conclusion: 'actionable takeaway'
  },
  prompt: `
    Create a myth-busting article about {topic}.
    Use recent data from: {insights}
    Debunk 3 common myths with statistics.
    Tone: {tone}
  `
};

// Use template
const content = await generateContent({
  topic: 'AI in marketing',
  insights: research.insights,
  template: customTemplate,
  tone: 'expert'
});
```

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['SaaS trends', 'Marketing automation', 'AI tools'];
  
  for (const topic of topics) {
    await runContentPipeline(topic);
    await new Promise(resolve => setTimeout(resolve, 5000)); // Rate limit
  }
});
```

### Batch Processing

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline(keyword);
    results.push(result);
    
    // Save to database
    await saveToDatabase({
      keyword,
      content: result.content,
      videoUrl: result.videoPath,
      publishedAt: new Date()
    });
  }
  
  return results;
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run linter
npm run lint

# Test Remotion compositions
npm run remotion:preview

# Render specific video template
npm run remotion:render -- <composition-id>

# Database migrations
npx prisma migrate dev
npx prisma studio
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function withRetry<T>(
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

// Usage
const content = await withRetry(() => 
  generateContent({ topic, insights, apiKey: process.env.OPENAI_API_KEY })
);
```

### Video Rendering Timeout

```typescript
// Increase timeout in config
const outputPath = await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: parseInt(process.env.REMOTION_TIMEOUT || '120000'),
  chromiumOptions: {
    headless: true,
    gl: 'swiftshader'
  }
});
```

### Memory Issues with Large Crawls

```typescript
// Use streaming for large datasets
import { pipeline } from 'stream/promises';

async function streamCrawl(keyword: string) {
  const chunkSize = 10;
  let offset = 0;
  const allResults = [];
  
  while (true) {
    const chunk = await crawlNews({
      keyword,
      limit: chunkSize,
      offset
    });
    
    if (chunk.length === 0) break;
    
    allResults.push(...chunk);
    offset += chunkSize;
    
    // Process chunk immediately
    await processChunk(chunk);
  }
  
  return allResults;
}
```

### Environment-Specific Configuration

```typescript
// lib/config.ts
export const config = {
  ai: {
    provider: process.env.AI_PROVIDER || 'claude',
    model: process.env.AI_MODEL || 'claude-3-sonnet-20240229',
    maxTokens: parseInt(process.env.AI_MAX_TOKENS || '4096')
  },
  crawler: {
    timeout: parseInt(process.env.CRAWLER_TIMEOUT || '30000'),
    maxPages: parseInt(process.env.CRAWLER_MAX_PAGES || '50')
  },
  video: {
    defaultFormat: process.env.VIDEO_FORMAT || 'reel',
    quality: process.env.VIDEO_QUALITY || 'high'
  }
};
```
