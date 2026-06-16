---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing
triggers:
  - automate content creation pipeline
  - research and generate marketing content
  - create videos from text automatically
  - build ai content workflow
  - set up automated social media posts
  - generate multi-format content with ai
  - crawl news and create content
  - automate video rendering for marketing
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Automation is an end-to-end content creation system that automates the entire workflow from research to video generation. It crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses AI (Claude 3, OpenAI) to generate multi-format content (articles, scripts, social posts) and renders videos automatically using Remotion.

**Key capabilities:**
- Auto-scan and research trending topics from major news sources
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Multi-language support (English/Vietnamese)
- Automatic video/infographic rendering via Remotion
- Next.js interface for content management and scheduling

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

Create a `.env.local` file with the following required variables:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── scraper/     # Content scraping modules
│   │   └── video/       # Remotion video generation
│   ├── services/        # Business logic services
│   └── types/           # TypeScript types
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Content Research & Scraping

```typescript
import { researchTopic } from '@/lib/scraper/research';

async function gatherInsights(topic: string) {
  const research = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  return {
    insights: research.insights,
    statistics: research.statistics,
    trendingTopics: research.trending,
    sourceUrls: research.sources
  };
}

// Example usage
const aiInsights = await gatherInsights('artificial intelligence trends');
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(research: ResearchData) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    format: 'toplist', // or 'pov', 'case-study', 'how-to'
    language: 'en', // or 'vi' for Vietnamese
    tone: 'expert', // or 'friendly', 'humorous'
    research: research,
    targetAudience: 'marketers',
    wordCount: 1500
  });

  return {
    title: content.title,
    body: content.body,
    hashtags: content.hashtags,
    metaDescription: content.meta,
    imagePrompts: content.imagePrompts
  };
}

// Multi-language generation
async function createBilingualContent(research: ResearchData) {
  const [english, vietnamese] = await Promise.all([
    generateContent({ ...options, language: 'en' }),
    generateContent({ ...options, language: 'vi' })
  ]);

  return { english, vietnamese };
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoComposition } from '@/remotion/compositions';

async function generateMarketingVideo(content: GeneratedContent) {
  const videoConfig = {
    composition: 'social-media-short',
    props: {
      title: content.title,
      keyPoints: content.keyPoints,
      statistics: content.statistics,
      branding: {
        logo: '/assets/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    },
    // Platform-specific aspect ratios
    format: 'reels', // or 'tiktok', 'shorts', 'youtube'
    duration: 30 // seconds
  };

  const video = await renderVideo(videoConfig);

  return {
    url: video.url,
    duration: video.duration,
    format: video.format
  };
}

// Batch render for multiple platforms
async function renderForAllPlatforms(content: GeneratedContent) {
  const platforms = ['reels', 'tiktok', 'shorts', 'youtube'];
  
  const videos = await Promise.all(
    platforms.map(platform => 
      renderVideo({ ...config, format: platform })
    )
  );

  return videos;
}
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/services/pipeline';

async function runFullPipeline(topic: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    enableVideo: true,
    publishAutomatically: false
  });

  // Step 1: Research
  const research = await pipeline.research({
    topic,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    depth: 'comprehensive'
  });

  // Step 2: Generate content in multiple formats
  const content = await pipeline.generate({
    research,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi']
  });

  // Step 3: Create videos
  const videos = await pipeline.renderVideos({
    content: content.en.toplist,
    platforms: ['reels', 'tiktok']
  });

  // Step 4: Schedule or publish
  await pipeline.schedule({
    content,
    videos,
    platforms: {
      facebook: { pageId: process.env.FB_PAGE_ID },
      instagram: { accountId: process.env.IG_ACCOUNT_ID },
      tiktok: { accountId: process.env.TIKTOK_ACCOUNT_ID }
    },
    scheduledTime: new Date('2024-12-20T10:00:00')
  });

  return {
    research,
    content,
    videos,
    status: 'scheduled'
  };
}
```

### 5. Custom Content Formats

```typescript
import { AIContentGenerator } from '@/lib/ai/generator';

// Define custom format template
const customFormat = {
  name: 'viral-hook',
  structure: [
    { section: 'hook', maxWords: 50 },
    { section: 'problem', maxWords: 100 },
    { section: 'solution', maxWords: 200 },
    { section: 'cta', maxWords: 30 }
  ],
  tone: 'engaging',
  includeEmojis: true
};

async function generateViralPost(topic: string) {
  const generator = new AIContentGenerator({
    provider: 'openai',
    model: 'gpt-4-turbo'
  });

  const post = await generator.createCustomFormat({
    format: customFormat,
    topic,
    targetPlatform: 'linkedin',
    includeHashtags: true,
    maxHashtags: 5
  });

  return post;
}
```

