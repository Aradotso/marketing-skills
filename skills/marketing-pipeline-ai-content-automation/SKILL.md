---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scripting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI research
  - set up marketing pipeline with video generation
  - automate content creation from research to video
  - use Claude and OpenAI for content automation
  - generate videos from AI written content
  - build automated marketing content workflow
  - create AI powered content pipeline
  - automatically research and generate social media content
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline AI Content Automation is a comprehensive TypeScript-based system that automates the entire content creation workflow: from real-time research (crawling TechCrunch, a16z, Twitter, LinkedIn), to AI-powered script generation (using Claude 3 and OpenAI), to automatic video rendering (via Remotion). It supports multiple content formats (toplists, POVs, case studies, how-tos), multilingual output (English/Vietnamese), and exports optimized videos for TikTok, Reels, and Shorts.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the project root:

```env
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Research/crawling modules
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Auto-Research & Content Crawling

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });

  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics,
    trends: research.trends
  };
}

// Usage
const insights = await gatherInsights('AI marketing automation');
console.log(insights.trends);
```

### 2. AI Content Generation with Claude/OpenAI

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createContentPiece(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format, // 'toplist', 'pov', 'case-study', 'how-to'
    aiProvider: 'claude', // or 'openai'
    tone: 'professional', // 'friendly', 'humorous'
    languages: ['en', 'vi'],
    includeResearch: true
  });

  return content;
}

// Generate multi-format content
const article = await createContentPiece(
  'Top 10 AI Tools for Content Creators',
  'toplist'
);

console.log(article.english);
console.log(article.vietnamese);
```

### 3. Claude API Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, research: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Based on this research data: ${JSON.stringify(research)}
        
        Create a comprehensive article about: ${prompt}
        
        Include:
        - Data-backed insights
        - Current trends from the last 24 hours
        - Actionable takeaways
        - SEO-optimized structure`
      }
    ]
  });

  return message.content[0].text;
}
```

### 4. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  topic: string,
  format: string,
  researchData: any
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${format} format.`
      },
      {
        role: 'user',
        content: `Create content about ${topic} using this research: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: any, platform: string) {
  // Platform-specific aspect ratios
  const aspectRatios = {
    'tiktok': '9:16',
    'reels': '9:16',
    'shorts': '9:16',
    'youtube': '16:9'
  };

  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      statistics: content.statistics,
      aspectRatio: aspectRatios[platform]
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.slug}-${platform}.mp4`,
  });

  return `out/${content.slug}-${platform}.mp4`;
}

// Generate for multiple platforms
const platforms = ['tiktok', 'reels', 'youtube'];
for (const platform of platforms) {
  await generateVideo(articleContent, platform);
}
```

### 6. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runCompletePipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    autoPublish: false
  });

  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  });

  // Step 2: Generate Content
  const content = await pipeline.generateContent({
    research,
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'professional'
  });

  // Step 3: Create Videos
  const videos = await pipeline.generateVideos({
    content,
    platforms: ['tiktok', 'reels', 'youtube']
  });

  // Step 4: Schedule Publishing (optional)
  if (pipeline.config.autoPublish) {
    await pipeline.schedulePublish({
      content,
      videos,
      platforms: ['facebook', 'instagram', 'youtube'],
      publishDate: new Date(Date.now() + 24 * 60 * 60 * 1000)
    });
  }

  return {
    content,
    videos,
    research
  };
}

// Execute pipeline
const result = await runCompletePipeline('AI content automation 2026');
```

## API Endpoints (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(req: NextRequest) {
  const { keyword, format, languages } = await req.json();

  try {
    const content = await generateContent({
      topic: keyword,
      format,
      languages,
      includeResearch: true
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

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { contentId, platforms } = await req.json();

  try {
    const videos = await Promise.all(
      platforms.map(platform => generateVideo(contentId, platform))
    );

    return NextResponse.json({ success: true, videos });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Content Format Templates

```typescript
export const ContentFormats = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 15,
    includeRankings: true
  },
  pov: {
    structure: ['hook', 'perspective', 'arguments', 'conclusion'],
    tone: 'opinionated',
    includeCounterarguments: true
  },
  caseStudy: {
    structure: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true,
    dataPoints: 'required'
  },
  howTo: {
    structure: ['intro', 'steps', 'tips', 'conclusion'],
    stepByStep: true,
    includeVisuals: true
  }
};
```

### Research Data Processing

```typescript
interface ResearchData {
  articles: Article[];
  insights: Insight[];
  statistics: Statistic[];
  trends: Trend[];
}

async function processResearchData(raw: any): Promise<ResearchData> {
  return {
    articles: raw.articles.map(a => ({
      title: a.title,
      source: a.source,
      url: a.url,
      publishedAt: new Date(a.published_at),
      summary: a.summary
    })),
    insights: extractInsights(raw),
    statistics: extractStatistics(raw),
    trends: analyzeTrends(raw)
  };
}
```

### Multi-Language Content Generation

```typescript
async function generateMultiLanguage(
  topic: string,
  languages: string[]
) {
  const baseContent = await generateContent({
    topic,
    language: 'en',
    format: 'toplist'
  });

  const translations = await Promise.all(
    languages.filter(lang => lang !== 'en').map(async (lang) => {
      const translated = await translateContent(baseContent, lang);
      return { language: lang, content: translated };
    })
  );

  return {
    original: { language: 'en', content: baseContent },
    translations
  };
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

# Run type checking
npm run type-check

# Render Remotion video (development)
npm run remotion:render

# Preview Remotion composition
npm run remotion:preview
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Use chunked rendering for large videos
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(composition: any) {
  const chunkSize = 30; // frames per chunk
  const totalFrames = composition.durationInFrames;

  for (let start = 0; start < totalFrames; start += chunkSize) {
    await renderFrames({
      composition,
      serveUrl: bundleLocation,
      frameRange: [start, Math.min(start + chunkSize, totalFrames)],
      onFrameUpdate: (frame) => {
        console.log(`Rendered frame ${frame}`);
      }
    });
  }
}
```

### Claude/OpenAI Response Validation

```typescript
function validateAIResponse(response: any, requiredFields: string[]) {
  const missing = requiredFields.filter(field => !response[field]);
  
  if (missing.length > 0) {
    throw new Error(
      `AI response missing required fields: ${missing.join(', ')}`
    );
  }
  
  return true;
}

// Usage
const content = await generateWithClaude(prompt, research);
validateAIResponse(content, ['title', 'body', 'keyPoints']);
```

### Research Crawler Failures

```typescript
async function safeResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  const results = await Promise.allSettled(
    sources.map(source => crawlSource(source, keyword))
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);

  if (successful.length === 0) {
    throw new Error('All research sources failed');
  }

  return combineResearchData(successful);
}
```

## Best Practices

1. **Always validate environment variables** on startup
2. **Cache research data** to avoid redundant API calls
3. **Use TypeScript types** for all content structures
4. **Implement proper error handling** for API failures
5. **Test video rendering** with sample data before production
6. **Monitor API usage** to stay within rate limits
7. **Version control video templates** in Remotion for consistency
