---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video
  - set up marketing pipeline automation
  - generate content from research to video automatically
  - use Claude and OpenAI for content generation
  - create automated content workflow with Remotion
  - build AI-powered marketing content system
  - scrape news and generate social media videos
  - automate research writing and video rendering
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**marketing-pipeline-share** is a complete AI-powered content automation system that handles the entire content lifecycle: from research (crawling news sources like TechCrunch, Twitter, LinkedIn), to content generation (using Claude 3/OpenAI), to video rendering (Remotion). It automates up to 90% of content creation workflows for marketers and content creators.

**Key capabilities:**
- Auto-crawl trending news from multiple sources (last 24h)
- Generate content in multiple formats (Toplist, POV, Case Study, How-to)
- Bilingual support (English/Vietnamese) with customizable tone
- Automatic video/infographic rendering from content
- Next.js interface for managing the pipeline

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
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Social Media APIs (optional)
TWITTER_BEARER_TOKEN=your_twitter_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (Video Rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video preview
npm run remotion:preview

# Render video
npm run remotion:render
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # News crawling & research
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & News Crawling

```typescript
import { crawlNews } from '@/lib/research/crawler';
import { analyzeContent } from '@/lib/research/analyzer';

async function researchTopic(keyword: string) {
  // Crawl news from multiple sources
  const newsData = await crawlNews({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 20
  });

  // Analyze and extract insights
  const insights = await analyzeContent(newsData, {
    extractStats: true,
    identifyTrends: true,
    summarize: true
  });

  return insights;
}

// Usage
const aiInsights = await researchTopic('artificial intelligence');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Based on this research data: ${JSON.stringify(research)}
  
  Generate a ${format} article in ${language}.
  Tone: Professional yet engaging
  Include: Data-backed insights, actionable takeaways
  Length: 1000-1500 words`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Generate bilingual content
const enContent = await generateContent(aiInsights, 'toplist', 'en');
const viContent = await generateContent(aiInsights, 'toplist', 'vi');
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(research: any, format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and tech trends.'
      },
      {
        role: 'user',
        content: `Create a ${format} article based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: {
  title: string;
  points: string[];
  stats: { label: string; value: string }[];
}) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentInfographic',
    inputProps: content,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.title.replace(/\s/g, '-')}.mp4`,
    inputProps: content,
  });

  console.log('Video rendered successfully!');
}

// Usage
await generateVideo({
  title: 'Top 5 AI Trends 2024',
  points: [
    'Multimodal AI becomes mainstream',
    'AI agents in enterprise',
    'Open source LLMs gain traction',
  ],
  stats: [
    { label: 'Market Growth', value: '+150%' },
    { label: 'Adoption Rate', value: '67%' },
  ],
});
```

### 5. Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/research';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { publishToSocial } from '@/lib/publish';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic(keyword);

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent(research, 'toplist', 'en');
    
    // Step 3: Extract key points for video
    const videoData = {
      title: research.mainTrend,
      points: research.topInsights.slice(0, 5),
      stats: research.keyStats,
    };

    // Step 4: Render Video
    console.log('🎬 Rendering video...');
    await generateVideo(videoData);

    // Step 5: Publish (optional)
    console.log('📤 Publishing...');
    await publishToSocial({
      text: content,
      videoPath: `out/${videoData.title.replace(/\s/g, '-')}.mp4`,
      platforms: ['twitter', 'linkedin'],
    });

    console.log('✅ Pipeline completed!');
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
await runContentPipeline('generative AI marketing');
```

## API Routes (Next.js)

### POST /api/research

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();

  try {
    const data = await crawlNews({ keyword, sources, timeframe });
    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### POST /api/generate

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  const { research, format, language } = await request.json();

  try {
    const content = await generateContent(research, format, language);
    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### POST /api/render-video

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/remotion';

export async function POST(request: NextRequest) {
  const videoData = await request.json();

  try {
    const result = await generateVideo(videoData);
    return NextResponse.json({ success: true, result });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Custom Tone/Voice

```typescript
interface ToneConfig {
  style: 'professional' | 'friendly' | 'humorous' | 'academic';
  audience: 'experts' | 'general' | 'beginners';
  language: 'en' | 'vi';
}

async function generateWithTone(
  research: any,
  format: string,
  tone: ToneConfig
) {
  const tonePrompt = `
    Style: ${tone.style}
    Audience: ${tone.audience}
    Language: ${tone.language}
    
    ${tone.style === 'humorous' ? 'Use witty analogies and light humor.' : ''}
    ${tone.audience === 'experts' ? 'Use technical terminology.' : 'Explain concepts simply.'}
  `;

  // Include tonePrompt in your AI generation
  return await generateContent(research, format, tone.language, tonePrompt);
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await researchTopic(keyword);
      const content = await generateContent(research, 'toplist', 'en');
      return { keyword, content };
    })
  );

  return results
    .filter((r) => r.status === 'fulfilled')
    .map((r) => r.value);
}
```

### Scheduled Automation

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const todayKeyword = getTrendingKeyword(); // Your logic
  await runContentPipeline(todayKeyword);
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => generateContent(research, 'toplist', 'en'));
```

### Memory Issues with Video Rendering

```typescript
// Render in chunks for long videos
import { getCompositions } from '@remotion/renderer';

async function renderLargeVideo(segments: any[]) {
  const renderedPaths: string[] = [];
  
  for (const [index, segment] of segments.entries()) {
    const outputPath = `out/segment-${index}.mp4`;
    await renderMedia({
      composition: segment.composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      chromiumOptions: {
        // Reduce memory usage
        headless: true,
        args: ['--no-sandbox', '--disable-setuid-sandbox'],
      },
    });
    renderedPaths.push(outputPath);
  }
  
  // Concatenate segments using ffmpeg
  return concatenateVideos(renderedPaths);
}
```

### Missing Research Data

```typescript
// Fallback to cached data or alternative sources
async function safeResearch(keyword: string) {
  try {
    return await crawlNews({ keyword, sources: ['techcrunch'] });
  } catch (error) {
    console.warn('Primary source failed, trying fallback...');
    try {
      return await crawlNews({ keyword, sources: ['twitter'] });
    } catch (fallbackError) {
      // Use cached data or throw
      return getCachedResearch(keyword);
    }
  }
}
```

## TypeScript Types

```typescript
// types/content.ts
export interface ResearchData {
  keyword: string;
  sources: string[];
  articles: Article[];
  insights: Insight[];
  stats: Statistic[];
  timestamp: Date;
}

export interface Article {
  title: string;
  url: string;
  source: string;
  publishedAt: Date;
  content: string;
}

export interface ContentFormat {
  type: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
}

export interface VideoConfig {
  title: string;
  points: string[];
  stats: Statistic[];
  duration: number;
  aspectRatio: '16:9' | '9:16' | '1:1';
}
```

## Best Practices

1. **Always validate research data** before passing to AI
2. **Cache research results** to avoid redundant API calls
3. **Use environment variables** for all API keys
4. **Implement proper error handling** in the pipeline
5. **Monitor AI token usage** to control costs
6. **Test video templates** before batch rendering
7. **Set rate limits** when crawling news sources
8. **Version control your prompts** for consistency
