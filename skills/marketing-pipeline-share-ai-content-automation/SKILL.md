---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, auto-posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for auto content posting
  - generate videos from AI-written articles automatically
  - create AI content pipeline with Claude and Remotion
  - build automated marketing content system
  - integrate AI content research with video rendering
  - configure auto-posting content pipeline
  - set up end-to-end AI content automation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an all-in-one AI content automation system that handles the entire content lifecycle: from researching trending topics, generating multi-format articles in multiple languages, to automatically rendering videos and scheduling posts. Built with Next.js, TypeScript, Claude/OpenAI APIs, and Remotion for video generation.

## What It Does

The Marketing Pipeline Share automates:

1. **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in various formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language Output**: Generates content simultaneously in English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically converts written content into infographics and short-form videos using Remotion
5. **Auto-Publishing**: Schedules and posts content to multiple platforms

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

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_API_KEY=your_twitter_api_key_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key

# Publishing Platforms
FACEBOOK_PAGE_TOKEN=your_facebook_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run render
```

Access the application at `http://localhost:3000`

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { researchTopic } from '@/lib/research';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 50
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    dataPoints: research.statistics
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: string, research: any) {
  const message = await anthropic.messages.create({
    model: "claude-3-5-sonnet-20241022",
    max_tokens: 4096,
    messages: [{
      role: "user",
      content: `Create a ${format} format article about "${topic}".
      
Research data:
${JSON.stringify(research, null, 2)}

Requirements:
- Write in both English and Vietnamese
- Include data-backed insights
- Use engaging storytelling
- Optimize for SEO`
    }]
  });
  
  return message.content;
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: "gpt-4-turbo-preview",
    messages: [
      {
        role: "system",
        content: `You are an expert content creator with a ${tone} tone. Create engaging, data-backed content.`
      },
      {
        role: "user",
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: any) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/video/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      stats: content.statistics,
      branding: content.brandAssets
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
    inputProps: composition.props,
  });
  
  return `out/${content.slug}.mp4`;
}
```

## Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/research';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { publishToSocial } from '@/lib/publishing/social';

async function runContentPipeline(keyword: string, options: PipelineOptions) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: options.sources || ['techcrunch', 'a16z'],
      timeframe: '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      topic: keyword,
      format: options.format || 'toplist',
      tone: options.tone || 'professional',
      languages: ['en', 'vi'],
      research: research.insights,
      aiProvider: options.aiProvider || 'claude'
    });

    // Step 3: Render video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      content: content.en,
      style: 'infographic',
      duration: 60,
      aspectRatio: '9:16', // For Reels/TikTok/Shorts
      branding: {
        logo: options.logoPath,
        colors: options.brandColors
      }
    });

    // Step 4: Publish
    console.log('📤 Publishing...');
    const published = await publishToSocial({
      platforms: ['facebook', 'linkedin', 'twitter'],
      content: {
        text: content.en.excerpt,
        media: videoPath,
        hashtags: content.en.hashtags
      },
      schedule: options.scheduleTime
    });

    return {
      success: true,
      content,
      videoPath,
      published
    };

  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runContentPipeline('AI Marketing Automation', {
  format: 'case-study',
  tone: 'expert',
  sources: ['techcrunch', 'a16z', 'twitter'],
  aiProvider: 'claude',
  scheduleTime: new Date('2024-12-25T10:00:00'),
  brandColors: ['#3B82F6', '#8B5CF6']
});
```

## Configuration Patterns

### Content Format Types

```typescript
type ContentFormat = 
  | 'toplist'      // "Top 10 Ways to..."
  | 'pov'          // Personal perspective/opinion
  | 'case-study'   // In-depth analysis
  | 'how-to'       // Step-by-step guide
  | 'news'         // Breaking news format
  | 'comparison';  // A vs B analysis

interface ContentConfig {
  format: ContentFormat;
  tone: 'professional' | 'friendly' | 'humorous' | 'expert';
  length: 'short' | 'medium' | 'long';
  includeStats: boolean;
  includeCTA: boolean;
  seoOptimized: boolean;
}
```

### Video Rendering Configurations

```typescript
interface VideoConfig {
  aspectRatio: '16:9' | '9:16' | '1:1' | '4:5';
  duration: number; // seconds
  fps: 30 | 60;
  style: 'minimal' | 'infographic' | 'dynamic' | 'corporate';
  transitions: boolean;
  audio?: {
    background?: string;
    voiceover?: boolean;
  };
}
```

### Multi-Platform Publishing

```typescript
interface PublishConfig {
  platforms: Array<'facebook' | 'linkedin' | 'twitter' | 'instagram'>;
  schedule?: Date;
  crossPost: boolean;
  platformSpecific?: {
    facebook?: { pageId: string };
    linkedin?: { companyId?: string };
    twitter?: { threadMode: boolean };
  };
}
```

## Common Workflows

### Daily Trending Content

```typescript
import { schedule } from 'node-cron';

// Run every day at 8 AM
schedule('0 8 * * *', async () => {
  const trendingTopics = await researchTopic({
    keyword: 'AI trends',
    sources: ['twitter', 'techcrunch'],
    timeframe: '24h',
    trending: true
  });

  for (const topic of trendingTopics.top5) {
    await runContentPipeline(topic.keyword, {
      format: 'pov',
      tone: 'expert',
      aiProvider: 'claude'
    });
  }
});
```

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runContentPipeline(keyword, {
        format: 'toplist',
        tone: 'friendly',
        scheduleTime: getNextAvailableSlot()
      })
    )
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  return { successful, failed };
}
```

## Troubleshooting

### API Rate Limits

```typescript
import pLimit from 'p-limit';

// Limit concurrent API calls
const limit = pLimit(3);

const results = await Promise.all(
  topics.map(topic => 
    limit(() => generateContent(topic, 'toplist', research))
  )
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  }
});
```

### Memory Issues with Large Research Data

```typescript
// Stream and process research data in chunks
async function processLargeResearch(keyword: string) {
  const chunkSize = 10;
  const allResults = [];
  
  for (let i = 0; i < totalSources; i += chunkSize) {
    const chunk = sources.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(source => crawlSource(source, keyword))
    );
    allResults.push(...chunkResults);
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }
  
  return allResults;
}
```

### Claude/OpenAI API Errors

```typescript
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(prompt);
    } catch (error) {
      if (error.status === 429) {
        // Rate limit - wait and retry
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      if (error.status === 500 && i < maxRetries - 1) {
        // Server error - retry
        continue;
      }
      throw error;
    }
  }
}
```

## Additional Resources

- See `HUONG_DAN_CAI_DAT.md` in the project root for detailed setup instructions in Vietnamese
- Check `/src/lib` for core library implementations
- Video templates are in `/src/video`
- API routes in `/src/pages/api` or `/src/app/api` depending on Next.js version
