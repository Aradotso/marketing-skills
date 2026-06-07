---
name: ultimate-ai-content-pipeline
description: Automated Vietnamese/English content research, scriptwriting, and video generation pipeline using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline for automated research
  - generate Vietnamese and English marketing content automatically
  - create video content from text using remotion integration
  - crawl news sources and generate blog posts with AI
  - automate content creation workflow from research to video
  - set up Claude or OpenAI for multilingual content generation
  - build automated marketing content pipeline with AI
  - generate social media videos from articles automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript project provides an end-to-end automated content creation system that handles research (crawling news sources), content generation (using Claude/OpenAI), and video rendering (using Remotion). It's designed for marketers and content creators who need to produce high-quality Vietnamese and English content at scale.

## What It Does

- **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Multilingual Output**: Generates parallel Vietnamese and English versions
- **Video Rendering**: Automatically converts content to video/infographics using Remotion
- **Social Media Ready**: Outputs optimized for Reels, TikTok, Shorts

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
# AI Provider Keys (choose one or both)
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research & Crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_here

# Application Settings
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawlers/    # News source crawlers
│   │   ├── generators/  # Content generators
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key Usage Patterns

### 1. Basic Content Generation

```typescript
import { generateContent } from '@/lib/generators/content';
import { AnthropicProvider } from '@/lib/ai/anthropic';

const aiProvider = new AnthropicProvider(process.env.ANTHROPIC_API_KEY);

const content = await generateContent({
  keyword: 'AI automation',
  format: 'toplist',
  languages: ['vi', 'en'],
  tone: 'professional',
  aiProvider
});

console.log(content.vietnamese);
console.log(content.english);
```

### 2. Research & Data Crawling

```typescript
import { NewsResearcher } from '@/lib/crawlers/researcher';

const researcher = new NewsResearcher({
  rapidApiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter']
});

const insights = await researcher.research({
  topic: 'AI automation',
  timeframe: '24h',
  maxResults: 10
});

// insights contains: { title, url, summary, sentiment, source, date }
```

### 3. Full Pipeline Execution

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY
});

const result = await pipeline.execute({
  keyword: 'marketing automation',
  format: 'case-study',
  includeVideo: true,
  videoFormat: 'reels' // or 'tiktok', 'shorts'
});

// result contains:
// - article: { vi: string, en: string }
// - research: Array<ResearchData>
// - video: { url: string, thumbnail: string }
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/remotion/templates/article';

const videoPath = await renderVideo({
  composition: VideoTemplate,
  props: {
    title: content.title,
    sections: content.sections,
    duration: 60, // seconds
    style: 'modern'
  },
  outputFormat: 'mp4',
  dimensions: { width: 1080, height: 1920 } // 9:16 for Reels/TikTok
});
```

### 5. API Routes for Next.js Integration

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  const { keyword, format, languages } = await request.json();
  
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  try {
    const content = await pipeline.execute({
      keyword,
      format,
      languages
    });
    
    return NextResponse.json(content);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Content Formats

### Available Formats

```typescript
type ContentFormat = 
  | 'toplist'      // Top 10 lists with bullet points
  | 'pov'          // Opinion/perspective piece
  | 'case-study'   // In-depth analysis with examples
  | 'how-to'       // Step-by-step guide
  | 'news'         // News article style
  | 'comparison';  // Product/service comparison

const formatConfig = {
  toplist: {
    structure: 'numbered-list',
    sections: ['intro', 'items', 'conclusion'],
    minItems: 5
  },
  'case-study': {
    structure: 'narrative',
    sections: ['challenge', 'solution', 'results', 'takeaways'],
    includeData: true
  }
};
```

### Tone Customization

```typescript
import { generateContent } from '@/lib/generators/content';

const content = await generateContent({
  keyword: 'social media marketing',
  format: 'how-to',
  tone: 'friendly', // 'professional' | 'friendly' | 'humorous' | 'authoritative'
  targetAudience: 'small-business-owners',
  languages: ['vi', 'en']
});
```

## AI Provider Integration

### Using Claude (Anthropic)

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string) {
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

### Using OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'user',
      content: prompt
    }],
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content;
}
```

## Video Rendering Configuration

### Remotion Template Setup

