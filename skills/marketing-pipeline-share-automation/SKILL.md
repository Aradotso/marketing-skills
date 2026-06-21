---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese/English marketing content
triggers:
  - how do I automate content research and writing with AI
  - generate marketing videos from text automatically
  - create bilingual content in Vietnamese and English
  - build an AI-powered content pipeline
  - automate social media content creation with video
  - research and write blog posts using Claude API
  - set up automated content workflow with Remotion
  - crawl news and generate marketing content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use **marketing-pipeline-share**, an automated content creation pipeline that handles research, scriptwriting, and video generation. The system crawls fresh news from sources like TechCrunch, Twitter/X, and LinkedIn, then generates multi-format content (blog posts, case studies, how-tos) in both English and Vietnamese, and renders videos/infographics using Remotion.

## What It Does

- **Auto-Research**: Crawls real-time data from news sources and social media
- **AI Content Generation**: Creates diverse formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Bilingual Output**: Generates parallel English and Vietnamese content
- **Video Rendering**: Converts text to infographic videos using Remotion for Reels/TikTok/Shorts
- **Multi-Platform**: Optimized for Next.js with API integrations

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
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Social Media APIs
TWITTER_BEARER_TOKEN=your_twitter_token_here
LINKEDIN_API_KEY=your_linkedin_key_here

# Next.js Configuration
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
│   │   ├── crawlers/    # Content crawlers
│   │   ├── generators/  # Content generators
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
// src/lib/crawlers/news-crawler.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
  source: string;
}

export async function crawlTechNews(
  keyword: string,
  hours: number = 24
): Promise<NewsArticle[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const options = {
    method: 'GET',
    url: 'https://news-api14.p.rapidapi.com/v2/search/articles',
    params: {
      query: keyword,
      language: 'en',
      sortBy: 'publishedAt',
      from: new Date(Date.now() - hours * 60 * 60 * 1000).toISOString()
    },
    headers: {
      'X-RapidAPI-Key': rapidApiKey,
      'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
    }
  };

  const response = await axios.request(options);
  return response.data.articles.map((article: any) => ({
    title: article.title,
    url: article.url,
    publishedAt: article.publishedAt,
    content: article.description || article.content,
    source: article.source.name
  }));
}

// Usage example
const articles = await crawlTechNews('AI automation', 24);
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string[];
}

export async function generateContentWithClaude(
  request: ContentRequest
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const systemPrompt = buildSystemPrompt(request.format, request.language, request.tone);
  const userPrompt = buildUserPrompt(request.keyword, request.researchData);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(format: string, language: string, tone: string): string {
  const prompts = {
    'toplist': {
      'en': 'You are an expert content writer creating engaging top 10 lists...',
      'vi': 'Bạn là chuyên gia viết nội dung tạo các bài top 10 hấp dẫn...'
    },
    'case-study': {
      'en': 'You are a business analyst writing detailed case studies...',
      'vi': 'Bạn là chuyên gia phân tích viết case study chi tiết...'
    }
  };
  
  return prompts[format]?.[language] || prompts['toplist']['en'];
}

function buildUserPrompt(keyword: string, researchData: string[]): string {
  return `
Topic: ${keyword}

Research Data:
${researchData.join('\n\n')}

Create comprehensive content based on the above research. Include:
- Engaging headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure
  `.trim();
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateContentWithOpenAI(
  request: ContentRequest
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: buildSystemPrompt(request.format, request.language, request.tone)
      },
      {
        role: 'user',
        content: buildUserPrompt(request.keyword, request.researchData)
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  stats: Array<{ label: string; value: string }>;
  platform: 'tiktok' | 'reels' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
      stats: config.stats,
    },
  });

  const dimensions = getPlatformDimensions(config.platform);
  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${config.platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.props,
    ...dimensions
  });

  return outputPath;
}

