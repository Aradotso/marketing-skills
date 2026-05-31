---
name: marketing-pipeline-share-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese and English marketing content
triggers:
  - how do I automate content creation with AI research and video generation
  - set up marketing pipeline for auto content from keyword to video
  - use Claude and OpenAI to generate multilingual marketing content
  - create automated content workflow with research crawling and video rendering
  - build AI content pipeline with TechCrunch research and Remotion videos
  - generate marketing content automatically from trending topics to social media posts
  - automate Vietnamese and English content creation with AI and video
  - set up content automation pipeline with web scraping and AI writing
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with Marketing Pipeline Share, an end-to-end content automation system that handles research (web scraping), AI content generation (Claude/OpenAI), and video rendering (Remotion) for Vietnamese and English marketing content.

## What This Project Does

Marketing Pipeline Share is a TypeScript/Next.js-based content automation pipeline that:

1. **Auto-Research**: Crawls trending topics from TechCrunch, a16z, Twitter, LinkedIn within 24h
2. **AI Content Generation**: Creates multilingual content (English/Vietnamese) in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 and OpenAI
3. **Video Generation**: Automatically renders infographics and short-form videos using Remotion for Reels/TikTok/Shorts
4. **Multi-Platform Publishing**: Prepares content for automatic posting to various social platforms

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
cp .env.example .env.local
```

### Required Environment Variables

```bash
# .env.local
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPID_API_KEY=your_rapidapi_key

# Optional for specific features
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Remotion configuration
REMOTION_RENDERER_CONCURRENCY=4
```

### Start Development Server

```bash
npm run dev
# Application runs on http://localhost:3000
```

## Core Architecture

```typescript
// src/types/content.ts
export interface ContentPipeline {
  keyword: string;
  language: 'en' | 'vi' | 'both';
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  sources: ResearchSource[];
  generateVideo: boolean;
}

export interface ResearchSource {
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  articles: Article[];
  insights: string[];
}

export interface Article {
  title: string;
  url: string;
  publishedAt: Date;
  summary: string;
  relevanceScore: number;
}
```

## Key Features and Usage

### 1. Research & Data Crawling

```typescript
// src/lib/research/crawler.ts
import { ResearchCrawler } from '@/lib/research/crawler';

async function fetchLatestTrends(keyword: string) {
  const crawler = new ResearchCrawler({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeWindow: '24h',
    maxResults: 20
  });

  const research = await crawler.scan(keyword);
  
  return {
    articles: research.articles,
    insights: research.extractInsights(),
    trendingTopics: research.getTrendingTopics()
  };
}

// Usage
const trends = await fetchLatestTrends('AI marketing automation');
console.log(trends.insights);
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import { ContentGenerator } from '@/lib/ai/content-generator';

