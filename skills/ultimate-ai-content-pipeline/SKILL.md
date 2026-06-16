---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - "create automated content pipeline"
  - "generate AI video from article"
  - "scrape and research content automatically"
  - "build marketing content automation"
  - "create multilingual blog posts with AI"
  - "automate content from research to video"
  - "set up AI content generation workflow"
  - "generate social media videos automatically"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive content automation system that transforms a single keyword into complete content packages including research, written articles, and rendered videos. The system integrates Claude 3, OpenAI, web scraping APIs, and Remotion video rendering to create an end-to-end content production workflow.

**Key capabilities:**
- Auto-scrape recent news from TechCrunch, a16z, Twitter, LinkedIn
- Generate multi-format content (toplist, POV, case study, how-to)
- Bilingual support (English & Vietnamese)
- Automatic video rendering with Remotion
- Next.js web interface for content management

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the required API keys:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── scraper/     # Web scraping modules
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Usage Patterns

### 1. Content Research & Scraping

```typescript
import { researchTopic } from '@/lib/scraper/research';
import { analyzeSources } from '@/lib/ai/analyzer';

async function gatherResearch(keyword: string) {
  // Scrape recent news from multiple sources
  const sources = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // AI-powered analysis of scraped content
  const insights = await analyzeSources(sources, {
    extractStats: true,
    findTrends: true,
    language: 'en'
  });

  return {
    rawData: sources,
    insights,
    timestamp: new Date()
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(topic: string, research: any) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    prompt: {
      topic,
      format: 'case-study', // toplist, pov, how-to
      tone: 'professional', // friendly, humorous
      research: research.insights
    },
    languages: ['en', 'vi'],
    includeMetadata: true
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.metadata,
    seo: content.seo
  };
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { createVideoScript } from '@/lib/ai/video-script';

async function generateVideoFromContent(article: any) {
  // Create video script from article
  const script = await createVideoScript({
    content: article.english,
    duration: 60, // seconds
    platform: 'instagram-reels', // tiktok, youtube-shorts
    style: 'infographic'
  });

  // Render video with Remotion
  const video = await renderVideo({
    composition: 'ContentVideo',
    props: {
      script: script.scenes,
      title: article.metadata.title,
      branding: {
        logo: '/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    },
    output: {
      format: 'mp4',
      fps: 30,
      resolution: [1080, 1920] // vertical for social
    }
  });

  return {
    videoUrl: video.url,
    duration: video.duration,
    thumbnail: video.thumbnail
  };
}
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: process.env.ANTHROPIC_API_KEY ? 'claude' : 'openai',
    videoEnabled: true,
    autoPublish: false
  });

  try {
    // Step 1: Research
    const research = await pipeline.research(keyword);
    console.log(`Found ${research.sources.length} sources`);

    // Step 2: Generate content in multiple formats
    const contents = await pipeline.generateMultiFormat({
      keyword,
      research,
      formats: ['toplist', 'case-study', 'how-to'],
      languages: ['en', 'vi']
    });

    // Step 3: Render videos for each format
    const videos = await Promise.all(
      contents.map(content => 
        pipeline.renderVideo(content, {
          platforms: ['instagram', 'tiktok', 'youtube']
        })
      )
    );

    // Step 4: Schedule for publishing
    const scheduled = await pipeline.schedule({
      contents,
      videos,
      publishAt: new Date(Date.now() + 24 * 60 * 60 * 1000)
    });

    return {
      research,
      contents,
      videos,
      scheduled
    };

  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runFullPipeline('AI in marketing automation 2024')
  .then(result => console.log('Pipeline complete:', result))
  .catch(console.error);
```

## API Routes (Next.js)

