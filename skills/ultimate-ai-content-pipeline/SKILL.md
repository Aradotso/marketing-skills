---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude, OpenAI) and Remotion for Vietnamese/English marketing content
triggers:
  - generate automated marketing content with AI
  - scrape news and create content pipeline
  - build content from research to video automatically
  - create multilingual marketing posts with Claude
  - automate content research and scriptwriting
  - generate social media videos from text content
  - set up AI content automation workflow
  - create data-backed marketing content automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end automated content creation pipeline that handles research, scriptwriting, and video generation. It crawls fresh data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses AI (Claude 3/OpenAI) to generate content in multiple formats and languages, finally rendering videos with Remotion.

## What It Does

- **Auto-Research**: Crawls real-time news and insights from major tech sources
- **AI Content Generation**: Creates multilingual content (EN/VI) in various formats (toplist, POV, case study, how-to)
- **Video Rendering**: Automatically generates infographics and short-form videos optimized for Reels/TikTok/Shorts
- **Multi-Format Output**: Supports different content tones (expert, friendly, humorous) and platform requirements

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following variables:

```env
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scrapers/    # News crawling modules
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript definitions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Scraping

```typescript
import { scrapeLatestNews } from '@/lib/scrapers/news-scraper';

interface NewsResult {
  title: string;
  url: string;
  source: string;
  publishedAt: Date;
  content: string;
  insights: string[];
}

// Scrape news from multiple sources
async function gatherResearch(keyword: string): Promise<NewsResult[]> {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await scrapeLatestNews({
    keyword,
    sources,
    timeRange: '24h',
    limit: 10
  });
  
  return results;
}

// Usage
const research = await gatherResearch('AI automation');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: NewsResult[];
}

