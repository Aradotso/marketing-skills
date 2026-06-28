---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, social media posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - set up AI marketing pipeline
  - generate videos from blog posts automatically
  - crawl news and create social media content
  - build automated content workflow with AI
  - create multi-format content with Claude and OpenAI
  - schedule auto-posting to social media pages
  - render videos from text using Remotion
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to work with **marketing-pipeline-share**, an end-to-end content automation system that researches topics, generates scripts, auto-posts to social media, and renders videos. Built with TypeScript, Next.js, and integrates Claude 3, OpenAI, and Remotion for video generation.

## What This Project Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese with customizable tone
4. **Auto-Posting**: Publishes directly to social media pages
5. **Video Generation**: Renders infographics and short videos using Remotion for Reels, TikTok, Shorts

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

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create `.env.local` with the following required API keys:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Social Media (if auto-posting)
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token
FACEBOOK_PAGE_ID=your_page_id

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app router
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI integration (Claude, OpenAI)
│   │   ├── crawler/        # News crawling logic
│   │   ├── generators/     # Content generators
│   │   └── social/         # Social media posting
│   ├── remotion/           # Video templates
│   └── types/              # TypeScript definitions
├── public/                 # Static assets
└── package.json
```

## Key Usage Patterns

### 1. Research and Crawl News

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { analyzeContent } from '@/lib/ai/content-analyzer';

async function researchTopic(keyword: string) {
  // Crawl recent news from multiple sources
  const articles = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Analyze and extract insights using AI
  const insights = await analyzeContent(articles, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    extractTopics: true,
    generateSummary: true
  });

  return insights;
}
```

### 2. Generate Content in Multiple Formats

```typescript
import { generateArticle } from '@/lib/generators/article-generator';

async function createContent(topic: string, format: string) {
  const article = await generateArticle({
    topic,
    format: format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'en', // or 'vi'
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    aiProvider: 'claude',
    includeStats: true,
    wordCount: 1500
  });

  return article;
}

// Example usage
const content = await createContent(
  'AI in marketing automation',
  'toplist'
);

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. Multi-language Content Generation

```typescript
import { generateMultilingualContent } from '@/lib/generators/multilingual';

async function createBilingualPost(topic: string) {
  const content = await generateMultilingualContent({
    topic,
    languages: ['en', 'vi'],
    format: 'pov',
    tone: 'friendly',
    syncMetadata: true
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    sharedHashtags: content.hashtags
  };
}
```

### 4. Auto-Post to Social Media

```typescript
import { postToFacebook } from '@/lib/social/facebook-poster';
import { schedulePost } from '@/lib/social/scheduler';

async function publishContent(content: any) {
  // Post immediately
  const result = await postToFacebook({
    pageId: process.env.FACEBOOK_PAGE_ID!,
    accessToken: process.env.FACEBOOK_PAGE_ACCESS_TOKEN!,
    message: content.body,
    link: content.url,
    published: true
  });

  return result;
}

// Schedule for later
async function scheduleContent(content: any, publishTime: Date) {
  const scheduled = await schedulePost({
    platform: 'facebook',
    content,
    scheduledTime: publishTime,
    timezone: 'Asia/Ho_Chi_Minh'
  });

  return scheduled;
}
```

### 5. Render Videos with Remotion

```typescript
import { renderVideo } from '@/lib/remotion/video-renderer';
import { createComposition } from '@/lib/remotion/compositions';

async function generateVideoFromContent(article: any) {
  // Create video composition from article
  const composition = createComposition({
    type: 'infographic', // 'infographic' | 'shorts' | 'reels'
    title: article.title,
    points: article.keyPoints,
    duration: 30, // seconds
    aspectRatio: '9:16', // vertical for Reels/TikTok
    style: 'modern'
  });

  // Render video
  const video = await renderVideo({
    composition,
    outputFormat: 'mp4',
    quality: 'high',
    fps: 30
  });

  return {
    videoUrl: video.url,
    thumbnail: video.thumbnail,
    duration: video.duration
  };
}
```

### 6. Complete Pipeline Example

```typescript
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

async function executeFullPipeline(keyword: string) {
  const result = await runContentPipeline({
    // Step 1: Research
    research: {
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    },
    
    // Step 2: Generate Content
    generation: {
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'professional',
      aiProvider: 'claude'
    },
    
    // Step 3: Create Video
    video: {
      enabled: true,
      type: 'reels',
      aspectRatio: '9:16'
    },
    
    // Step 4: Auto-publish
    publish: {
      platforms: ['facebook', 'linkedin'],
      schedule: new Date(Date.now() + 3600000) // 1 hour from now
    }
  });

  return result;
}

// Execute pipeline
const output = await executeFullPipeline('AI content automation');
console.log('Article:', output.article.url);
console.log('Video:', output.video.url);
console.log('Posted to:', output.published.platforms);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/news-crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  const results = await crawlNews({
    keyword,
    sources,
    timeframe
  });
  
  return NextResponse.json(results);
}
```

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateArticle } from '@/lib/generators/article-generator';

export async function POST(request: NextRequest) {
  const { topic, format, language, tone } = await request.json();
  
  const article = await generateArticle({
    topic,
    format,
    language,
    tone,
    aiProvider: 'claude'
  });
  
  return NextResponse.json(article);
}
```