### Create Content via API

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(req: NextRequest) {
  const { keyword, format, language } = await req.json();

  try {
    const content = await generateContent({
      provider: 'claude',
      prompt: {
        topic: keyword,
        format,
        tone: 'professional'
      },
      languages: [language]
    });

    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Trigger Video Rendering

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { contentId, platform } = await req.json();

  try {
    const video = await renderVideo({
      composition: 'ContentVideo',
      props: { contentId, platform },
      output: { format: 'mp4' }
    });

    return NextResponse.json({ 
      success: true, 
      videoUrl: video.url,
      thumbnail: video.thumbnail
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## CLI Commands (if applicable)

```bash
# Generate content from keyword
npm run generate -- --keyword "AI trends 2024" --format toplist

# Run research only
npm run research -- --keyword "marketing automation" --sources techcrunch,a16z

# Render video from existing content
npm run render -- --content-id 123 --platform instagram

# Run full pipeline
npm run pipeline -- --keyword "content marketing" --auto-publish
```

## Advanced Patterns

### Custom Content Templates

```typescript
import { ContentTemplate } from '@/lib/types/templates';

const customTemplate: ContentTemplate = {
  name: 'thought-leadership',
  structure: {
    hook: { type: 'question', minWords: 30 },
    intro: { type: 'story', minWords: 100 },
    body: {
      sections: 3,
      eachWithExample: true,
      includeStats: true
    },
    conclusion: { type: 'call-to-action', minWords: 50 }
  },
  tone: 'authoritative',
  seoOptimized: true
};

const article = await generateContent({
  template: customTemplate,
  research: researchData
});
```

### Multi-Platform Video Variants

```typescript
const platforms = [
  { name: 'instagram', aspect: [9, 16], duration: 60 },
  { name: 'tiktok', aspect: [9, 16], duration: 60 },
  { name: 'youtube', aspect: [16, 9], duration: 120 },
  { name: 'linkedin', aspect: [1, 1], duration: 90 }
];

const videoVariants = await Promise.all(
  platforms.map(platform => 
    renderVideo({
      composition: 'AdaptiveVideo',
      props: { content, platform: platform.name },
      output: {
        resolution: calculateResolution(platform.aspect),
        duration: platform.duration
      }
    })
  )
);
```

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const batchSize = 3; // Process 3 at a time to avoid rate limits
  const results = [];

  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    
    const batchResults = await Promise.all(
      batch.map(async keyword => {
        const research = await researchTopic(keyword);
        const content = await generateContent({ 
          prompt: { topic: keyword },
          research 
        });
        return { keyword, content };
      })
    );

    results.push(...batchResults);
    
    // Rate limiting delay
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

## Troubleshooting

### AI Provider Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMinutes: 1
});

async function safeAICall(prompt: any) {
  await limiter.waitForSlot();
  
  try {
    return await generateContent(prompt);
  } catch (error) {
    if (error.message.includes('rate_limit')) {
      console.log('Rate limited, waiting 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeAICall(prompt);
    }
    throw error;
  }
}
```

### Video Rendering Memory Issues

```typescript
// Reduce resolution for development
const isDev = process.env.NODE_ENV === 'development';

const videoConfig = {
  resolution: isDev ? [720, 1280] : [1080, 1920],
  quality: isDev ? 'medium' : 'high',
  concurrency: isDev ? 1 : 4
};
```

### Scraping Failures

```typescript
import { retry } from '@/lib/utils/retry';

const scrapedData = await retry(
  () => researchTopic(keyword),
  {
    maxAttempts: 3,
    backoff: 'exponential',
    onRetry: (attempt, error) => {
      console.log(`Retry attempt ${attempt}: ${error.message}`);
    }
  }
);
```

## Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run type checking
npm run type-check

# Lint code
npm run lint
```

## Best Practices

1. **Always validate API keys** before running pipelines
2. **Use rate limiting** for AI providers to avoid quota exhaustion
3. **Cache research results** to minimize redundant scraping
4. **Test video rendering locally** before batch processing
5. **Monitor token usage** for cost optimization
6. **Store generated content** with metadata for version control
