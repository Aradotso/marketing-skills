---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate video content from text automatically
  - build marketing content pipeline with AI
  - crawl news and create social media posts
  - set up automated content workflow
  - create AI-powered content generation system
  - automate research to video pipeline
  - build content automation with Claude and OpenAI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline crawls live data from sources like TechCrunch, Twitter/X, and LinkedIn, generates content using Claude 3 or OpenAI, and renders videos using Remotion.

## What This Project Does

The Marketing Pipeline system provides:

1. **Auto-Research**: Crawls and analyzes recent news from major tech/business sources
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
3. **Multi-language Support**: Generates Vietnamese and English versions simultaneously
4. **Video Rendering**: Automatically converts content to video using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ and npm/yarn required
node --version  # Should be 18.x or higher
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following required variables:

```bash
# AI Provider Keys (choose one or both)
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# News/Research API
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using persistent storage)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Core API & Usage Patterns

### 1. Research/Crawling Module

**Auto-crawl recent news sources:**

```typescript
import { crawlNewsData } from '@/lib/research/crawler';
import { analyzeInsights } from '@/lib/research/analyzer';

async function runResearch(keyword: string) {
  // Crawl data from configured sources
  const rawData = await crawlNewsData({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h'
  });
  
  // Extract insights using AI
  const insights = await analyzeInsights(rawData, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });
  
  return insights;
}
```

### 2. Content Generation Module

**Generate content in multiple formats:**

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, VoiceTone } from '@/types';

async function createContent(topic: string) {
  const content = await generateContent({
    topic,
    format: ContentFormat.TOPLIST, // or POV, CASE_STUDY, HOW_TO
    languages: ['en', 'vi'],
    voiceTone: VoiceTone.EXPERT, // or FRIENDLY, HUMOROUS
    provider: 'claude',
    researchData: insights, // from previous step
    includeStats: true
  });
  
  return content;
}
```

### 3. Video Rendering with Remotion

**Render video from generated content:**

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoFormat } from '@/types';

async function generateVideo(content: any) {
  const video = await renderVideo({
    content,
    format: VideoFormat.REELS, // or TIKTOK, SHORTS
    aspectRatio: '9:16',
    duration: 60, // seconds
    template: 'modern-infographic',
    outputPath: './output/videos'
  });
  
  return video.url;
}
```

### 4. Complete Pipeline Workflow

**End-to-end automation:**

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    enableVideoGeneration: true,
    autoPublish: false
  });
  
  // Execute complete workflow
  const result = await pipeline.execute({
    keyword,
    contentFormat: 'toplist',
    targetPlatforms: ['facebook', 'tiktok', 'youtube'],
    schedule: {
      autoPost: false,
      draftOnly: true
    }
  });
  
  return {
    content: result.generatedContent,
    videos: result.renderedVideos,
    insights: result.researchInsights
  };
}
```

## API Routes (Next.js)

