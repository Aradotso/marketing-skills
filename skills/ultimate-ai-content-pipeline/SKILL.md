---
name: ultimate-ai-content-pipeline
description: Automated AI-powered content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content pipeline"
  - "generate AI-powered marketing content"
  - "create video content from text automatically"
  - "scrape and research content with AI"
  - "build automated content workflow"
  - "generate multilingual content with Claude"
  - "render videos from blog posts"
  - "automate social media content creation"
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a complete content automation system built with TypeScript and Next.js that transforms a single keyword into fully-realized content including research, written articles, and rendered videos. It combines web scraping, AI content generation (Claude 3, OpenAI), and video rendering (Remotion) into one unified workflow.

**Key capabilities:**
- Auto-scrape research from TechCrunch, a16z, Twitter, LinkedIn
- Generate multi-format content (toplist, POV, case study, how-to)
- Multilingual support (English/Vietnamese with customizable tone)
- Automatic video/infographic rendering for social platforms
- Next.js interface for managing content pipeline

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

Create a `.env.local` file in the project root:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database for storing content
DATABASE_URL=your_database_connection_string

# Remotion configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core Modules

### 1. Research & Scraping

The scraper module fetches fresh content from multiple sources:

```typescript
import { ResearchScraper } from '@/lib/scraper';

async function gatherResearch(keyword: string) {
  const scraper = new ResearchScraper({
    apiKey: process.env.RAPIDAPI_KEY,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
  });

  const research = await scraper.scrape({
    keyword,
    timeframe: '24h',
    maxResults: 20
  });

  return research; // { articles: [], insights: [], stats: [] }
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { ContentGenerator } from '@/lib/ai/generator';
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Using Claude
const claude = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const generator = new ContentGenerator({ 
  provider: 'claude',
  client: claude 
});

async function generateContent(research: Research, format: ContentFormat) {
  const content = await generator.generate({
    research,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'en', // or 'vi'
    tone: 'professional', // 'friendly' | 'humorous' | 'expert'
    length: 'medium' // 'short' | 'medium' | 'long'
  });

  return content;
}
```

**Multi-language generation:**

```typescript
async function generateBilingual(research: Research) {
  const englishContent = await generator.generate({
    research,
    format: 'toplist',
    language: 'en',
    tone: 'professional'
  });

  const vietnameseContent = await generator.generate({
    research,
    format: 'toplist', 
    language: 'vi',
    tone: 'friendly'
  });

  return { en: englishContent, vi: vietnameseContent };
}
```

### 3. Video Rendering with Remotion

Transform content into video format:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: GeneratedContent) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      stats: content.statistics,
      branding: {
        logo: '/logo.png',
        colors: { primary: '#3B82F6', accent: '#10B981' }
      }
    }
  });

  // Render video
  const outputPath = path.resolve(`./output/${content.id}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps
  });

  return outputPath;
}
```

### 4. Complete Pipeline

End-to-end workflow:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  // 1. Research phase
  console.log('🔍 Gathering research...');
  const research = await pipeline.research(keyword);

  // 2. Content generation
  console.log('✍️ Generating content...');
  const content = await pipeline.generateContent({
    research,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi']
  });

  // 3. Video rendering
  console.log('🎬 Rendering videos...');
  const videos = await pipeline.renderVideos(content, {
    platforms: ['reels', 'tiktok', 'shorts'],
    aspectRatios: ['9:16', '1:1']
  });

  // 4. Export results
  return {
    content,
    videos,
    metadata: {
      keyword,
      generatedAt: new Date(),
      totalVideos: videos.length
    }
  };
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone } = await request.json();

    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY,
      openaiKey: process.env.OPENAI_API_KEY,
      rapidApiKey: process.env.RAPIDAPI_KEY
    });

    const research = await pipeline.research(keyword);
    const content = await pipeline.generateContent({
      research,
      formats: [format],
      languages: [language],
      tone
    });

    return NextResponse.json({ 
      success: true, 
      content 
    });

  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Render Video Endpoint

