---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - "set up automated content pipeline with AI"
  - "create content from research to video automatically"
  - "build AI-powered marketing content system"
  - "generate video content from text using Remotion"
  - "automate content research and scriptwriting"
  - "scrape news and create social media posts with AI"
  - "use Claude and OpenAI for content generation"
  - "build automated content workflow with video rendering"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based automation system for content creation that handles research, scriptwriting, multi-format content generation, and video rendering. The pipeline crawls news sources, generates content in multiple languages and formats using Claude/OpenAI, and renders videos with Remotion.

## What It Does

- **Auto Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for latest news (24h)
- **AI Content Generation**: Creates content in multiple formats (Top Lists, POV, Case Studies, How-To)
- **Multi-language Support**: Generates Vietnamese and English versions simultaneously
- **Video Rendering**: Converts text content to infographics and short videos using Remotion
- **Multi-platform Output**: Exports video in formats optimized for Reels, TikTok, Shorts

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database/Storage
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { researchTopic } from '@/lib/research/crawler';

async function fetchLatestNews(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 10
  });
  
  return research.articles.map(article => ({
    title: article.title,
    url: article.url,
    summary: article.summary,
    publishedAt: article.publishedAt,
    insights: article.keyInsights
  }));
}
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
  language: 'vi' | 'en'
) {
  const prompt = `
You are a content strategist. Create a ${format} article about: ${topic}
Language: ${language}
Include: Introduction, main points with data, conclusion
Tone: Professional yet engaging
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2048,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(
  researchData: any[],
  contentType: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content creator.'
      },
      {
        role: 'user',
        content: `Create a ${contentType} based on this research: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';
import path from 'path';

async function renderContentVideo(
  content: {
    title: string;
    points: string[];
    stats: Array<{ label: string; value: string }>;
  },
  format: 'reels' | 'tiktok' | 'shorts' = 'reels'
) {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: compositionId,
    inputProps: {
      title: content.title,
      points: content.points,
      stats: content.stats,
      format,
    },
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'public/videos', `output-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps,
  });

  return outputPath;
}
```

### 5. Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/render';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
      limit: 5
    });

    // Step 2: Generate content (both languages)
    console.log('✍️ Generating content...');
    const [contentVI, contentEN] = await Promise.all([
      generateContentWithClaude(keyword, 'toplist', 'vi'),
      generateContentWithClaude(keyword, 'toplist', 'en')
    ]);

    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(contentEN); // Your extraction logic

    // Step 4: Render video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: keyword,
      points: keyPoints.slice(0, 5),
      stats: research.articles.slice(0, 3).map(a => ({
        label: a.source,
        value: a.engagement || '0'
      }))
    }, 'reels');

    return {
      research,
      content: { vi: contentVI, en: contentEN },
      video: videoPath
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI Marketing Trends 2024').then(result => {
  console.log('✅ Pipeline completed!');
  console.log('Video:', result.video);
});
```

## Content Format Templates

### Top List Format

```typescript
interface TopListContent {
  title: string;
  introduction: string;
  items: Array<{
    rank: number;
    title: string;
    description: string;
    evidence: string;
  }>;
  conclusion: string;
}

async function generateTopList(topic: string, count: number = 5) {
  const prompt = `Create a top ${count} list about ${topic}. 
  Format: JSON with title, introduction, items (rank, title, description, evidence), conclusion`;
  
  const content = await generateContentWithClaude(topic, 'toplist', 'en');
  return JSON.parse(content) as TopListContent;
}
```

### POV (Point of View) Format

```typescript
interface POVContent {
  title: string;
  hook: string;
  perspective: string;
  arguments: Array<{
    point: string;
    support: string;
    counterpoint?: string;
  }>;
  conclusion: string;
}
```

## Remotion Video Templates

### Basic Video Composition

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <div style={{ opacity, padding: 40, color: 'white' }}>
        <h1 style={{ fontSize: 48, marginBottom: 40 }}>{title}</h1>
        {points.map((point, idx) => {
          const pointOpacity = interpolate(
            frame,
            [30 + idx * 20, 50 + idx * 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          return (
            <p key={idx} style={{ opacity: pointOpacity, fontSize: 24, marginBottom: 20 }}>
              {idx + 1}. {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

## Common Workflows

### Daily Content Automation

```typescript
// scripts/daily-content.ts
import cron from 'node-cron';

// Run every day at 6 AM
cron.schedule('0 6 * * *', async () => {
  const topics = ['AI Marketing', 'Social Media Trends', 'Content Strategy'];
  
  for (const topic of topics) {
    const result = await runContentPipeline(topic);
    
    // Save to database or publish
    await saveContent(result);
    
    console.log(`✅ Generated content for: ${topic}`);
  }
});
```

### Batch Processing

```typescript
async function processBatch(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  console.log(`✅ Success: ${successful.length}, ❌ Failed: ${failed.length}`);
  
  return successful.map(r => (r as PromiseFulfilledResult<any>).value);
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchGenerate(topics: string[]) {
  return Promise.all(
    topics.map(topic => 
      limit(() => generateContentWithClaude(topic, 'toplist', 'en'))
    )
  );
}
```

### Error Handling

```typescript
async function safeGenerate(topic: string, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await generateContentWithClaude(topic, 'toplist', 'en');
    } catch (error) {
      if (i === retries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000; // Exponential backoff
      console.log(`Retry ${i + 1}/${retries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion installation
import { getCompositions } from '@remotion/renderer';

async function validateRemotionSetup() {
  try {
    const compositions = await getCompositions(bundled);
    console.log('Available compositions:', compositions.map(c => c.id));
  } catch (error) {
    console.error('Remotion setup error:', error);
    throw new Error('Check Remotion configuration and dependencies');
  }
}
```

## Performance Optimization

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) return cached;
  
  const fresh = await researchTopic({ keyword, sources: ['techcrunch'], timeframe: '24h' });
  await redis.set(cacheKey, fresh, { ex: 3600 }); // Cache for 1 hour
  
  return fresh;
}
```

### Parallel Processing

```typescript
async function generateMultiLanguageContent(topic: string) {
  const [vi, en, video] = await Promise.all([
    generateContentWithClaude(topic, 'toplist', 'vi'),
    generateContentWithClaude(topic, 'toplist', 'en'),
    renderContentVideo({ title: topic, points: [], stats: [] })
  ]);
  
  return { vi, en, video };
}
```