async function generateContent(request: ContentRequest) {
  const prompt = `
Create a ${request.format} article about ${request.keyword} in ${request.language}.
Tone: ${request.tone}

Research data:
${request.research.map(r => `- ${r.title}: ${r.insights.join(', ')}`).join('\n')}

Generate:
1. Catchy headline
2. Introduction
3. Main content (3-5 sections)
4. Data-backed insights
5. Call-to-action
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(request: ContentRequest) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${request.tone} content writer specializing in ${request.format} articles.`
      },
      {
        role: 'user',
        content: `Create content about ${request.keyword} based on this research: ${JSON.stringify(request.research)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

async function generateVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: config.content,
      format: config.format,
    },
  });

  const outputPath = path.join(
    process.cwd(), 
    'output', 
    `video-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
  });

  return outputPath;
}
```

## Complete Pipeline Example

```typescript
import { scrapeLatestNews } from '@/lib/scrapers/news-scraper';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('📡 Gathering research...');
    const research = await scrapeLatestNews({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h',
      limit: 10
    });

    // Step 2: Generate content (English)
    console.log('🧠 Generating English content...');
    const englishContent = await generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      research
    });

    // Step 3: Generate content (Vietnamese)
    console.log('🧠 Generating Vietnamese content...');
    const vietnameseContent = await generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      research
    });

    // Step 4: Generate videos
    console.log('🎬 Rendering videos...');
    const reelsVideo = await generateVideo({
      content: englishContent,
      format: 'reels',
      duration: 30
    });

    const tiktokVideo = await generateVideo({
      content: vietnameseContent,
      format: 'tiktok',
      duration: 60
    });

    return {
      research,
      content: {
        english: englishContent,
        vietnamese: vietnameseContent
      },
      videos: {
        reels: reelsVideo,
        tiktok: tiktokVideo
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline('AI automation trends 2024');
console.log('✅ Pipeline complete!', result);
```

## Next.js API Routes

### Create Content API

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, formats, languages } = await request.json();

    const result = await runContentPipeline(keyword);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Research API

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeLatestNews } from '@/lib/scrapers/news-scraper';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');
  const sources = searchParams.get('sources')?.split(',') || [];

  const research = await scrapeLatestNews({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });

  return NextResponse.json({ data: research });
}
```

## CLI Usage (if applicable)

```bash
# Generate content
npm run generate -- --keyword "AI trends" --format toplist --language en

# Run research only
npm run research -- --keyword "marketing automation" --sources techcrunch,a16z

# Generate video
npm run video -- --input content.json --format reels --duration 30

# Full pipeline
npm run pipeline -- --keyword "content marketing" --all-formats
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}

// Generate content for multiple topics
const topics = ['AI automation', 'Content marketing', 'SEO trends'];
const results = await batchGenerate(topics);
```

### Custom Content Templates

```typescript
interface ContentTemplate {
  name: string;
  structure: string[];
  variables: Record<string, string>;
}

const templates: Record<string, ContentTemplate> = {
  'product-launch': {
    name: 'Product Launch',
    structure: [
      'hook',
      'problem',
      'solution',
      'features',
      'benefits',
      'social-proof',
      'cta'
    ],
    variables: {
      productName: '',
      targetAudience: '',
      painPoints: ''
    }
  },
  'thought-leadership': {
    name: 'Thought Leadership',
    structure: [
      'bold-statement',
      'context',
      'unique-perspective',
      'data-insights',
      'future-implications',
      'call-to-discussion'
    ],
    variables: {
      topic: '',
      stance: '',
      industry: ''
    }
  }
};

async function generateFromTemplate(
  templateId: string,
  variables: Record<string, string>
) {
  const template = templates[templateId];
  
  const prompt = `
Create content following this structure:
${template.structure.map((section, i) => `${i + 1}. ${section}`).join('\n')}

Variables:
${Object.entries(variables).map(([k, v]) => `${k}: ${v}`).join('\n')}
`;

  return await generateContent({
    keyword: variables.topic || variables.productName,
    format: 'custom',
    language: 'en',
    tone: 'expert',
    research: [],
    customPrompt: prompt
  });
}
```

### Scheduling Content

```typescript
import { CronJob } from 'cron';

// Schedule daily content generation
const dailyContentJob = new CronJob(
  '0 9 * * *', // Every day at 9 AM
  async () => {
    const trending = await fetchTrendingTopics();
    await runContentPipeline(trending[0]);
  },
  null,
  true,
  'America/New_York'
);

// Weekly batch generation
const weeklyBatchJob = new CronJob(
  '0 10 * * 1', // Every Monday at 10 AM
  async () => {
    const weeklyTopics = await getWeeklyContentCalendar();
    await batchGenerate(weeklyTopics);
  }
);
```

## Troubleshooting

### API Rate Limits

```typescript
import pRetry from 'p-retry';

async function generateWithRetry(request: ContentRequest) {
  return pRetry(
    async () => {
      return await generateContent(request);
    },
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(
          `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
        );
      }
    }
  );
}
```

### Memory Management for Video Rendering

```typescript
// Process videos sequentially to avoid memory issues
async function generateVideosSequentially(contents: string[]) {
  const videos = [];
  
  for (const content of contents) {
    const video = await generateVideo({
      content,
      format: 'reels',
      duration: 30
    });
    videos.push(video);
    
    // Allow garbage collection between renders
    if (global.gc) {
      global.gc();
    }
  }
  
  return videos;
}
```

### Handling Scraper Failures

```typescript
async function scrapeWithFallback(keyword: string) {
  const primarySources = ['techcrunch', 'a16z'];
  const fallbackSources = ['twitter', 'linkedin'];
  
  try {
    return await scrapeLatestNews({
      keyword,
      sources: primarySources,
      timeRange: '24h'
    });
  } catch (error) {
    console.warn('Primary sources failed, trying fallback...');
    return await scrapeLatestNews({
      keyword,
      sources: fallbackSources,
      timeRange: '48h'
    });
  }
}
```

### Debug Mode

```typescript
// Enable verbose logging
const DEBUG = process.env.DEBUG === 'true';

function debugLog(stage: string, data: any) {
  if (DEBUG) {
    console.log(`[${stage}]`, JSON.stringify(data, null, 2));
  }
}

async function runContentPipelineDebug(keyword: string) {
  debugLog('INPUT', { keyword });
  
  const research = await scrapeLatestNews({ keyword });
  debugLog('RESEARCH', research);
  
  const content = await generateContent({ keyword, research });
  debugLog('CONTENT', content);
  
  return content;
}
```

## Performance Tips

- Use streaming responses for real-time content generation feedback
- Cache research results for 24 hours to reduce API calls
- Generate videos asynchronously in background workers
- Implement request queuing for high-volume content generation
- Use CDN for generated video assets
