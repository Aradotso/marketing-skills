---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scripting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation from research to video
  - generate blog posts and videos from keywords
  - crawl news and create content automatically
  - build AI-powered content pipeline
  - create multilingual content with Claude
  - auto-generate social media videos from articles
  - setup content automation workflow
  - research and write articles with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Ultimate AI Content Pipeline**, a TypeScript-based automation system that handles the entire content creation workflow: from news research and article generation to automated video rendering and social media posting.

## What It Does

The marketing-pipeline-share project provides:

- **Auto-Scan Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multilingual Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Automatically creates videos and infographics using Remotion
- **Platform Optimization**: Exports content optimized for Reels, TikTok, and YouTube Shorts

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

## Configuration

Create a `.env.local` file in the project root:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion configuration
REMOTION_TIMEOUT=300000
```

## Key Components

### 1. Research & Crawling Module

Automatically fetches and analyzes recent news:

```typescript
import { crawlNews } from '@/lib/crawler';
import { analyzeContent } from '@/lib/analyzer';

async function researchTopic(keyword: string) {
  const sources = {
    techcrunch: true,
    a16z: true,
    twitter: true,
    linkedin: true
  };
  
  const newsData = await crawlNews({
    keyword,
    sources,
    timeRange: '24h'
  });
  
  const insights = await analyzeContent(newsData);
  
  return {
    rawData: newsData,
    insights,
    dataPoints: insights.statistics
  };
}
```

### 2. AI Content Generation

Generate articles using Claude or OpenAI:

```typescript
import { generateArticle } from '@/lib/ai/generator';
import { ContentFormat, Language } from '@/types';

async function createContent(topic: string) {
  const config = {
    format: 'toplist' as ContentFormat, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'] as Language[],
    tone: 'expert', // 'expert' | 'friendly' | 'humorous'
    provider: 'claude' // 'claude' | 'openai'
  };
  
  const article = await generateArticle({
    topic,
    research: await researchTopic(topic),
    config
  });
  
  return article;
}
```

### 3. Video Rendering with Remotion

Convert articles to videos:

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoConfig } from '@/types/video';

async function generateVideo(articleId: string) {
  const videoConfig: VideoConfig = {
    format: 'reels', // 'reels' | 'tiktok' | 'shorts'
    aspectRatio: '9:16',
    duration: 30,
    template: 'infographic'
  };
  
  const video = await renderVideo({
    articleId,
    config: videoConfig,
    outputDir: './public/videos'
  });
  
  return video.url;
}
```

## API Routes

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { crawlNews } from '@/lib/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, sources } = req.body;
  
  try {
    const data = await crawlNews({ keyword, sources, timeRange: '24h' });
    res.status(200).json({ success: true, data });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateArticle } from '@/lib/ai/generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { topic, format, languages, provider } = req.body;
  
  try {
    const article = await generateArticle({
      topic,
      config: { format, languages, provider }
    });
    
    res.status(200).json({ success: true, article });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Video Rendering Endpoint

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { articleId, format } = req.body;
  
  try {
    const video = await renderVideo({
      articleId,
      config: { format, aspectRatio: '9:16' }
    });
    
    res.status(200).json({ success: true, videoUrl: video.url });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Complete Workflow Example

Full pipeline from keyword to video:

```typescript
import { researchTopic, createContent, generateVideo } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic(keyword);
    
    // Step 2: Generate article
    console.log('✍️ Generating content...');
    const article = await createContent({
      topic: keyword,
      research,
      config: {
        format: 'toplist',
        languages: ['en', 'vi'],
        tone: 'expert',
        provider: 'claude'
      }
    });
    
    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const video = await generateVideo(article.id);
    
    return {
      article: {
        id: article.id,
        title: article.title,
        content: article.content,
        languages: article.languages
      },
      video: {
        url: video.url,
        format: video.format
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI automation trends 2026')
  .then(result => console.log('✅ Pipeline completed:', result))
  .catch(error => console.error('❌ Pipeline failed:', error));
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Custom Content Templates

```typescript
import { defineTemplate } from '@/lib/templates';

const customTemplate = defineTemplate({
  name: 'product-launch',
  structure: {
    intro: { tone: 'exciting', length: 'short' },
    features: { format: 'bullets', count: 5 },
    benefits: { format: 'paragraphs', count: 3 },
    cta: { tone: 'urgent', length: 'medium' }
  }
});

const article = await generateArticle({
  topic: 'New SaaS Launch',
  template: customTemplate
});
```

### Video Customization

```typescript
import { VideoComposition } from '@/lib/video/composition';

const customVideo = new VideoComposition({
  intro: {
    duration: 3,
    animation: 'fade-in',
    text: article.title
  },
  body: {
    slides: article.keyPoints.map(point => ({
      duration: 5,
      text: point,
      background: 'gradient'
    }))
  },
  outro: {
    duration: 2,
    cta: 'Follow for more',
    logo: '/logo.png'
  }
});

await customVideo.render();
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run tests
npm test

# Lint code
npm run lint

# Type check
npm run type-check
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  timeWindow: 60000 // 1 minute
});

await limiter.execute(async () => {
  return await generateArticle(config);
});
```

### Video Rendering Timeout

Increase timeout in `.env.local`:

```bash
REMOTION_TIMEOUT=600000 # 10 minutes
```

Or programmatically:

```typescript
const video = await renderVideo({
  articleId,
  config: { format: 'reels' },
  timeout: 600000
});
```

### Claude API Errors

Handle token limits:

```typescript
import { chunkContent } from '@/lib/utils/chunking';

async function generateLongContent(topic: string) {
  const chunks = await chunkContent(topic, { maxTokens: 4000 });
  
  const sections = await Promise.all(
    chunks.map(chunk => generateArticle({ topic: chunk }))
  );
  
  return mergeArticles(sections);
}
```

### News Crawling Failures

Implement retry logic:

```typescript
import { retry } from '@/lib/utils/retry';

const newsData = await retry(
  () => crawlNews({ keyword, sources }),
  { attempts: 3, delay: 2000 }
);
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Queue video rendering** for resource-intensive operations
3. **Validate API keys** before pipeline execution
4. **Use webhooks** for async operations and notifications
5. **Store generated content** in database for versioning
6. **Implement analytics** to track content performance
