---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, script generation, and video creation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate video content from research automatically
  - create marketing content using Claude and OpenAI
  - build automated content research pipeline
  - set up AI content generation workflow
  - use Remotion for automated video rendering
  - create multilingual marketing content with AI
  - automate social media content production
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, an end-to-end content automation system that handles research, script generation, and video creation. The pipeline automatically crawls news sources, generates content in multiple formats and languages using Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

Marketing Pipeline Share is a complete content production factory that:

- **Auto-scans research sources** (TechCrunch, a16z, Twitter, LinkedIn) for trending topics
- **Generates multi-format content** (toplist, POV, case study, how-to) using AI
- **Produces bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

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
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_api_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Application Settings
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── services/        # Business logic services
│   └── utils/           # Utility functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Core API Usage

### 1. Research & Content Crawling

```typescript
import { ContentCrawler } from '@/lib/crawler/content-crawler';

// Initialize crawler
const crawler = new ContentCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'marketing', 'automation']
});

// Fetch trending content
const research = await crawler.scan();

// Get insights
const insights = await crawler.extractInsights(research);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a ${format} article about ${topic}. 
      Use recent data and insights. 
      Write in both English and Vietnamese.
      Tone: professional yet engaging.`
    }]
  });

  return message.content;
}

// Usage
const content = await generateContent('AI Marketing Trends 2026', 'toplist');
```

### 3. OpenAI Alternative for Content Generation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, language: 'en' | 'vi') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert marketing content creator. 
        Generate content in ${language === 'vi' ? 'Vietnamese' : 'English'}.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Multi-Format Content Pipeline

```typescript
import { ContentPipeline } from '@/services/content-pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  languages: ['en', 'vi'],
  formats: ['toplist', 'pov', 'case-study']
});

async function runPipeline(keyword: string) {
  // Step 1: Research
  const research = await pipeline.research(keyword);
  
  // Step 2: Generate content in multiple formats
  const contents = await pipeline.generateMultiFormat(research);
  
  // Step 3: Create videos
  const videos = await pipeline.renderVideos(contents);
  
  return {
    research,
    contents,
    videos
  };
}

// Execute pipeline
const result = await runPipeline('AI Content Marketing');
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: any, platform: 'reels' | 'tiktok' | 'shorts') {
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.points,
      language: content.language,
    },
  });

  // Render video
  const outputLocation = `public/videos/${content.id}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    ...dimensions[platform],
  });

  return outputLocation;
}
```

### 6. Complete Content Generation Workflow

```typescript
import { z } from 'zod';

// Define content schema
const ContentSchema = z.object({
  topic: z.string(),
  format: z.enum(['toplist', 'pov', 'case-study', 'how-to']),
  language: z.enum(['en', 'vi']),
  tone: z.enum(['professional', 'friendly', 'humorous']),
  platforms: z.array(z.enum(['reels', 'tiktok', 'shorts', 'blog'])),
});

type ContentConfig = z.infer<typeof ContentSchema>;

async function createCompleteContent(config: ContentConfig) {
  // Validate input
  const validated = ContentSchema.parse(config);

  // 1. Research phase
  const crawler = new ContentCrawler({
    keywords: [validated.topic],
    timeRange: '24h',
  });
  const researchData = await crawler.scan();

  // 2. Content generation
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = `
    Topic: ${validated.topic}
    Format: ${validated.format}
    Language: ${validated.language}
    Tone: ${validated.tone}
    
    Research data:
    ${JSON.stringify(researchData, null, 2)}
    
    Create comprehensive content based on the research.
    Include data-backed insights and actionable takeaways.
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 8192,
    messages: [{ role: 'user', content: prompt }],
  });

  const generatedContent = message.content;

  // 3. Video rendering for selected platforms
  const videos = await Promise.all(
    validated.platforms
      .filter(p => ['reels', 'tiktok', 'shorts'].includes(p))
      .map(platform => renderContentVideo(generatedContent, platform as any))
  );

  return {
    research: researchData,
    content: generatedContent,
    videos,
  };
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { topic, format, language } = body;

    // Validate
    if (!topic || !format || !language) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Generate content
    const result = await createCompleteContent({
      topic,
      format,
      language,
      tone: 'professional',
      platforms: ['blog', 'reels'],
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentCrawler } from '@/lib/crawler/content-crawler';

export async function POST(request: NextRequest) {
  const { keywords, timeRange = '24h' } = await request.json();

  const crawler = new ContentCrawler({
    sources: ['techcrunch', 'a16z', 'twitter'],
    keywords,
    timeRange,
  });

  const data = await crawler.scan();
  const insights = await crawler.extractInsights(data);

  return NextResponse.json({ data, insights });
}
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function batchGenerateContent(topics: string[]) {
  const results = await Promise.all(
    topics.map(async (topic) => {
      return await createCompleteContent({
        topic,
        format: 'toplist',
        language: 'en',
        tone: 'professional',
        platforms: ['blog', 'reels'],
      });
    })
  );

  return results;
}
```

### Pattern 2: Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Running daily content generation...');
  
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics.slice(0, 3)) {
    await createCompleteContent({
      topic,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      platforms: ['tiktok', 'reels'],
    });
  }
});
```

### Pattern 3: Multi-Language Content Generation

```typescript
async function generateMultiLanguageContent(topic: string) {
  const languages = ['en', 'vi'] as const;
  
  const contents = await Promise.all(
    languages.map(async (lang) => {
      return {
        language: lang,
        content: await generateWithOpenAI(topic, lang),
      };
    })
  );

  return contents;
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting and retries
import pRetry from 'p-retry';

async function generateWithRetry(prompt: string) {
  return pRetry(
    async () => {
      const anthropic = new Anthropic({
        apiKey: process.env.ANTHROPIC_API_KEY,
      });
      
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });
    },
    {
      retries: 3,
      minTimeout: 2000,
      maxTimeout: 10000,
    }
  );
}
```

### Issue: Remotion Rendering Fails

```typescript
// Add error handling and logging
async function safeRenderVideo(content: any, platform: string) {
  try {
    return await renderContentVideo(content, platform as any);
  } catch (error) {
    console.error(`Video rendering failed for ${platform}:`, error);
    
    // Fallback: generate static image instead
    return await generateStaticImage(content);
  }
}
```

### Issue: Crawler Blocked

```typescript
// Use rotating user agents and delays
import { chromium } from 'playwright';

async function crawlWithPlaywright(url: string) {
  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext({
    userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
  });
  
  const page = await context.newPage();
  
  try {
    await page.goto(url, { waitUntil: 'networkidle' });
    const content = await page.content();
    return content;
  } finally {
    await browser.close();
  }
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview
npm run remotion:preview

# Render Remotion video
npm run remotion:render

# Run tests
npm test

# Lint code
npm run lint
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** to avoid API quota exhaustion
3. **Cache research data** to reduce redundant API calls
4. **Use TypeScript types** for content schemas
5. **Handle errors gracefully** with fallback mechanisms
6. **Monitor API costs** when using Claude/OpenAI extensively
7. **Optimize Remotion compositions** for faster rendering
8. **Validate user inputs** before processing

This skill enables comprehensive automation of marketing content creation workflows using modern AI and video rendering technologies.
