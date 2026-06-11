---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research and video generation
  - set up the ultimate AI content pipeline for marketing automation
  - generate automated content from research to video with Claude and OpenAI
  - create content pipeline with automatic news scraping and video rendering
  - configure AI marketing pipeline for multi-language content generation
  - automate content workflow from keyword research to video production
  - integrate Claude and OpenAI for automated content creation pipeline
  - build automated marketing content system with Remotion video rendering
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based automation system creates a complete content production pipeline: from automated research and news scraping to AI-powered content generation and video rendering. It combines Claude 3, OpenAI, web scraping, and Remotion to transform a single keyword into finished articles and videos.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Scrapes latest news from TechCrunch, a16z, Twitter/X, LinkedIn within 24 hours
2. **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
3. **Multi-Language Support**: Generates content in English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders infographics and short videos using Remotion
5. **Platform Optimization**: Outputs content optimized for Reels, TikTok, YouTube Shorts

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
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_OUTPUT_DIR=./output/videos

# Content Settings
DEFAULT_LANGUAGE=vi
DEFAULT_TONE=professional
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── lib/
│   │   ├── ai/           # AI provider integrations
│   │   ├── scraper/      # Web scraping modules
│   │   ├── content/      # Content generation logic
│   │   └── video/        # Remotion video rendering
│   ├── pages/            # Next.js pages
│   ├── components/       # React components
│   └── utils/            # Utility functions
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Core API Usage

### 1. Research & Data Collection

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';
import { analyzeResearch } from '@/lib/ai/research-analyzer';

async function gatherResearch(keyword: string) {
  // Scrape news from multiple sources
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await scrapeNews({
    keyword,
    sources,
    timeframe: '24h',
    limit: 20
  });

  // Analyze and extract insights using AI
  const insights = await analyzeResearch({
    data: newsData,
    keyword,
    language: 'vi',
    extractMetrics: true
  });

  return {
    rawData: newsData,
    insights,
    trends: insights.trends,
    statistics: insights.stats
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language, Tone } from '@/types';

async function createArticle(
  keyword: string,
  research: any,
  format: ContentFormat = 'toplist'
) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    prompt: {
      keyword,
      research: research.insights,
      format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
      language: 'vi' as Language,
      tone: 'professional' as Tone,
      includeStatistics: true,
      wordCount: 1500
    }
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    sources: content.citations
  };
}
```

### 3. Multi-Language Generation

```typescript
import { translateContent } from '@/lib/ai/translator';

