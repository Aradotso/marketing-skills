---
name: ai-content-pipeline-automation
description: Automated AI-powered content pipeline for research, scriptwriting, social posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content from research to video
  - build content automation system with Remotion rendering
  - integrate AI content research and auto-posting
  - generate videos automatically from written content
  - use AI pipeline for multi-format content creation
  - automate social media content with AI research
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**Ultimate AI Content Pipeline** is a comprehensive TypeScript-based automation system that transforms a single keyword into complete content packages including research, written content in multiple formats, and rendered videos. The system crawls real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn, then uses Claude 3 or OpenAI to generate content in various formats (top lists, POV pieces, case studies, how-tos) in both English and Vietnamese, and finally renders videos using Remotion.

Key capabilities:
- **Auto-research**: Crawls and analyzes recent news/data from major sources
- **Multi-format AI writing**: Generates content in multiple styles and languages
- **Video generation**: Converts written content to videos via Remotion
- **Social automation**: Can auto-post to Facebook pages
- **Next.js interface**: Web UI for managing the entire pipeline

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Clone and Install

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media (optional)
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token_here
FACEBOOK_PAGE_ID=your_page_id_here

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
REMOTION_BUCKET_NAME=your_s3_bucket_name

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

### Development Mode

```bash
npm run dev
# or
yarn dev
```

Access the web interface at `http://localhost:3000`

### Production Build

```bash
npm run build
npm start
# or
yarn build
yarn start
```

## Core Architecture

The pipeline consists of several key modules:

1. **Research Module** (`/src/lib/research`) - Data crawling and analysis
2. **Content Generation** (`/src/lib/ai`) - AI-powered writing
3. **Video Rendering** (`/src/lib/video`) - Remotion integration
4. **Scheduling** (`/src/lib/scheduler`) - Auto-posting logic
5. **API Routes** (`/src/pages/api`) - Backend endpoints

## Usage Patterns

### 1. Basic Content Generation Flow

```typescript
import { generateContentPipeline } from '@/lib/pipeline';
import { ContentFormat } from '@/types';

async function createContent(keyword: string) {
  const result = await generateContentPipeline({
    keyword: keyword,
    format: ContentFormat.TOPLIST,
    languages: ['en', 'vi'],
    includeVideo: true,
    aiProvider: 'claude', // or 'openai'
  });

  return {
    englishContent: result.content.en,
    vietnameseContent: result.content.vi,
    videoUrl: result.video?.url,
    metadata: result.metadata,
  };
}

// Usage
const content = await createContent('AI automation tools 2026');
```

### 2. Research-Only Mode

```typescript
import { researchTopic } from '@/lib/research';

async function performResearch(topic: string) {
  const research = await researchTopic({
    query: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20,
  });

  return {
    insights: research.insights,
    trendingTopics: research.trends,
    statistics: research.stats,
    sources: research.sourceLinks,
  };
}

// Usage
const insights = await performResearch('marketing automation');
console.log(insights.insights);
```

### 3. AI Content Writing with Custom Tone

```typescript
import { generateArticle } from '@/lib/ai/writer';
import { ToneType } from '@/types';

async function writeArticle(researchData: any) {
  const article = await generateArticle({
    researchData: researchData,
    format: 'how-to',
    tone: ToneType.FRIENDLY,
    language: 'vi',
    targetAudience: 'marketers',
    wordCount: 1500,
    provider: 'claude', // Uses Claude 3
  });

  return {
    title: article.title,
    content: article.body,
    metadata: article.meta,
    suggestedImages: article.images,
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderContentVideo } from '@/lib/video/renderer';
import { VideoStyle } from '@/types';

async function createVideo(content: string, title: string) {
  const video = await renderContentVideo({
    content: content,
    title: title,
    style: VideoStyle.INFOGRAPHIC,
    aspectRatio: '9:16', // For Reels/TikTok/Shorts
    duration: 60, // seconds
    voiceover: false,
    backgroundMusic: true,
  });

  return {
    videoUrl: video.url,
    thumbnailUrl: video.thumbnail,
    duration: video.duration,
    fileSize: video.size,
  };
}

// Usage
const video = await createVideo(
  'Your article content here...',
  'Top 5 AI Tools for Marketers'
);
```

### 5. Full Pipeline with Auto-Posting

```typescript
import { runFullPipeline } from '@/lib/pipeline';

async function automatedContentCreation() {
  const pipeline = await runFullPipeline({
    keyword: 'content marketing trends',
    research: {
      enabled: true,
      sources: ['techcrunch', 'twitter'],
      timeRange: '48h',
    },
    content: {
      formats: ['toplist', 'how-to'],
      languages: ['en', 'vi'],
      tone: 'expert',
      aiProvider: 'openai',
    },
    video: {
      enabled: true,
      style: 'infographic',
      platforms: ['reels', 'tiktok', 'shorts'],
    },
    publishing: {
      autoPost: true,
      platforms: ['facebook'],
      scheduleTime: new Date('2026-06-25T10:00:00Z'),
    },
  });

  return pipeline;
}
```

## API Endpoints

### POST `/api/content/generate`

Generate content from a keyword.

```typescript
// Request
fetch('/api/content/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI marketing tools',
    format: 'toplist',
    language: 'vi',
    includeResearch: true,
  }),
});

// Response
{
  "id": "content_123",
  "title": "Top 10 AI Marketing Tools 2026",
  "content": "...",
  "metadata": {
    "wordCount": 1500,
    "readTime": 6,
    "sources": []
  },
  "status": "completed"
}
```

