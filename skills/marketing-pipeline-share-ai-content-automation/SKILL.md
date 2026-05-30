---
name: marketing-pipeline-share-ai-content-automation
description: AI-powered content pipeline for automated research, scriptwriting, video generation, and multi-platform publishing using Claude and OpenAI
triggers:
  - automate content creation with AI research
  - generate videos from articles automatically
  - create multi-language marketing content
  - set up AI content pipeline with remotion
  - scrape news and generate social media posts
  - build automated content workflow with Claude
  - generate infographics and reels from text
  - create content pipeline from research to video
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers build and use an end-to-end automated content pipeline that handles research, scriptwriting, multi-language article generation, and video rendering using Claude AI, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share is a comprehensive content automation system that:

- **Auto-crawls** trending news from TechCrunch, a16z, Twitter/X, LinkedIn within 24 hours
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports bilingual output** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for Reels, TikTok, and Shorts
- **Provides a Next.js interface** for easy content management and scheduling

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Key Components and Architecture

### 1. Research Module (Auto-Scan)

The research module crawls and aggregates content from multiple sources:

```typescript
// src/services/research/crawler.ts
import { RapidAPI } from '@/lib/rapidapi';

interface ResearchSource {
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeframe: '24h' | '7d' | '30d';
  keywords: string[];
}

export async function crawlNews(source: ResearchSource) {
  const api = new RapidAPI(process.env.RAPIDAPI_KEY!);
  
  const results = await api.search({
    platform: source.platform,
    query: source.keywords.join(' OR '),
    timeRange: source.timeframe,
    limit: 50
  });
  
  return results.map(article => ({
    title: article.title,
    url: article.url,
    publishedAt: article.publishedAt,
    summary: article.excerpt,
    insights: extractInsights(article.content)
  }));
}

function extractInsights(content: string): string[] {
  // AI-powered insight extraction
  const insights = [];
  // Data points, statistics, quotes extraction logic
  return insights;
}
```

### 2. Content Generation with AI

Generate articles in multiple formats using Claude or OpenAI:

```typescript
// src/services/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  keywords: string[];
  researchData: any[];
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return {
    content: message.content[0].text,
    metadata: {
      model: 'claude-3-5-sonnet',
      format: config.format,
      language: config.language
    }
  };
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with 5-10 items',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Analyze a real example with data and outcomes',
    'how-to': 'Provide step-by-step actionable instructions'
  };
  
  return `
You are an expert ${config.tone} content writer.

Format: ${formatInstructions[config.format]}
Language: ${config.language === 'en' ? 'English' : 'Vietnamese'}
Keywords: ${config.keywords.join(', ')}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Create a comprehensive article based on this research. Include:
- Attention-grabbing headline
- Data-backed insights
- Practical takeaways
- SEO-optimized structure
`;
}
```

### 3. Bilingual Content Support

Generate content in both English and Vietnamese simultaneously:

```typescript
// src/services/content/bilingual.ts
export async function generateBilingualContent(
  topic: string,
  researchData: any[]
) {
  const [enContent, viContent] = await Promise.all([
    generateContent({
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      keywords: [topic],
      researchData
    }),
    generateContent({
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      keywords: [topic],
      researchData
    })
  ]);
  
  return {
    english: enContent,
    vietnamese: viContent,
    syncedAt: new Date().toISOString()
  };
}
```

### 4. Video Generation with Remotion

Automatically render videos from article content:

```typescript
// src/services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';
import { getCompositions } from '@remotion/renderer';

interface VideoConfig {
  title: string;
  content: string[];
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function renderVideo(config: VideoConfig) {
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  const compositions = await getCompositions(bundleLocation);
  const composition = compositions.find(c => c.id === 'ContentVideo');
  
  if (!composition) {
    throw new Error('Video composition not found');
  }
  
  const dimensions = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'shorts': { width: 1080, height: 1920 }
  };
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${config.title}.mp4`,
    inputProps: {
      title: config.title,
      slides: config.content,
      duration: config.duration
    },
    ...dimensions[config.format]
  });
  
  return `out/${config.title}.mp4`;
}
```

### 5. Remotion Video Composition

Create a basic video composition for content:

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  slides: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  slides,
  duration
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const framesPerSlide = Math.floor((duration * fps) / slides.length);
  const currentSlideIndex = Math.min(
    Math.floor(frame / framesPerSlide),
    slides.length - 1
  );
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60
      }}
    >
      <h1
        style={{
          fontSize: 80,
          color: '#fff',
          textAlign: 'center',
          marginBottom: 40
        }}
      >
        {title}
      </h1>
      <p
        style={{
          fontSize: 48,
          color: '#fff',
          textAlign: 'center',
          lineHeight: 1.6
        }}
      >
        {slides[currentSlideIndex]}
      </p>
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

Here's a full end-to-end pipeline execution:

```typescript
// src/workflows/complete-pipeline.ts
import { crawlNews } from '@/services/research/crawler';
import { generateBilingualContent } from '@/services/content/bilingual';
import { renderVideo } from '@/services/video/renderer';

export async function runCompletePipeline(keyword: string) {
  console.log(`🚀 Starting pipeline for: ${keyword}`);
  
  // Step 1: Research
  console.log('📡 Crawling news sources...');
  const [techcrunch, a16z, twitter] = await Promise.all([
    crawlNews({
      platform: 'techcrunch',
      timeframe: '24h',
      keywords: [keyword]
    }),
    crawlNews({
      platform: 'a16z',
      timeframe: '24h',
      keywords: [keyword]
    }),
    crawlNews({
      platform: 'twitter',
      timeframe: '24h',
      keywords: [keyword]
    })
  ]);
  
  const researchData = [...techcrunch, ...a16z, ...twitter];
  console.log(`✅ Found ${researchData.length} articles`);
  
  // Step 2: Generate Content
  console.log('🧠 Generating bilingual content...');
  const content = await generateBilingualContent(keyword, researchData);
  console.log('✅ Content generated in both languages');
  
  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(content.english.content);
  
  // Step 4: Render Videos
  console.log('🎬 Rendering videos...');
  const [reelsVideo, tiktokVideo] = await Promise.all([
    renderVideo({
      title: keyword,
      content: keyPoints,
      format: 'reels',
      duration: 30
    }),
    renderVideo({
      title: keyword,
      content: keyPoints,
      format: 'tiktok',
      duration: 30
    })
  ]);
  
  console.log('✅ Videos rendered successfully');
  
  return {
    research: researchData,
    content,
    videos: {
      reels: reelsVideo,
      tiktok: tiktokVideo
    },
    completedAt: new Date().toISOString()
  };
}

function extractKeyPoints(content: string): string[] {
  // Extract 5-7 key points from content for video slides
  const sentences = content.split(/[.!?]+/).filter(s => s.trim().length > 0);
  return sentences.slice(0, 7).map(s => s.trim());
}
```

## API Routes (Next.js)

Create API endpoints for the pipeline:

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runCompletePipeline } from '@/workflows/complete-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runCompletePipeline(keyword);
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## CLI Usage

If the project includes CLI tools:

```bash
# Run complete pipeline
npm run pipeline -- --keyword "AI automation" --format toplist

# Research only
npm run research -- --platform techcrunch --days 7

# Generate content from existing research
npm run generate -- --input research.json --language vi

# Render video
npm run render -- --input content.json --format reels
```

## Configuration File

Create a `pipeline.config.ts` for customization:

```typescript
// pipeline.config.ts
export const config = {
  research: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxArticles: 50
  },
  content: {
    defaultFormat: 'toplist',
    defaultTone: 'expert',
    languages: ['en', 'vi'],
    minWordCount: 800
  },
  video: {
    defaultDuration: 30,
    formats: ['reels', 'tiktok', 'shorts'],
    outputDir: './out/videos'
  },
  ai: {
    provider: 'claude', // or 'openai'
    model: 'claude-3-5-sonnet-20241022',
    temperature: 0.7
  }
};
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// src/schedulers/content-scheduler.ts
import cron from 'node-cron';
import { runCompletePipeline } from '@/workflows/complete-pipeline';

