---
name: ultimate-ai-content-pipeline
description: Automated content pipeline with AI research, multi-format article generation, and video rendering using Claude/OpenAI and Remotion
triggers:
  - generate automated content with AI research
  - create video content from articles automatically
  - build content pipeline with Claude and OpenAI
  - automate content creation from research to video
  - set up AI content generation workflow
  - render videos from written content
  - crawl news and generate articles
  - create multi-format content with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow from research/crawling to article generation and video rendering.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** latest news from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
- **Supports bilingual** output (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire pipeline

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
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Content Sources Configuration
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key

# Remotion Configuration (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Content crawling logic
│   │   ├── generators/  # Content format generators
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key API Patterns

### 1. Content Research & Crawling

```typescript
// src/lib/crawler/news-crawler.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  summary: string;
  source: string;
}

export async function crawlRecentNews(
  topic: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    const response = await axios.get(
      `https://api.rapidapi.com/news/${source}`,
      {
        params: { q: topic, timeframe: '24h' },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        },
      }
    );
    
    articles.push(...response.data.articles);
  }
  
  return articles;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article',
    'pov': 'Write from a personal perspective',
    'case-study': 'Analyze with data and examples',
    'how-to': 'Provide step-by-step instructions',
  };
  
  return `
You are a professional content writer. 
Topic: ${request.topic}
Format: ${formatInstructions[request.format]}
Language: ${request.language === 'en' ? 'English' : 'Vietnamese'}
Tone: ${request.tone}

Research Data:
${request.researchData}

Write a comprehensive article based on the research data above.
Include actionable insights and real examples.
  `.trim();
}
```

### 3. OpenAI Alternative Generator

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithGPT4(
  topic: string,
  context: string
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and technology.',
      },
      {
        role: 'user',
        content: `Topic: ${topic}\n\nContext:\n${context}\n\nGenerate a comprehensive article.`,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlRecentNews } from '@/lib/crawler/news-crawler';
import { generateContent } from '@/lib/ai/claude-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { topic, format, language, tone } = body;
    
    // Step 1: Crawl research data
    const articles = await crawlRecentNews(topic);
    const researchData = articles
      .map(a => `${a.title}\n${a.summary}`)
      .join('\n\n');
    
    // Step 2: Generate content
    const content = await generateContent({
      topic,
      format,
      language,
      tone,
      researchData,
    });
    
    return NextResponse.json({ 
      success: true, 
      content,
      sources: articles.map(a => a.url),
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: (error as Error).message },
      { status: 500 }
    );
  }
}
```

### 5. Video Rendering with Remotion

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoContent {
  title: string;
  points: string[];
  brandColor: string;
}

export async function renderContentVideo(
  content: VideoContent,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: content,
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content,
  });
  
  return outputPath;
}
```

### 6. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  brandColor,
}) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
          }}
        >
          <h1
            style={{
              color: brandColor,
              fontSize: 72,
              fontWeight: 'bold',
              opacity: Math.min(1, frame / 30),
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
            }}
          >
            <p
              style={{
                color: 'white',
                fontSize: 48,
                textAlign: 'center',
              }}
            >
              {point}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 7. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlRecentNews } from '../crawler/news-crawler';
import { generateContent } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render-video';

interface PipelineConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
}

interface PipelineResult {
  article: string;
  videoPath?: string;
  sources: string[];
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  // Step 1: Research
  console.log('🔍 Crawling research data...');
  const articles = await crawlRecentNews(config.topic);
  const researchData = articles
    .map(a => `${a.title}\n${a.summary}`)
    .join('\n\n');
  
  // Step 2: Generate article
  console.log('✍️ Generating content...');
  const article = await generateContent({
    topic: config.topic,
    format: config.format,
    language: config.language,
    tone: config.tone,
    researchData,
  });
  
  // Step 3: Extract key points for video
  let videoPath: string | undefined;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    const points = extractKeyPoints(article);
    videoPath = await renderContentVideo(
      {
        title: config.topic,
        points,
        brandColor: '#FF6B6B',
      },
      `./output/${config.topic.replace(/\s+/g, '-')}.mp4`
    );
  }
  
  return {
    article,
    videoPath,
    sources: articles.map(a => a.url),
  };
}

function extractKeyPoints(article: string): string[] {
  // Simple extraction - you can enhance with AI
  const lines = article.split('\n');
  return lines
    .filter(line => line.match(/^\d+\.|^-|^•/))
    .slice(0, 5)
    .map(line => line.replace(/^\d+\.|^-|^•/, '').trim());
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render a single video (if you have a standalone script)
npm run render-video
```