async function generateContent(pipeline: ContentPipeline) {
  const generator = new ContentGenerator({
    provider: 'claude', // or 'openai'
    model: 'claude-3-sonnet-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  // Generate content based on research
  const content = await generator.create({
    keyword: pipeline.keyword,
    format: pipeline.format,
    tone: pipeline.tone,
    language: pipeline.language,
    researchData: pipeline.sources
  });

  return content;
}

// Multi-language generation
async function generateBilingual(keyword: string, research: ResearchSource[]) {
  const englishContent = await generateContent({
    keyword,
    language: 'en',
    format: 'toplist',
    tone: 'expert',
    sources: research,
    generateVideo: false
  });

  const vietnameseContent = await generateContent({
    keyword,
    language: 'vi',
    format: 'toplist',
    tone: 'friendly',
    sources: research,
    generateVideo: false
  });

  return { en: englishContent, vi: vietnameseContent };
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: GeneratedContent) {
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: 'modern',
      aspectRatio: '9:16' // For Reels/TikTok
    }
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'public/videos', `${content.id}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps
  });

  return outputPath;
}
```

### 4. Complete Pipeline Execution

```typescript
// src/lib/pipeline/executor.ts
import { PipelineExecutor } from '@/lib/pipeline/executor';

async function runFullPipeline(keyword: string) {
  const executor = new PipelineExecutor();

  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await executor.research(keyword);

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await executor.generateContent({
    keyword,
    language: 'both',
    format: 'toplist',
    tone: 'expert',
    sources: research,
    generateVideo: true
  });

  // Step 3: Render video
  if (content.generateVideo) {
    console.log('🎬 Rendering video...');
    const videoPath = await executor.renderVideo(content);
    content.videoUrl = videoPath;
  }

  // Step 4: Schedule publishing
  console.log('📅 Scheduling posts...');
  await executor.schedulePublishing({
    content,
    platforms: ['facebook', 'linkedin', 'twitter'],
    scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
  });

  return content;
}

// Execute pipeline
const result = await runFullPipeline('AI content automation trends 2026');
```

## API Routes

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PipelineExecutor } from '@/lib/pipeline/executor';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, language, format, tone, generateVideo } = body;

    const executor = new PipelineExecutor();
    
    const result = await executor.runFullPipeline({
      keyword,
      language: language || 'both',
      format: format || 'toplist',
      tone: tone || 'expert',
      generateVideo: generateVideo !== false
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

### Research Only Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchCrawler } from '@/lib/research/crawler';

export async function GET(request: NextRequest) {
  const keyword = request.nextUrl.searchParams.get('keyword');
  
  if (!keyword) {
    return NextResponse.json({
      error: 'Keyword parameter required'
    }, { status: 400 });
  }

  const crawler = new ResearchCrawler({
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeWindow: '24h'
  });

  const research = await crawler.scan(keyword);

  return NextResponse.json({
    keyword,
    articles: research.articles,
    insights: research.insights,
    timestamp: new Date()
  });
}
```

## Configuration Patterns

### Custom Content Format Templates

```typescript
// src/config/formats.ts
export const contentFormats = {
  toplist: {
    structure: 'numbered-list',
    minPoints: 5,
    maxPoints: 10,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    structure: 'opinion-based',
    perspective: 'first-person',
    includeCounterarguments: true
  },
  'case-study': {
    structure: 'problem-solution',
    sections: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true
  },
  'how-to': {
    structure: 'step-by-step',
    minSteps: 3,
    maxSteps: 12,
    includePrerequisites: true
  }
};
```

### Multi-AI Provider Setup

```typescript
// src/lib/ai/provider-config.ts
export const aiProviders = {
  claude: {
    model: 'claude-3-sonnet-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY,
    maxTokens: 4096,
    temperature: 0.7,
    bestFor: ['long-form', 'analytical', 'multilingual']
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    apiKey: process.env.OPENAI_API_KEY,
    maxTokens: 4096,
    temperature: 0.8,
    bestFor: ['creative', 'concise', 'technical']
  }
};

// Automatic provider selection
function selectBestProvider(format: string) {
  if (['case-study', 'pov'].includes(format)) {
    return 'claude';
  }
  return 'openai';
}
```

## Common Workflows

### Workflow 1: Daily Trending Content

```typescript
// src/workflows/daily-trending.ts
import cron from 'node-cron';

// Run every day at 8 AM
cron.schedule('0 8 * * *', async () => {
  const trendingKeywords = await getTrendingKeywords();
  
  for (const keyword of trendingKeywords) {
    await runFullPipeline(keyword);
  }
});

async function getTrendingKeywords() {
  // Implement trending topic detection
  const crawler = new ResearchCrawler({
    sources: ['techcrunch', 'twitter'],
    timeWindow: '24h'
  });
  
  return crawler.getTrendingKeywords({ limit: 5 });
}
```

### Workflow 2: Custom Batch Generation

```typescript
// src/workflows/batch-content.ts
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runFullPipeline(keyword).catch(err => ({
        keyword,
        error: err.message
      }))
    )
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  return {
    total: keywords.length,
    successful: successful.length,
    failed: failed.length,
    results
  };
}
```

## Remotion Video Composition

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  style: 'modern' | 'minimal' | 'bold';
  aspectRatio: '16:9' | '9:16' | '1:1';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  style,
  aspectRatio
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill className={`bg-gradient-to-br from-blue-600 to-purple-700 ${style}`}>
      <div className="flex flex-col items-center justify-center h-full p-8">
        <h1 
          className="text-4xl font-bold text-white text-center mb-8"
          style={{ opacity: titleOpacity }}
        >
          {title}
        </h1>
        
        <ul className="space-y-4">
          {points.map((point, index) => {
            const pointOpacity = interpolate(
              frame,
              [60 + index * 30, 90 + index * 30],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <li 
                key={index}
                className="text-xl text-white"
                style={{ opacity: pointOpacity }}
              >
                {index + 1}. {point}
              </li>
            );
          })}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limiting

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await aiProvider.generate(prompt);
    } catch (error) {
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Memory Issues

```typescript
// Adjust Remotion concurrency
process.env.REMOTION_RENDERER_CONCURRENCY = '2'; // Lower for limited memory

// Use streaming for large videos
import { renderMedia } from '@remotion/renderer';

await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  chromiumOptions: {
    headless: true,
    args: ['--disable-dev-shm-usage', '--no-sandbox']
  }
});
```

### Research Crawler Blocked

```typescript
// Implement user agent rotation and delays
const crawler = new ResearchCrawler({
  sources: ['techcrunch'],
  userAgent: 'Mozilla/5.0 (compatible; ContentBot/1.0)',
  requestDelay: 2000, // 2 seconds between requests
  useProxy: process.env.PROXY_URL // Optional proxy
});
```

### Multilingual Content Quality

```typescript
// Use language-specific prompts
const prompts = {
  vi: `Viết bài phong cách ${tone} về ${keyword}, tập trung vào thị trường Việt Nam`,
  en: `Write ${tone} content about ${keyword} for international audience`
};

const content = await generator.create({
  prompt: prompts[language],
  model: language === 'vi' ? 'claude-3-sonnet-20240229' : 'gpt-4-turbo'
});
```

This skill provides comprehensive coverage for working with the Marketing Pipeline Share project, enabling AI agents to assist developers in automating content creation from research through video generation.
