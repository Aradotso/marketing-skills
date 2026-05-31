---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up marketing pipeline for auto-posting
  - generate video content from scripts automatically
  - crawl news sources and create content
  - build AI-powered content automation system
  - create multilingual marketing content with Claude
  - render videos from text using Remotion
  - automate social media content pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based automation system that creates a complete content production pipeline: from researching trending topics by crawling news sources (TechCrunch, a16z, Twitter, LinkedIn), to generating scripts with Claude/OpenAI in multiple formats and languages, to automatically rendering videos with Remotion. It's designed for content creators, marketers, and businesses to optimize their workflow by up to 90%.

## What It Does

The system provides four core capabilities:

1. **Auto-Scan Research**: Crawls real-time data from major tech news sources and social platforms within the last 24 hours
2. **AI Content Generation**: Uses Claude 3/OpenAI to create diverse content formats (Toplist, POV, Case Study, How-to) in multiple languages with customizable tone
3. **Video Rendering**: Automatically generates infographics and short-form videos from written content using Remotion
4. **Multi-Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts with proper aspect ratios

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

```env
# AI Providers
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database (if using)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   ├── config/          # Configuration files
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Content Crawling

```typescript
import { crawlNewsSources } from '@/lib/crawler';

// Crawl news from multiple sources
async function fetchLatestNews(keyword: string) {
  const sources = {
    techcrunch: true,
    a16z: true,
    twitter: true,
    linkedin: true
  };

  const results = await crawlNewsSource({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });

  return results;
}

// Example usage
const newsData = await fetchLatestNews('AI automation');
console.log(newsData.articles);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = `
Create a ${format} article about: ${topic}
Language: ${language}
Tone: ${tone}
Include recent data and insights.
Format with proper headings and bullet points.
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

// Generate bilingual content
const englishContent = await generateContent(
  'AI in Marketing 2024',
  'toplist',
  'en',
  'expert'
);

const vietnameseContent = await generateContent(
  'AI trong Marketing 2024',
  'toplist',
  'vi',
  'friendly'
);
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  researchData: any[],
  contentType: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and tech trends.'
      },
      {
        role: 'user',
        content: `Based on this research data: ${JSON.stringify(researchData)}, create a ${contentType} article.`
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

async function renderContentVideo(
  scriptData: {
    title: string;
    points: string[];
    duration: number;
  },
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const compositionId = 'ContentVideo';
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: compositionId,
    inputProps: scriptData,
  });

  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: scriptData,
  });

  return outputPath;
}

// Usage
const videoPath = await renderContentVideo(
  {
    title: 'Top 5 AI Marketing Tools 2024',
    points: [
      'AI-powered content generation',
      'Automated social media scheduling',
      'Predictive analytics',
      'Customer segmentation',
      'ROI tracking'
    ],
    duration: 30
  },
  'reels'
);
```

## Complete Pipeline Example

```typescript
import { crawlNewsSource } from '@/lib/crawler';
import { generateContent } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/remotion';
import { publishToSocial } from '@/lib/publishing';

async function runFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching latest news...');
    const research = await crawlNewsSource({
      keyword,
      sources: { techcrunch: true, twitter: true },
      timeRange: '24h',
      limit: 10
    });

    // Step 2: Generate content in both languages
    console.log('✍️ Generating content...');
    const englishArticle = await generateContent(
      keyword,
      'toplist',
      'en',
      'expert'
    );

    const vietnameseArticle = await generateContent(
      keyword,
      'toplist',
      'vi',
      'friendly'
    );

    // Step 3: Parse content for video script
    const videoScript = {
      title: englishArticle.split('\n')[0],
      points: englishArticle
        .split('\n')
        .filter(line => line.startsWith('-'))
        .slice(0, 5),
      duration: 30
    };

    // Step 4: Render video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo(
      videoScript,
      'reels'
    );

    // Step 5: Schedule publishing (optional)
    console.log('📤 Publishing content...');
    await publishToSocial({
      platforms: ['facebook', 'instagram', 'tiktok'],
      content: {
        en: englishArticle,
        vi: vietnameseArticle
      },
      video: videoPath,
      scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
    });

    return {
      success: true,
      articles: { en: englishArticle, vi: vietnameseArticle },
      video: videoPath
    };

  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runFullPipeline('AI automation trends').then(result => {
  console.log('✅ Pipeline completed:', result);
});
```

## Next.js API Routes

Create API endpoints for the web interface:

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNewsSource } from '@/lib/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources } = await request.json();

  const results = await crawlNewsSource({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });

  return NextResponse.json(results);
}

// app/api/generate/route.ts
export async function POST(request: NextRequest) {
  const { topic, format, language, tone } = await request.json();

  const content = await generateContent(topic, format, language, tone);

  return NextResponse.json({ content });
}

// app/api/render-video/route.ts
export async function POST(request: NextRequest) {
  const { scriptData, platform } = await request.json();

  const videoPath = await renderContentVideo(scriptData, platform);

  return NextResponse.json({ videoPath });
}
```

## Common Patterns

### Content Format Templates

```typescript
const formatTemplates = {
  toplist: {
    structure: 'numbered list with explanations',
    sections: ['intro', 'items', 'conclusion'],
    tone: 'authoritative'
  },
  pov: {
    structure: 'opinion-based narrative',
    sections: ['hook', 'argument', 'counterpoint', 'conclusion'],
    tone: 'conversational'
  },
  caseStudy: {
    structure: 'problem-solution-results',
    sections: ['challenge', 'approach', 'implementation', 'results'],
    tone: 'analytical'
  },
  howTo: {
    structure: 'step-by-step guide',
    sections: ['overview', 'prerequisites', 'steps', 'tips'],
    tone: 'instructional'
  }
};
```

### Batch Processing Multiple Keywords

```typescript
async function batchProcessKeywords(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runFullPipeline(keyword))
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

## Troubleshooting

**API Rate Limits:**
```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff(fn: Function, maxRetries = 3) {
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
}
```

**Remotion Rendering Errors:**
- Ensure ffmpeg is installed: `brew install ffmpeg` (Mac) or `apt-get install ffmpeg` (Linux)
- Check composition dimensions match platform requirements
- Verify input props structure matches composition expectations

**News Crawling Failures:**
- Validate RapidAPI key is active and has sufficient credits
- Check network connectivity and firewall settings
- Implement fallback to cached data if live crawling fails

**Memory Issues with Large Batches:**
```typescript
// Process in chunks
async function processInChunks<T>(
  items: T[],
  chunkSize: number,
  processor: (chunk: T[]) => Promise<any>
) {
  const results = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await processor(chunk);
    results.push(...chunkResults);
  }
  return results;
}
```