const topics = ['AI', 'Marketing', 'SaaS', 'Startup'];

// Run every day at 8 AM
cron.schedule('0 8 * * *', async () => {
  const todayTopic = topics[Math.floor(Math.random() * topics.length)];
  
  try {
    await runCompletePipeline(todayTopic);
    console.log(`✅ Daily content generated for: ${todayTopic}`);
  } catch (error) {
    console.error('Scheduled pipeline failed:', error);
  }
});
```

### Pattern 2: Content Quality Validation

```typescript
// src/validators/content-validator.ts
export function validateContent(content: string): boolean {
  const wordCount = content.split(/\s+/).length;
  const hasHeadings = /#{1,3}\s/.test(content);
  const hasDataPoints = /\d+%|\$\d+/.test(content);
  
  return wordCount >= 800 && hasHeadings && hasDataPoints;
}

export async function generateValidContent(config: ContentConfig) {
  let attempts = 0;
  const maxAttempts = 3;
  
  while (attempts < maxAttempts) {
    const content = await generateContent(config);
    
    if (validateContent(content.content)) {
      return content;
    }
    
    attempts++;
    console.log(`Regenerating content (attempt ${attempts}/${maxAttempts})`);
  }
  
  throw new Error('Failed to generate valid content after max attempts');
}
```

### Pattern 3: Multi-Platform Publishing

```typescript
// src/publishers/social-publisher.ts
interface PublishConfig {
  content: string;
  videoPath: string;
  platforms: ('facebook' | 'instagram' | 'tiktok')[];
}

export async function publishToSocial(config: PublishConfig) {
  const results = await Promise.allSettled(
    config.platforms.map(async platform => {
      switch (platform) {
        case 'facebook':
          return await publishToFacebook(config.content, config.videoPath);
        case 'instagram':
          return await publishToInstagram(config.content, config.videoPath);
        case 'tiktok':
          return await publishToTikTok(config.content, config.videoPath);
      }
    })
  );
  
  return results.map((result, index) => ({
    platform: config.platforms[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// src/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function batchProcess<T>(
  items: T[],
  processor: (item: T) => Promise<any>
) {
  return await Promise.all(
    items.map(item => limit(() => processor(item)))
  );
}
```

### Issue: Video Rendering Fails

```bash
# Check Remotion installation
npx remotion versions

# Test basic rendering
npx remotion render src/remotion/index.ts ContentVideo out/test.mp4

# Increase memory if needed
NODE_OPTIONS="--max-old-space-size=4096" npm run render
```

### Issue: AI Content Quality

Adjust prompts and temperature:

```typescript
const message = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  temperature: 0.7, // Lower for more focused content
  messages: [{
    role: 'user',
    content: prompt
  }],
  system: 'You are an expert content writer with deep knowledge in marketing and technology. Always provide data-backed insights and actionable advice.'
});
```

### Issue: Research Data Quality

Filter and deduplicate results:

```typescript
function deduplicateArticles(articles: any[]) {
  const seen = new Set();
  return articles.filter(article => {
    const key = `${article.title}-${article.url}`;
    if (seen.has(key)) return false;
    seen.add(key);
    return true;
  });
}
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';
const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

export async function getCachedResearch(keyword: string) {
  const cached = cache.get(keyword);
  if (cached) return cached;
  
  const fresh = await crawlNews({ keywords: [keyword] });
  cache.set(keyword, fresh);
  return fresh;
}
```

This skill provides comprehensive guidance for building and using an automated AI content pipeline with research, generation, and video rendering capabilities.