### Video Render Endpoint

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/remotion/video-renderer';

export async function POST(request: NextRequest) {
  const { articleId, videoType } = await request.json();
  
  const video = await renderVideo({
    articleId,
    type: videoType,
    aspectRatio: '9:16'
  });
  
  return NextResponse.json({ videoUrl: video.url });
}
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (if separate)
npm run remotion:render

# Run content pipeline (custom script)
npm run pipeline -- --keyword "AI marketing" --format toplist

# Crawl news
npm run crawl -- --source techcrunch --keyword "AI"
```

## Common Workflows

### Workflow 1: Daily Automated Content

```typescript
import cron from 'node-cron';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI trends', 'Marketing automation', 'Content creation'];
  
  for (const topic of topics) {
    await runContentPipeline({
      research: { keyword: topic, timeframe: '24h' },
      generation: { format: 'toplist', languages: ['en', 'vi'] },
      video: { enabled: true, type: 'reels' },
      publish: { platforms: ['facebook'], schedule: null }
    });
  }
});
```

### Workflow 2: Batch Video Generation

```typescript
async function batchGenerateVideos(articleIds: string[]) {
  const videos = await Promise.all(
    articleIds.map(id => 
      renderVideo({
        articleId: id,
        type: 'shorts',
        aspectRatio: '9:16',
        quality: 'high'
      })
    )
  );
  
  return videos;
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000 // 1 minute
});

async function callAI(prompt: string) {
  await limiter.waitForSlot();
  return await aiProvider.generate(prompt);
}
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
const video = await renderVideo({
  composition,
  timeout: 300000, // 5 minutes
  concurrency: 4
});
```

### Content Quality Issues

```typescript
// Add validation before publishing
import { validateContent } from '@/lib/validators/content-validator';

const isValid = await validateContent(article, {
  minWordCount: 1000,
  requireSources: true,
  checkGrammar: true,
  checkPlagiarism: false
});

if (!isValid.passed) {
  console.error('Validation failed:', isValid.errors);
  // Regenerate or manual review
}
```

### Multi-language Sync Issues

```typescript
// Ensure consistent metadata across languages
async function syncMetadata(contents: Record<string, any>) {
  const baseMetadata = {
    publishDate: new Date(),
    author: 'AI Pipeline',
    tags: contents.en.tags // Use English as base
  };
  
  return Object.keys(contents).reduce((acc, lang) => {
    acc[lang] = { ...contents[lang], ...baseMetadata };
    return acc;
  }, {} as Record<string, any>);
}
```

This skill covers the complete content automation pipeline from research to video generation and publishing. All code examples use environment variables for sensitive data and follow TypeScript best practices.