### POST /api/research

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { crawlNewsData } from '@/lib/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, sources, timeRange } = req.body;
  
  try {
    const data = await crawlNewsData({ keyword, sources, timeRange });
    res.status(200).json({ success: true, data });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### POST /api/generate-content

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/content/generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { topic, format, languages, voiceTone } = req.body;
  
  try {
    const content = await generateContent({
      topic,
      format,
      languages,
      voiceTone,
      provider: process.env.AI_PROVIDER || 'claude'
    });
    
    res.status(200).json({ success: true, content });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### POST /api/render-video

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { content, format, aspectRatio } = req.body;
  
  try {
    const video = await renderVideo({
      content,
      format,
      aspectRatio,
      outputPath: './public/videos'
    });
    
    res.status(200).json({ success: true, videoUrl: video.url });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Configuration Files

### Content Generation Config

```typescript
// config/content.config.ts
export const contentConfig = {
  formats: {
    toplist: {
      minItems: 5,
      maxItems: 10,
      includeIntro: true,
      includeConclusion: true
    },
    pov: {
      perspective: 'first-person',
      includePersonalStory: true
    },
    caseStudy: {
      includeMetrics: true,
      includeTakeaways: true
    },
    howTo: {
      stepByStep: true,
      includeVisuals: true
    }
  },
  
  voiceTones: {
    expert: 'professional, authoritative, data-driven',
    friendly: 'conversational, approachable, warm',
    humorous: 'witty, entertaining, casual'
  },
  
  languages: ['en', 'vi'],
  
  aiProviders: {
    claude: {
      model: 'claude-3-opus-20240229',
      maxTokens: 4000
    },
    openai: {
      model: 'gpt-4-turbo-preview',
      maxTokens: 4000
    }
  }
};
```

### Video Rendering Config

```typescript
// config/video.config.ts
export const videoConfig = {
  templates: {
    'modern-infographic': {
      backgroundColor: '#1a1a2e',
      primaryColor: '#16213e',
      accentColor: '#0f3460',
      font: 'Inter'
    },
    'minimal': {
      backgroundColor: '#ffffff',
      primaryColor: '#000000',
      accentColor: '#666666',
      font: 'Helvetica'
    }
  },
  
  formats: {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 },
    landscape: { width: 1920, height: 1080, fps: 30 }
  },
  
  defaultDuration: 60,
  outputFormat: 'mp4'
};
```

## Common Patterns

### Pattern 1: Research + Content Generation

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function createToplistPost(topic: string) {
  const pipeline = new ContentPipeline();
  
  // Step 1: Research
  const insights = await pipeline.research({
    keyword: topic,
    depth: 'comprehensive'
  });
  
  // Step 2: Generate content
  const content = await pipeline.generateContent({
    researchData: insights,
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'expert'
  });
  
  // Step 3: Save draft
  await pipeline.saveDraft(content);
  
  return content;
}
```

### Pattern 2: Batch Processing

```typescript
async function processBatchKeywords(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runFullPipeline(keyword);
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Pattern 3: Multi-Platform Video Export

```typescript
import { renderVideo } from '@/lib/video/renderer';

async function exportToAllPlatforms(content: any) {
  const platforms = ['reels', 'tiktok', 'shorts', 'landscape'];
  const videos = {};
  
  for (const platform of platforms) {
    videos[platform] = await renderVideo({
      content,
      format: platform,
      outputPath: `./output/${platform}`
    });
  }
  
  return videos;
}
```

### Pattern 4: Scheduled Content Generation

```typescript
import cron from 'node-cron';
import { ContentPipeline } from '@/lib/pipeline';

// Run daily at 8 AM
cron.schedule('0 8 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics.slice(0, 3)) {
    await runFullPipeline(topic);
  }
});
```

## CLI Commands (if applicable)

```bash
# Generate content from keyword
npm run generate -- --keyword "AI trends 2024" --format toplist

# Render video from existing content
npm run render-video -- --input ./content/post-123.json --format reels

# Run research only
npm run research -- --keyword "startup funding" --sources all

# Export to multiple platforms
npm run export -- --content-id 123 --platforms reels,tiktok,shorts
```

## Troubleshooting

### Issue: API Rate Limiting

```typescript
// Implement retry logic with exponential backoff
import { retryWithBackoff } from '@/lib/utils/retry';

async function generateWithRetry(params: any) {
  return await retryWithBackoff(
    () => generateContent(params),
    {
      maxRetries: 3,
      baseDelay: 1000,
      maxDelay: 10000
    }
  );
}
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for Remotion rendering
import { renderVideo } from '@/lib/video/renderer';

const video = await renderVideo({
  content,
  format: 'reels',
  timeout: 300000, // 5 minutes
  concurrency: 1 // Reduce concurrency for stability
});
```

### Issue: Missing Research Data

```typescript
// Add fallback data sources
async function robustResearch(keyword: string) {
  try {
    return await crawlNewsData({ keyword, sources: ['techcrunch', 'twitter'] });
  } catch (error) {
    console.warn('Primary sources failed, using fallback');
    return await crawlNewsData({ keyword, sources: ['rss-feeds'] });
  }
}
```

### Issue: AI Provider Errors

```typescript
// Implement provider fallback
async function generateWithFallback(params: any) {
  try {
    return await generateContent({ ...params, provider: 'claude' });
  } catch (error) {
    console.warn('Claude failed, falling back to OpenAI');
    return await generateContent({ ...params, provider: 'openai' });
  }
}
```

## Testing

```typescript
// Example test for content generation
import { generateContent } from '@/lib/content/generator';

describe('Content Generation', () => {
  it('should generate toplist content', async () => {
    const content = await generateContent({
      topic: 'AI tools for marketing',
      format: 'toplist',
      languages: ['en'],
      voiceTone: 'expert',
      provider: 'claude'
    });
    
    expect(content).toHaveProperty('title');
    expect(content).toHaveProperty('body');
    expect(content.items).toHaveLength.greaterThan(5);
  });
});
```

## Advanced Usage

### Custom AI Prompts

```typescript
import { generateContent } from '@/lib/content/generator';

const customContent = await generateContent({
  topic: 'SaaS growth strategies',
  format: 'custom',
  customPrompt: `
    Create a detailed analysis of SaaS growth strategies with:
    - 3 case studies from successful companies
    - Data-backed metrics and KPIs
    - Actionable takeaways for each strategy
    - Write in a professional, authoritative tone
  `,
  provider: 'claude'
});
```

### Integration with CMS

```typescript
import { ContentPipeline } from '@/lib/pipeline';
import { publishToWordPress } from '@/integrations/wordpress';

async function generateAndPublish(keyword: string) {
  const pipeline = new ContentPipeline();
  const result = await pipeline.execute({ keyword });
  
  // Publish to WordPress
  await publishToWordPress({
    title: result.generatedContent.title,
    content: result.generatedContent.body,
    featuredImage: result.generatedContent.thumbnail,
    status: 'draft'
  });
}
```

This skill provides AI agents with comprehensive knowledge to help developers implement automated content marketing pipelines using this TypeScript-based system.
