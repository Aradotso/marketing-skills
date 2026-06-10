---
name: ultimate-ai-content-pipeline
description: Automated content creation system from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline for marketing
  - generate videos from text automatically
  - create content with Claude and OpenAI APIs
  - automate research and content writing workflow
  - build an AI-powered content generation system
  - use Remotion for automated video rendering
  - scrape news and generate marketing content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that handles research, scriptwriting, and video generation. This pipeline automatically scrapes recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses AI (Claude 3, OpenAI) to generate content in multiple formats and languages, finally rendering videos with Remotion.

## What This Project Does

- **Auto-Research**: Crawls and analyzes real-time data from major tech news sources
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multi-language Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Automatically converts written content into infographics and short videos using Remotion
- **Platform Optimization**: Exports videos in formats suitable for Reels, TikTok, and YouTube Shorts

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for content scraping
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Database (if using)
DATABASE_URL=your_database_connection_string
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/                   # Core utilities
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── scrapers/         # Content scraping modules
│   └── video/            # Remotion video rendering
├── remotion/             # Video templates
└── public/               # Static assets
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Render videos (Remotion)
npm run remotion:render
```

## Core API Usage

### 1. Content Research & Scraping

```typescript
import { scrapeNewsArticles } from '@/lib/scrapers/news-scraper';

async function researchTopic(keyword: string) {
  const articles = await scrapeNewsArticles({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    limit: 20
  });
  
  return articles;
}

// Usage
const research = await researchTopic('AI automation');
console.log(`Found ${research.length} articles`);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `
    Topic: ${topic}
    Format: ${format}
    Language: ${language}
    
    Based on the latest research, create a comprehensive ${format} article.
    Include data-backed insights and current trends.
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
const content = await generateContentWithClaude(
  'Marketing Automation Trends',
  'toplist',
  'en'
);
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithGPT(
  researchData: any[],
  tone: 'professional' | 'friendly' | 'humorous'
) {
  const systemPrompt = `You are an expert content creator. 
    Tone: ${tone}. 
    Create engaging content based on provided research.`;

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { 
        role: 'user', 
        content: `Research data: ${JSON.stringify(researchData)}. 
                  Create a comprehensive article.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Complete Content Pipeline

```typescript
import { scrapeNewsArticles } from '@/lib/scrapers/news-scraper';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { renderVideo } from '@/lib/video/remotion-renderer';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await scrapeNewsArticles({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h'
  });

  // Step 2: Generate content (English)
  console.log('✍️ Generating English content...');
  const contentEN = await generateContentWithClaude(
    keyword,
    'toplist',
    'en'
  );

  // Step 3: Generate content (Vietnamese)
  console.log('✍️ Generating Vietnamese content...');
  const contentVI = await generateContentWithClaude(
    keyword,
    'toplist',
    'vi'
  );

  // Step 4: Render video
  console.log('🎬 Rendering video...');
  const video = await renderVideo({
    content: contentEN,
    format: 'reel', // 9:16 for Instagram/TikTok
    duration: 60
  });

  return {
    research,
    contentEN,
    contentVI,
    videoPath: video.path
  };
}

// Usage
const result = await runContentPipeline('AI in Marketing 2024');
console.log('Pipeline complete!', result);
```

## Remotion Video Rendering

### Basic Video Component

```typescript
// remotion/VideoTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e', padding: 40 }}>
      <h1 style={{ opacity, color: 'white', fontSize: 48 }}>
        {title}
      </h1>
      <ul>
        {points.map((point, i) => {
          const pointOpacity = interpolate(
            frame,
            [30 + i * 20, 50 + i * 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          return (
            <li key={i} style={{ opacity: pointOpacity, color: 'white' }}>
              {point}
            </li>
          );
        })}
      </ul>
    </AbsoluteFill>
  );
};
```

### Render Video Programmatically

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderVideo(props: {
  content: string;
  format: 'reel' | 'landscape';
  duration: number;
}) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: props,
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `output-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: props,
  });

  return { path: outputLocation };
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLanguageContent(topic: string) {
  const languages = ['en', 'vi'];
  const formats = ['toplist', 'pov', 'how-to'];
  
  const results = await Promise.all(
    languages.flatMap(lang =>
      formats.map(format =>
        generateContentWithClaude(topic, format, lang)
          .then(content => ({ lang, format, content }))
      )
    )
  );

  return results;
}
```

### Content Scheduling System

```typescript
interface ContentSchedule {
  content: string;
  publishAt: Date;
  platforms: ('facebook' | 'twitter' | 'linkedin')[];
}

async function scheduleContent(schedule: ContentSchedule) {
  // Store in database with scheduled time
  await db.scheduledPosts.create({
    data: {
      content: schedule.content,
      publishAt: schedule.publishAt,
      platforms: schedule.platforms,
      status: 'pending'
    }
  });
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const batchSize = 3; // Process 3 at a time to avoid rate limits
  const results = [];

  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(keyword => runContentPipeline(keyword))
    );
    results.push(...batchResults);
    
    // Rate limit delay
    if (i + batchSize < keywords.length) {
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }

  return results;
}
```

## API Integration Patterns

### RapidAPI News Scraping

```typescript
async function scrapeWithRapidAPI(query: string) {
  const options = {
    method: 'GET',
    url: 'https://news-api14.p.rapidapi.com/search',
    params: {
      q: query,
      language: 'en',
      sortBy: 'publishedAt'
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
    }
  };

  const response = await fetch(options.url, {
    headers: options.headers
  });

  return await response.json();
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() =>
  generateContentWithClaude('topic', 'toplist', 'en')
);
```

### Memory Issues with Video Rendering

```typescript
// Process videos sequentially for large batches
async function renderVideosSequentially(contentList: any[]) {
  const results = [];
  
  for (const content of contentList) {
    const video = await renderVideo({
      content: content.text,
      format: 'reel',
      duration: 60
    });
    results.push(video);
    
    // Clean up memory between renders
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Error Handling

```typescript
async function safeContentGeneration(topic: string) {
  try {
    const content = await generateContentWithClaude(topic, 'toplist', 'en');
    return { success: true, content };
  } catch (error) {
    console.error('Content generation failed:', error);
    
    // Fallback to OpenAI
    try {
      const fallbackContent = await generateContentWithGPT([{ topic }], 'professional');
      return { success: true, content: fallbackContent, fallback: true };
    } catch (fallbackError) {
      return { 
        success: false, 
        error: 'Both AI providers failed',
        details: fallbackError 
      };
    }
  }
}
```

## Next.js API Routes

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
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

This skill enables AI coding agents to understand and work with the Ultimate AI Content Pipeline for automated content creation, from research through video generation.
