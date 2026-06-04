---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate AI-powered marketing content with video
  - create automated content pipeline with Claude and OpenAI
  - build AI content automation system
  - set up marketing content generation workflow
  - use Remotion for automated video rendering
  - implement AI research to video pipeline
  - create multi-format content with AI automation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research, scriptwriting, multi-format content generation, and automated video rendering. It crawls news sources, generates content in multiple languages and formats, and creates videos using Remotion.

## What This Project Does

**Ultimate AI Content Pipeline** automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders infographics and short-form videos using Remotion
5. **Platform Optimization**: Exports video in formats optimized for Reels, TikTok, Shorts

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

### Environment Setup

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion:dev
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Web scraping & research
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video compositions
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { researchTopic } from '@/lib/research/crawler';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });

  return {
    articles: research.articles,
    insights: research.keyInsights,
    trends: research.trendingTopics,
    dataPoints: research.statistics
  };
}

// Usage
const data = await gatherInsights('AI marketing automation');
console.log(data.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

async function generateContent(request: ContentRequest) {
  const prompt = `
Create a ${request.format} article about ${request.topic} in ${request.language}.
Tone: ${request.tone}

Research data:
${JSON.stringify(request.researchData, null, 2)}

Requirements:
- Use real data and statistics from the research
- Include actionable insights
- Format for ${request.language === 'en' ? 'English' : 'Vietnamese'} readers
- Optimize for social media sharing
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].text;
}

// Usage
const content = await generateContent({
  topic: 'AI Content Marketing Trends 2026',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: data
});
```

### 3. OpenAI Integration for Variations

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentVariations(baseContent: string, count: number = 3) {
  const variations = [];

  for (let i = 0; i < count; i++) {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are a content marketing expert who creates engaging variations of content while maintaining core message and SEO value.'
        },
        {
          role: 'user',
          content: `Create a unique variation of this content for different platform:\n\n${baseContent}`
        }
      ],
      temperature: 0.8,
    });

    variations.push(completion.choices[0].message.content);
  }

  return variations;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

async function generateVideo(config: VideoConfig) {
  // Define video dimensions based on format
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  const { width, height } = dimensions[config.format];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      duration: config.duration,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${config.format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  content: content,
  title: 'AI Marketing Trends 2026',
  format: 'reels',
  duration: 30
});
```

### 5. Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

async function fullContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
      language: 'en'
    });

    // Step 2: Generate Content (English)
    console.log('✍️ Generating English content...');
    const contentEN = await generateContent({
      topic: keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research
    });

    // Step 3: Generate Content (Vietnamese)
    console.log('✍️ Generating Vietnamese content...');
    const contentVI = await generateContent({
      topic: keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData: research
    });

    // Step 4: Generate Videos
    console.log('🎬 Generating videos...');
    const videoReels = await generateVideo({
      content: contentEN,
      title: keyword,
      format: 'reels',
      duration: 30
    });

    const videoTikTok = await generateVideo({
      content: contentVI,
      title: keyword,
      format: 'tiktok',
      duration: 30
    });

    return {
      research,
      content: {
        english: contentEN,
        vietnamese: contentVI
      },
      videos: {
        reels: videoReels,
        tiktok: videoTikTok
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await fullContentPipeline('AI Content Automation');
console.log('✅ Pipeline complete!', result);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();

  try {
    const data = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h',
      language: 'en'
    });

    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  const { topic, format, language, tone, researchData } = await request.json();

  try {
    const content = await generateContent({
      topic,
      format,
      language,
      tone,
      researchData
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

### Video Rendering Endpoint

```typescript
// src/app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/remotion';

export async function POST(request: NextRequest) {
  const { content, title, format, duration } = await request.json();

  try {
    const videoPath = await generateVideo({
      content,
      title,
      format: format || 'reels',
      duration: duration || 30
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

## Configuration

### Content Formats

```typescript
// src/types/content.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

export interface ContentConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  minWords?: number;
  maxWords?: number;
  includeSEO?: boolean;
  includeHashtags?: boolean;
}
```

### Video Templates

```typescript
// remotion/config.ts
export const VIDEO_CONFIGS = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900, // 30 seconds
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900,
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 1800, // 60 seconds
  },
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await researchTopic({ keyword });
      const content = await generateContent({
        topic: keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        researchData: research
      });
      return { keyword, content };
    })
  );

  return results;
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run content pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await fullContentPipeline(topic);
  }
});
```

### Content with SEO Optimization

```typescript
async function generateSEOContent(keyword: string) {
  const research = await researchTopic({ keyword });
  
  const seoPrompt = `
Generate SEO-optimized content for keyword: ${keyword}

Include:
- Meta title (60 chars max)
- Meta description (160 chars max)
- H1, H2, H3 headings
- Target keyword density: 1-2%
- LSI keywords from research
- Internal linking opportunities
`;

  const content = await generateContent({
    topic: seoPrompt,
    format: 'how-to',
    language: 'en',
    tone: 'expert',
    researchData: research
  });

  return content;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function apiCallWithRetry(apiCall: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await apiCall();
    } catch (error) {
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Use chunked rendering for long videos
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(config: VideoConfig) {
  const chunkSize = 300; // 10 seconds at 30fps
  
  // Render in chunks to avoid memory issues
  for (let i = 0; i < config.duration * 30; i += chunkSize) {
    await renderFrames({
      // ... config
      frameRange: [i, Math.min(i + chunkSize, config.duration * 30)]
    });
  }
}
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

### Research Crawler Errors

```typescript
async function safeResearch(keyword: string) {
  try {
    return await researchTopic({ keyword });
  } catch (error) {
    console.error('Research failed:', error);
    
    // Fallback to cached data or simplified research
    return {
      articles: [],
      insights: ['Research temporarily unavailable'],
      trends: [],
      statistics: {}
    };
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement caching** for research data to reduce API calls
3. **Use rate limiting** when making multiple AI requests
4. **Monitor token usage** for Claude and OpenAI to control costs
5. **Validate input data** before passing to AI models
6. **Store generated content** in a database for reuse
7. **Implement error handling** at every pipeline stage
8. **Use TypeScript types** for type safety across the pipeline