### 6. API Routes (Next.js)

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/services/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { topic, formats, languages } = await request.json();

    const pipeline = new ContentPipeline({
      aiProvider: process.env.AI_PROVIDER || 'claude'
    });

    const research = await pipeline.research({ topic });
    const content = await pipeline.generate({ research, formats, languages });

    return NextResponse.json({
      success: true,
      data: content
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

```typescript
// app/api/video/render/route.ts
export async function POST(request: NextRequest) {
  const { contentId, platforms } = await request.json();

  const content = await getContentById(contentId);
  const videos = await renderForAllPlatforms(content);

  return NextResponse.json({
    success: true,
    videos
  });
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

# Render Remotion videos locally
npm run remotion:preview

# Type checking
npm run type-check

# Linting
npm run lint
```

## Common Workflows

### Workflow 1: Daily Trend Analysis & Content

```typescript
import { scheduleDaily } from '@/lib/scheduler';

scheduleDaily(async () => {
  const topics = await getTrendingTopics(['ai', 'marketing', 'tech']);
  
  for (const topic of topics) {
    const research = await researchTopic(topic);
    
    if (research.trendScore > 0.7) {
      const content = await generateContent({
        research,
        format: 'toplist',
        language: 'en'
      });

      await saveToDatabase(content);
      await notifyTeam(`New trending content: ${content.title}`);
    }
  }
}, '08:00'); // Run at 8 AM daily
```

### Workflow 2: Batch Video Creation

```typescript
async function batchCreateVideos(contentIds: string[]) {
  const results = [];

  for (const id of contentIds) {
    const content = await getContent(id);
    
    const videos = await Promise.all([
      renderVideo({ content, format: 'reels' }),
      renderVideo({ content, format: 'tiktok' }),
      renderVideo({ content, format: 'youtube' })
    ]);

    results.push({
      contentId: id,
      videos,
      status: 'completed'
    });
  }

  return results;
}
```

### Workflow 3: A/B Testing Content Variants

```typescript
async function createVariants(topic: string, variantCount: number = 3) {
  const research = await researchTopic(topic);
  
  const variants = await Promise.all(
    Array.from({ length: variantCount }).map((_, i) => 
      generateContent({
        research,
        format: 'pov',
        tone: ['expert', 'friendly', 'humorous'][i],
        temperature: 0.7 + (i * 0.1)
      })
    )
  );

  return variants.map((content, index) => ({
    variant: `v${index + 1}`,
    content,
    metrics: { impressions: 0, engagement: 0, conversions: 0 }
  }));
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  openai: { requestsPerMinute: 60 },
  anthropic: { requestsPerMinute: 50 },
  rapidapi: { requestsPerMinute: 100 }
});

await limiter.throttle('openai', async () => {
  return await generateContent({ provider: 'openai', ... });
});
```

### Video Rendering Timeouts

```typescript
// Increase timeout for long videos
const video = await renderVideo({
  ...config,
  timeout: 300000, // 5 minutes
  quality: 'medium' // Use lower quality for faster rendering
});
```

### Memory Issues with Large Datasets

```typescript
// Process research in chunks
async function researchInChunks(topics: string[]) {
  const chunkSize = 5;
  const results = [];

  for (let i = 0; i < topics.length; i += chunkSize) {
    const chunk = topics.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(topic => researchTopic(topic))
    );
    results.push(...chunkResults);
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }

  return results;
}
```

### Error Handling Best Practices

```typescript
import { logger } from '@/lib/utils/logger';

async function safeContentGeneration(research: ResearchData) {
  try {
    return await generateContent({ research });
  } catch (error) {
    logger.error('Content generation failed', {
      error: error.message,
      research: research.topic
    });

    // Fallback to simpler generation
    return await generateContent({
      research,
      format: 'simple',
      fallbackMode: true
    });
  }
}
```

## Advanced Configuration

### Custom AI Prompts

```typescript
// lib/ai/prompts.ts
export const customPrompts = {
  toplist: `
    Create a compelling top list article about {topic}.
    Use these insights: {insights}
    Include {itemCount} items.
    Tone: {tone}
    Language: {language}
  `,
  
  'case-study': `
    Write a detailed case study about {topic}.
    Research data: {research}
    Focus on ROI and actionable insights.
    Target audience: {audience}
  `
};
```

### Video Template Customization

```typescript
// remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000', opacity }}>
      <h1>{title}</h1>
      {points.map((point, i) => (
        <p key={i}>{point}</p>
      ))}
    </AbsoluteFill>
  );
};
```

This skill enables AI coding agents to effectively use the Marketing Pipeline Automation system for end-to-end content creation, from research to video generation and publishing.