```typescript
// src/app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { contentId, platform, aspectRatio } = await request.json();

    // Fetch content from database
    const content = await getContentById(contentId);

    const videoPath = await renderContentVideo(content, {
      platform,
      aspectRatio
    });

    return NextResponse.json({ 
      success: true, 
      videoUrl: `/videos/${path.basename(videoPath)}`
    });

  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Custom Content Templates

```typescript
interface ContentTemplate {
  format: string;
  structure: string[];
  minWords: number;
  maxWords: number;
}

const templates: Record<string, ContentTemplate> = {
  toplist: {
    format: 'toplist',
    structure: ['intro', 'items', 'conclusion'],
    minWords: 800,
    maxWords: 1500
  },
  caseStudy: {
    format: 'case-study',
    structure: ['problem', 'solution', 'results', 'lessons'],
    minWords: 1200,
    maxWords: 2000
  }
};

async function generateFromTemplate(
  research: Research, 
  templateName: string
) {
  const template = templates[templateName];
  
  return await generator.generate({
    research,
    format: template.format,
    instructions: `Follow this structure: ${template.structure.join(' → ')}. 
                   Target length: ${template.minWords}-${template.maxWords} words.`
  });
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      try {
        return await runContentPipeline(keyword);
      } catch (error) {
        console.error(`Failed for keyword: ${keyword}`, error);
        throw error;
      }
    })
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);

  const failed = results
    .filter(r => r.status === 'rejected')
    .map((r, idx) => ({ keyword: keywords[idx], error: r.reason }));

  return { successful, failed };
}
```

### Content Scheduling

```typescript
import { scheduledJobs } from 'node-schedule';

function scheduleContentGeneration(
  keyword: string, 
  cronExpression: string
) {
  const job = scheduledJobs.scheduleJob(cronExpression, async () => {
    console.log(`Running scheduled generation for: ${keyword}`);
    
    const result = await runContentPipeline(keyword);
    
    // Auto-post to social media or save to CMS
    await publishContent(result);
  });

  return job;
}

// Generate daily at 9 AM
scheduleContentGeneration('AI trends', '0 9 * * *');
```

## Running the Application

### Development Mode

```bash
npm run dev
# Server starts at http://localhost:3000
```

### Production Build

```bash
npm run build
npm start
```

### Render Videos Only

```bash
# If you have a separate script for video rendering
npm run render -- --content-id=123 --platform=tiktok
```

## Troubleshooting

### API Rate Limits

```typescript
import pRetry from 'p-retry';

async function generateWithRetry(params: GenerateParams) {
  return pRetry(
    async () => {
      return await generator.generate(params);
    },
    {
      retries: 3,
      onFailedAttempt: (error) => {
        console.log(
          `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
        );
      }
    }
  );
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos sequentially instead of parallel
async function renderVideosSequentially(contents: Content[]) {
  const results = [];
  
  for (const content of contents) {
    const video = await renderContentVideo(content);
    results.push(video);
    
    // Force garbage collection if available
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Scraping Errors

```typescript
async function robustScraping(keyword: string) {
  const scraper = new ResearchScraper({
    apiKey: process.env.RAPIDAPI_KEY,
    timeout: 30000,
    retries: 2
  });

  try {
    return await scraper.scrape({ keyword });
  } catch (error) {
    // Fallback to cached data or alternative sources
    console.warn('Primary scraping failed, using fallback');
    return await getCachedResearch(keyword);
  }
}
```

## Best Practices

1. **Always validate research data** before passing to AI generators
2. **Cache research results** to avoid redundant API calls
3. **Use streaming** for long-form content generation to improve UX
4. **Optimize video rendering** by processing in background jobs
5. **Monitor API costs** by tracking token usage across Claude/OpenAI calls
6. **Version your prompts** to maintain consistent output quality

## Resources

- Check `HUONG_DAN_CAI_DAT.md` for detailed Vietnamese installation guide
- Review `/src/lib/ai/prompts.ts` for prompt engineering examples
- Explore `/remotion` folder for video template customization