### POST `/api/video/render`

Render video from content.

```typescript
fetch('/api/video/render', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    contentId: 'content_123',
    style: 'infographic',
    aspectRatio: '9:16',
  }),
});
```

### GET `/api/research/:topic`

Fetch research data for a topic.

```typescript
const research = await fetch('/api/research/AI%20automation');
const data = await research.json();
```

## Configuration Files

### Content Templates

Create custom templates in `/src/config/templates.ts`:

```typescript
export const contentTemplates = {
  toplist: {
    structure: [
      'introduction',
      'criteria',
      'items',
      'comparison',
      'conclusion',
    ],
    toneVariations: ['expert', 'friendly', 'casual'],
  },
  howto: {
    structure: [
      'problem',
      'solution_overview',
      'steps',
      'tips',
      'conclusion',
    ],
    includeImages: true,
  },
  caseStudy: {
    structure: [
      'background',
      'challenge',
      'solution',
      'results',
      'takeaways',
    ],
    requiresData: true,
  },
};
```

### Research Sources

Configure sources in `/src/config/sources.ts`:

```typescript
export const researchSources = {
  techcrunch: {
    enabled: true,
    endpoint: process.env.RAPIDAPI_TECHCRUNCH_ENDPOINT,
    apiKey: process.env.RAPIDAPI_KEY,
    priority: 1,
  },
  twitter: {
    enabled: true,
    hashtags: ['#marketing', '#AI', '#automation'],
    minEngagement: 100,
  },
  linkedin: {
    enabled: true,
    profiles: ['influencer1', 'influencer2'],
  },
};
```

## Common Workflows

### Workflow 1: Daily Automated Content

```typescript
import { scheduleContentGeneration } from '@/lib/scheduler';

// Run daily at 6 AM
scheduleContentGeneration({
  cron: '0 6 * * *',
  keywords: ['AI trends', 'marketing automation', 'content tools'],
  rotate: true,
  config: {
    research: true,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    video: true,
    autoPost: true,
  },
});
```

### Workflow 2: Batch Content Generation

```typescript
import { batchGenerate } from '@/lib/pipeline';

async function generateWeeklyContent() {
  const keywords = [
    'AI content creation',
    'Marketing automation 2026',
    'Video marketing tips',
    'Social media trends',
    'SEO strategies',
  ];

  const results = await batchGenerate({
    keywords: keywords,
    parallel: 2, // Process 2 at a time
    format: 'toplist',
    languages: ['en', 'vi'],
    includeVideo: true,
  });

  return results;
}
```

### Workflow 3: Multi-Platform Video Creation

```typescript
import { generateMultiPlatformVideos } from '@/lib/video';

async function createPlatformVideos(contentId: string) {
  const videos = await generateMultiPlatformVideos({
    contentId: contentId,
    platforms: [
      { name: 'reels', aspectRatio: '9:16', duration: 60 },
      { name: 'youtube', aspectRatio: '16:9', duration: 300 },
      { name: 'tiktok', aspectRatio: '9:16', duration: 45 },
    ],
    style: 'infographic',
    brandColors: ['#FF6B6B', '#4ECDC4'],
  });

  return videos;
}
```

## Troubleshooting

### Issue: AI Provider Rate Limits

```typescript
// Implement retry logic with exponential backoff
import { retryWithBackoff } from '@/lib/utils/retry';

const content = await retryWithBackoff(
  () => generateArticle(params),
  {
    maxRetries: 3,
    initialDelay: 1000,
    backoffMultiplier: 2,
  }
);
```

### Issue: Video Rendering Fails

```typescript
// Check Remotion configuration and AWS credentials
import { validateRemotionConfig } from '@/lib/video/validate';

try {
  await validateRemotionConfig();
} catch (error) {
  console.error('Remotion config error:', error.message);
  // Check: REMOTION_AWS_ACCESS_KEY_ID, REMOTION_AWS_SECRET_ACCESS_KEY
}
```

### Issue: Research Data Quality

```typescript
// Filter and validate research results
import { validateResearchQuality } from '@/lib/research/validator';

const research = await researchTopic(query);
const validatedData = validateResearchQuality(research, {
  minSources: 3,
  minRecency: '24h',
  requireStats: true,
});
```

### Issue: Content Language Detection

```typescript
// Ensure proper language handling
import { detectLanguage, translateContent } from '@/lib/ai/language';

const detectedLang = await detectLanguage(content);
if (detectedLang !== targetLanguage) {
  content = await translateContent(content, {
    from: detectedLang,
    to: targetLanguage,
    provider: 'openai',
  });
}
```

## Performance Optimization

### Caching Research Results

```typescript
import { cacheResearch } from '@/lib/cache';

const cachedResearch = await cacheResearch(
  topic,
  async () => await researchTopic(topic),
  { ttl: 3600 } // Cache for 1 hour
);
```

### Parallel Processing

```typescript
import { processInParallel } from '@/lib/utils';

const results = await processInParallel(
  keywords,
  async (keyword) => await generateContentPipeline({ keyword }),
  { concurrency: 3 }
);
```

## Best Practices

1. **Always validate API keys** before running the pipeline
2. **Use caching** for research data to avoid redundant API calls
3. **Implement rate limiting** when working with external APIs
4. **Monitor costs** for AI provider usage (Claude/OpenAI tokens)
5. **Test video rendering** locally before deploying to production
6. **Use environment-specific configs** for development vs. production
7. **Implement error tracking** for pipeline failures
8. **Schedule content generation** during off-peak hours to reduce costs