## Common Usage Patterns

### Pattern 1: Simple Article Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

const result = await runContentPipeline({
  topic: 'AI in Marketing 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  generateVideo: false,
});

console.log(result.article);
```

### Pattern 2: Full Pipeline with Video

```typescript
const result = await runContentPipeline({
  topic: 'Top 10 Marketing Trends',
  format: 'toplist',
  language: 'vi',
  tone: 'friendly',
  generateVideo: true,
});

console.log('Article:', result.article);
console.log('Video:', result.videoPath);
console.log('Sources:', result.sources);
```

### Pattern 3: Batch Content Generation

```typescript
const topics = [
  'AI Marketing Tools',
  'Social Media Trends 2024',
  'Content Marketing Best Practices',
];

const results = await Promise.all(
  topics.map(topic =>
    runContentPipeline({
      topic,
      format: 'how-to',
      language: 'en',
      tone: 'expert',
      generateVideo: true,
    })
  )
);

results.forEach((result, i) => {
  console.log(`Topic ${i + 1}: ${topics[i]}`);
  console.log(`Article length: ${result.article.length}`);
  console.log(`Video: ${result.videoPath || 'N/A'}`);
});
```

## Configuration Options

### Content Formats

- `toplist` - Numbered list articles (Top 10, Top 5, etc.)
- `pov` - Point of view / opinion pieces
- `case-study` - Data-driven analysis with examples
- `how-to` - Step-by-step instructional content

### Supported Languages

- `en` - English
- `vi` - Vietnamese (Tiếng Việt)

### Tone Options

- `expert` - Professional, authoritative
- `friendly` - Conversational, approachable
- `humorous` - Entertaining, lighthearted

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(request: ContentRequest, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request);
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

For large video renders:

```typescript
// Reduce composition complexity or split into chunks
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: compositionId,
  inputProps: {
    ...content,
    quality: 'medium', // Use 'medium' instead of 'high'
  },
});
```

### Missing Environment Variables

```typescript
// Add validation at app startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

### Crawling Blocked or Failing

Implement fallback sources:

```typescript
export async function crawlRecentNews(
  topic: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(/* ... */);
      articles.push(...response.data.articles);
    } catch (error) {
      console.warn(`Failed to crawl ${source}, continuing...`);
      // Continue with other sources
    }
  }
  
  if (articles.length === 0) {
    throw new Error('All crawl sources failed');
  }
  
  return articles;
}
```

## Advanced Tips

### Streaming Content Generation

For real-time updates in the UI:

```typescript
export async function* generateContentStream(
  request: ContentRequest
): AsyncGenerator<string> {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: buildPrompt(request) }],
  });
  
  for await (const chunk of stream) {
    if (
      chunk.type === 'content_block_delta' &&
      chunk.delta.type === 'text_delta'
    ) {
      yield chunk.delta.text;
    }
  }
}
```

### Caching Research Data

To avoid redundant crawls:

```typescript
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

export async function getCachedNews(topic: string) {
  const cacheKey = `news:${topic}`;
  const cached = cache.get<NewsArticle[]>(cacheKey);
  
  if (cached) {
    return cached;
  }
  
  const articles = await crawlRecentNews(topic);
  cache.set(cacheKey, articles);
  return articles;
}
```

This skill enables comprehensive automation of content creation from research to publication-ready articles and videos using state-of-the-art AI models and video rendering technology.