async function generateBilingual(keyword: string, research: any) {
  // Generate Vietnamese version
  const vietnameseContent = await generateContent({
    provider: 'claude',
    prompt: {
      keyword,
      research: research.insights,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly'
    }
  });

  // Generate English version
  const englishContent = await generateContent({
    provider: 'openai',
    model: 'gpt-4-turbo',
    prompt: {
      keyword,
      research: research.insights,
      format: 'toplist',
      language: 'en',
      tone: 'professional'
    }
  });

  return {
    vi: vietnameseContent,
    en: englishContent
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { createVideoScript } from '@/lib/ai/video-script';

async function generateVideo(content: any, platform: string = 'reels') {
  // Create video script from article
  const script = await createVideoScript({
    content: content.body,
    duration: 30, // seconds
    format: platform, // 'reels' | 'tiktok' | 'shorts'
    includeSubtitles: true
  });

  // Define video dimensions based on platform
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
    youtube: { width: 1920, height: 1080 }
  };

  // Render video using Remotion
  const videoPath = await renderVideo({
    compositionId: 'ContentVideo',
    script: script.scenes,
    inputProps: {
      title: content.title,
      scenes: script.scenes,
      bgMusic: 'upbeat',
      subtitles: script.subtitles
    },
    output: `${process.env.REMOTION_OUTPUT_DIR}/${Date.now()}.mp4`,
    ...dimensions[platform]
  });

  return {
    path: videoPath,
    duration: script.duration,
    scenes: script.scenes.length
  };
}
```

## Complete Pipeline Example

```typescript
import { runContentPipeline } from '@/lib/pipeline';

async function automatedContentCreation(keyword: string) {
  try {
    // Execute full pipeline
    const result = await runContentPipeline({
      keyword,
      steps: {
        research: true,
        content: true,
        translation: true,
        video: true
      },
      config: {
        contentFormat: 'toplist',
        languages: ['vi', 'en'],
        videoFormats: ['reels', 'tiktok'],
        tone: 'professional',
        wordCount: 1500
      }
    });

    console.log('Pipeline completed:', {
      researchSources: result.research.sources.length,
      articles: result.content.length,
      videos: result.videos.length
    });

    return result;
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const content = await automatedContentCreation('AI marketing automation');
```

## Advanced Patterns

### Custom Content Format

```typescript
import { createCustomFormat } from '@/lib/ai/format-builder';

const customFormat = createCustomFormat({
  name: 'comparison',
  structure: [
    { section: 'intro', maxWords: 200 },
    { section: 'features', type: 'table', items: 5 },
    { section: 'pros-cons', type: 'list' },
    { section: 'verdict', maxWords: 150 }
  ],
  tone: 'analytical',
  includeDataPoints: true
});

const article = await generateContent({
  provider: 'claude',
  prompt: {
    keyword: 'CRM software comparison',
    research: researchData,
    customFormat,
    language: 'en'
  }
});
```

### Batch Processing

```typescript
import { processBatch } from '@/lib/pipeline/batch';

async function batchContentGeneration(keywords: string[]) {
  const results = await processBatch({
    items: keywords,
    processor: async (keyword) => {
      const research = await gatherResearch(keyword);
      const content = await generateBilingual(keyword, research);
      const video = await generateVideo(content.vi, 'reels');
      
      return { keyword, content, video };
    },
    concurrency: 3, // Process 3 at a time
    onProgress: (completed, total) => {
      console.log(`Progress: ${completed}/${total}`);
    }
  });

  return results;
}
```

### Scheduled Content Pipeline

```typescript
import { scheduleContentGeneration } from '@/lib/scheduler';

scheduleContentGeneration({
  keywords: ['AI trends', 'Marketing automation', 'Content strategy'],
  schedule: '0 9 * * *', // Daily at 9 AM
  pipeline: {
    research: true,
    content: true,
    video: true,
    autoPublish: false // Save for review
  },
  notifications: {
    email: process.env.NOTIFICATION_EMAIL,
    onComplete: true,
    onError: true
  }
});
```

## CLI Commands

If the project includes CLI tools:

```bash
# Generate content from keyword
npm run generate -- --keyword "AI marketing" --format toplist --lang vi

# Run full pipeline
npm run pipeline -- --config pipeline.config.json

# Render videos from existing content
npm run render-video -- --input content.json --platform reels

# Batch process keywords
npm run batch -- --keywords keywords.txt --output ./output
```

## API Routes (Next.js)

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, languages, generateVideo } = req.body;

  try {
    const result = await runContentPipeline({
      keyword,
      steps: {
        research: true,
        content: true,
        translation: languages?.length > 1,
        video: generateVideo
      },
      config: {
        contentFormat: format || 'toplist',
        languages: languages || ['vi'],
        videoFormats: generateVideo ? ['reels'] : [],
        tone: 'professional'
      }
    });

    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Content generation error:', error);
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  openai: { requests: 50, per: 'minute' },
  anthropic: { requests: 40, per: 'minute' },
  rapidapi: { requests: 100, per: 'hour' }
});

await limiter.wait('openai');
const response = await generateContent({ provider: 'openai', ... });
```

### Error Handling

```typescript
import { PipelineError, RecoverableError } from '@/lib/errors';

try {
  const result = await runContentPipeline(config);
} catch (error) {
  if (error instanceof RecoverableError) {
    // Retry with fallback
    console.log('Retrying with fallback provider...');
    const result = await runContentPipeline({
      ...config,
      fallbackProvider: 'openai'
    });
  } else if (error instanceof PipelineError) {
    console.error('Pipeline failed at step:', error.step);
    // Save partial results
    await savePartialResults(error.partialData);
  } else {
    throw error;
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion dependencies
import { getVideoMetadata } from '@remotion/renderer';

try {
  await renderVideo(config);
} catch (error) {
  if (error.message.includes('ffmpeg')) {
    console.error('FFmpeg not found. Install with: npm install @ffmpeg-installer/ffmpeg');
  } else if (error.message.includes('memory')) {
    // Reduce video quality
    await renderVideo({
      ...config,
      scale: 0.5,
      quality: 70
    });
  }
}
```

### Content Quality Issues

```typescript
import { validateContent } from '@/lib/validation';

const content = await generateContent(config);

const validation = validateContent(content, {
  minWords: 1000,
  requireSources: true,
  checkGrammar: true,
  checkPlagiarism: false
});

if (!validation.isValid) {
  console.log('Content issues:', validation.issues);
  // Regenerate with adjusted parameters
  const improvedContent = await generateContent({
    ...config,
    temperature: 0.7, // More creative
    includeMoreSources: true
  });
}
```

## Best Practices

1. **Always validate API keys** before running pipelines
2. **Use rate limiting** to avoid hitting API quotas
3. **Cache research data** for 24 hours to reduce API calls
4. **Test video rendering** with short durations first
5. **Monitor content quality** with validation checks
6. **Store generated content** with metadata for tracking
7. **Use environment-specific configs** for dev/prod

## Integration Examples

### WordPress Auto-Publishing

```typescript
import { publishToWordPress } from '@/lib/integrations/wordpress';

const content = await runContentPipeline({ keyword });

await publishToWordPress({
  url: process.env.WORDPRESS_URL,
  auth: {
    username: process.env.WP_USERNAME,
    password: process.env.WP_APP_PASSWORD
  },
  post: {
    title: content.title,
    content: content.body,
    status: 'draft',
    categories: ['AI', 'Marketing'],
    featured_media: content.featuredImageId
  }
});
```

### Social Media Scheduling

```typescript
import { schedulePost } from '@/lib/integrations/social';

await schedulePost({
  platform: 'facebook',
  content: content.vi.body.substring(0, 500),
  media: [video.path],
  scheduledTime: new Date('2024-12-25T09:00:00Z'),
  pageId: process.env.FB_PAGE_ID
});
```