```typescript
// remotion/templates/article.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const VideoTemplate: React.FC<{
  title: string;
  sections: Array<{ heading: string; content: string }>;
  duration: number;
}> = ({ title, sections, duration }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <TitleScene title={title} />
      </Sequence>
      {sections.map((section, i) => (
        <Sequence
          key={i}
          from={60 + i * 120}
          durationInFrames={120}
        >
          <ContentScene {...section} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### Render Settings

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function renderContentVideo(articleData: any) {
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ArticleVideo',
    inputProps: articleData
  });
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${articleData.title}.mp4`,
    imageFormat: 'jpeg'
  });
}
```

## Running the Application

### Development Server

```bash
npm run dev
# Runs on http://localhost:3000
```

### Build for Production

```bash
npm run build
npm start
```

### Video Rendering Server

```bash
# Start Remotion preview
npx remotion studio remotion/index.ts

# Render specific composition
npx remotion render remotion/index.ts ArticleVideo out/video.mp4
```

## Common Workflows

### Workflow 1: Daily News Digest

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function createDailyDigest() {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  const topics = ['AI', 'marketing', 'automation'];
  
  for (const topic of topics) {
    const content = await pipeline.execute({
      keyword: topic,
      format: 'news',
      languages: ['vi', 'en'],
      includeVideo: false // Just articles
    });
    
    // Save to database or publish
    await publishContent(content);
  }
}
```

### Workflow 2: Social Media Content Series

```typescript
async function createContentSeries(mainTopic: string) {
  const formats = ['toplist', 'how-to', 'case-study'];
  const results = [];
  
  for (const format of formats) {
    const content = await pipeline.execute({
      keyword: mainTopic,
      format,
      languages: ['vi'],
      includeVideo: true,
      videoFormat: 'reels'
    });
    
    results.push({
      day: formats.indexOf(format) + 1,
      article: content.article,
      videoUrl: content.video.url
    });
  }
  
  return results; // 3-day content calendar
}
```

### Workflow 3: Multilingual Blog Post

```typescript
async function createMultilingualPost(keyword: string) {
  const researcher = new NewsResearcher({
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  // 1. Research phase
  const research = await researcher.research({
    topic: keyword,
    timeframe: '24h'
  });
  
  // 2. Generate content in both languages
  const content = await generateContent({
    keyword,
    format: 'case-study',
    languages: ['vi', 'en'],
    researchData: research,
    aiProvider: new AnthropicProvider(process.env.ANTHROPIC_API_KEY)
  });
  
  // 3. Create accompanying visuals
  const thumbnail = await generateThumbnail(content.vietnamese.title);
  const infographic = await generateInfographic(content.vietnamese.keyPoints);
  
  return {
    vietnamese: content.vietnamese,
    english: content.english,
    assets: { thumbnail, infographic }
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateWithClaude(prompt);
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
// Use smaller compositions or split rendering
const renderConfig = {
  concurrency: 1, // Reduce parallel rendering
  envVariables: {
    NODE_OPTIONS: '--max-old-space-size=4096'
  },
  chromiumOptions: {
    headless: true,
    gl: 'swiftshader' // Use software renderer if GPU issues
  }
};
```

### Crawler Timeout Issues

```typescript
const researcher = new NewsResearcher({
  rapidApiKey: process.env.RAPIDAPI_KEY,
  timeout: 30000, // 30 seconds
  retries: 3,
  fallbackSources: ['rss-feeds'] // Fallback if API fails
});
```

### Language Detection Errors

```typescript
import { detectLanguage } from '@/lib/utils/language';

// Ensure proper language handling
function sanitizeContent(text: string, targetLang: 'vi' | 'en') {
  const detected = detectLanguage(text);
  
  if (detected !== targetLang) {
    console.warn(`Language mismatch: expected ${targetLang}, got ${detected}`);
    // Re-generate or translate
  }
  
  return text;
}
```

## Performance Optimization

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.REDIS_URL,
  token: process.env.REDIS_TOKEN
});

async function getCachedResearch(topic: string) {
  const cacheKey = `research:${topic}:${new Date().toDateString()}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) return cached;
  
  const research = await researcher.research({ topic });
  await redis.set(cacheKey, research, { ex: 86400 }); // 24h cache
  
  return research;
}
```

### Parallel Content Generation

```typescript
async function generateBulkContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      pipeline.execute({ keyword, format: 'toplist' })
    )
  );
  
  return results;
}
```

This skill enables AI coding agents to help developers set up and use the Ultimate AI Content Pipeline for automated marketing content creation, from research through to video generation.