function getPlatformDimensions(platform: string) {
  const dimensions = {
    'tiktok': { width: 1080, height: 1920 },   // 9:16
    'reels': { width: 1080, height: 1920 },    // 9:16
    'shorts': { width: 1080, height: 1920 },   // 9:16
    'default': { width: 1920, height: 1080 }   // 16:9
  };
  
  return dimensions[platform] || dimensions['default'];
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  stats: Array<{ label: string; value: string }>;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  keyPoints, 
  stats 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const pointsStart = 60;
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      {/* Title Section */}
      <div
        style={{
          position: 'absolute',
          top: '15%',
          left: '10%',
          right: '10%',
          opacity: titleOpacity,
        }}
      >
        <h1 style={{ 
          color: '#fff', 
          fontSize: 72, 
          fontWeight: 'bold',
          textAlign: 'center'
        }}>
          {title}
        </h1>
      </div>

      {/* Key Points */}
      <div style={{ position: 'absolute', top: '35%', left: '10%', right: '10%' }}>
        {keyPoints.map((point, index) => {
          const pointFrame = pointsStart + (index * 45);
          const opacity = interpolate(
            frame,
            [pointFrame, pointFrame + 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );

          return (
            <div
              key={index}
              style={{
                opacity,
                marginBottom: 30,
                padding: 20,
                backgroundColor: 'rgba(255,255,255,0.1)',
                borderRadius: 10,
              }}
            >
              <p style={{ color: '#fff', fontSize: 32 }}>
                {index + 1}. {point}
              </p>
            </div>
          );
        })}
      </div>

      {/* Stats Section */}
      <div style={{ 
        position: 'absolute', 
        bottom: '10%', 
        left: '10%', 
        right: '10%',
        display: 'flex',
        justifyContent: 'space-around'
      }}>
        {stats.map((stat, index) => (
          <div key={index} style={{ textAlign: 'center' }}>
            <div style={{ color: '#00ff88', fontSize: 48, fontWeight: 'bold' }}>
              {stat.value}
            </div>
            <div style={{ color: '#fff', fontSize: 24 }}>
              {stat.label}
            </div>
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Example

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlTechNews } from '../crawlers/news-crawler';
import { generateContentWithClaude } from '../ai/claude-generator';
import { renderContentVideo } from '../video/remotion-renderer';

export async function runContentPipeline(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
): Promise<{
  englishContent: string;
  vietnameseContent: string;
  videoPath: string;
}> {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const articles = await crawlTechNews(keyword, 24);
  const researchData = articles.map(a => `${a.title}\n${a.content}`);

  // Step 2: Generate English Content
  console.log('📝 Generating English content...');
  const englishContent = await generateContentWithClaude({
    keyword,
    format,
    language: 'en',
    tone: 'expert',
    researchData
  });

  // Step 3: Generate Vietnamese Content
  console.log('📝 Generating Vietnamese content...');
  const vietnameseContent = await generateContentWithClaude({
    keyword,
    format,
    language: 'vi',
    tone: 'expert',
    researchData
  });

  // Step 4: Extract key points for video
  const keyPoints = extractKeyPoints(englishContent);
  const stats = extractStats(englishContent);

  // Step 5: Render Video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo({
    title: keyword,
    keyPoints: keyPoints.slice(0, 3),
    stats,
    platform: 'reels'
  });

  console.log('✅ Pipeline complete!');
  
  return {
    englishContent,
    vietnameseContent,
    videoPath
  };
}

function extractKeyPoints(content: string): string[] {
  const lines = content.split('\n');
  const points: string[] = [];
  
  for (const line of lines) {
    if (line.match(/^[\d\-\*•]/) || line.includes('Key point')) {
      points.push(line.replace(/^[\d\-\*•]\s*/, '').trim());
    }
  }
  
  return points.filter(p => p.length > 0);
}

function extractStats(content: string): Array<{ label: string; value: string }> {
  const stats: Array<{ label: string; value: string }> = [];
  const statPattern = /(\d+%|\d+[KMB]?)\s+([a-zA-Z\s]+)/g;
  
  let match;
  while ((match = statPattern.exec(content)) !== null && stats.length < 3) {
    stats.push({
      value: match[1],
      label: match[2].trim().slice(0, 30)
    });
  }
  
  return stats;
}
```

### 7. Next.js API Route

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(
      keyword,
      format || 'toplist'
    );

    return NextResponse.json({
      success: true,
      data: result
    });

  } catch (error: any) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: error.message || 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render video only
npm run remotion:render
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple pieces of content
const topics = ['AI automation', 'Content marketing', 'SEO trends'];

const results = await Promise.all(
  topics.map(topic => 
    runContentPipeline(topic, 'toplist')
  )
);
```

### Custom Tone Adjustment

```typescript
// Adjust content tone dynamically
const tones = ['expert', 'friendly', 'humorous'] as const;

const contentVariations = await Promise.all(
  tones.map(tone =>
    generateContentWithClaude({
      keyword: 'marketing automation',
      format: 'how-to',
      language: 'en',
      tone,
      researchData
    })
  )
);
```

### Multi-Platform Video Export

```typescript
const platforms = ['tiktok', 'reels', 'shorts'] as const;

const videos = await Promise.all(
  platforms.map(platform =>
    renderContentVideo({ title, keyPoints, stats, platform })
  )
);
```

## Troubleshooting

### API Rate Limits
```typescript
// Add retry logic
async function withRetry<T>(
  fn: () => Promise<T>,
  retries = 3
): Promise<T> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === retries - 1) throw error;
      if (error.status === 429) {
        await new Promise(r => setTimeout(r, 2000 * (i + 1)));
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Timeout
```typescript
// Increase timeout in renderMedia
await renderMedia({
  // ... other options
  timeoutInMilliseconds: 120000, // 2 minutes
});
```

### Memory Issues
```typescript
// Process in batches for large datasets
async function processBatch<T>(
  items: T[],
  batchSize: number,
  processor: (item: T) => Promise<any>
) {
  const results = [];
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(batch.map(processor));
    results.push(...batchResults);
  }
  return results;
}
```

This skill provides comprehensive automation for marketing content creation, from research through publication-ready videos.
