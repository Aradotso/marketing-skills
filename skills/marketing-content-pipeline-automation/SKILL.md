---
name: marketing-content-pipeline-automation
description: Automated AI content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - build automated marketing content pipeline
  - create AI-powered content workflow from research to video
  - set up automated blog post and video generation
  - implement content automation with Claude and OpenAI
  - generate videos automatically from written content
  - automate social media content research and creation
  - build end-to-end content marketing automation
---

# Marketing Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research, script writing, and video generation. It automatically crawls fresh data from sources like TechCrunch, Twitter/X, and LinkedIn, generates content in multiple formats using Claude/OpenAI, and renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from news sources and social media
- **AI Content Generation**: Creates blog posts, case studies, how-tos in multiple languages and tones
- **Video Rendering**: Automatically converts written content into videos optimized for Reels, TikTok, Shorts
- **Multi-format Output**: Supports various content formats (toplist, POV, case study, tutorial)

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

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Components

### 1. Research Module

The research module crawls and aggregates content from various sources:

```typescript
// lib/research/crawler.ts
import { RapidAPI } from '@/lib/api/rapid';

interface ResearchSource {
  source: string;
  url: string;
  content: string;
  publishedAt: Date;
}

export async function fetchLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'twitter', 'linkedin']
): Promise<ResearchSource[]> {
  const results: ResearchSource[] = [];
  
  for (const source of sources) {
    const data = await RapidAPI.search({
      query: keyword,
      source: source,
      timeRange: '24h'
    });
    
    results.push(...data.map(item => ({
      source: item.source,
      url: item.url,
      content: item.content,
      publishedAt: new Date(item.publishedAt)
    })));
  }
  
  return results;
}

export async function analyzeResearch(
  sources: ResearchSource[],
  apiKey: string
): Promise<string> {
  const Anthropic = require('@anthropic-ai/sdk');
  const client = new Anthropic({ apiKey });
  
  const prompt = `Analyze the following research data and extract key insights:
  
${sources.map(s => `Source: ${s.source}\n${s.content}`).join('\n\n')}

Provide:
1. Top 3 trends
2. Key statistics with sources
3. Unique insights`;

  const response = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return response.content[0].text;
}
```

### 2. Content Generation

Generate content in multiple formats and languages:

```typescript
// lib/content/generator.ts
import OpenAI from 'openai';
import Anthropic from '@anthropic-ai/sdk';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Tone = 'expert' | 'friendly' | 'humorous';
type Language = 'en' | 'vi';

interface ContentRequest {
  topic: string;
  research: string;
  format: ContentFormat;
  tone: Tone;
  language: Language;
}

export async function generateContent(
  request: ContentRequest,
  provider: 'openai' | 'claude' = 'claude'
): Promise<string> {
  const formatPrompts = {
    'toplist': 'Create a numbered list article with rankings and explanations',
    'pov': 'Write an opinion piece with a clear point of view',
    'case-study': 'Develop a detailed case study with problem, solution, results',
    'how-to': 'Write a step-by-step tutorial'
  };
  
  const toneAdjustments = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and engaging, light-hearted tone'
  };
  
  const prompt = `${formatPrompts[request.format]}.

Topic: ${request.topic}
Research Data:
${request.research}

Style: ${toneAdjustments[request.tone]}
Language: ${request.language === 'vi' ? 'Vietnamese' : 'English'}

Create a comprehensive article (800-1200 words) with:
- Compelling headline
- Introduction
- Main body with subheadings
- Data-backed insights
- Conclusion with call-to-action`;

  if (provider === 'claude') {
    const client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const response = await client.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{ role: 'user', content: prompt }]
    });
    
    return response.content[0].text;
  } else {
    const client = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
    
    const response = await client.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 4000
    });
    
    return response.choices[0].message.content || '';
  }
}

// Generate bilingual content
export async function generateBilingualContent(
  request: Omit<ContentRequest, 'language'>
): Promise<{ en: string; vi: string }> {
  const [enContent, viContent] = await Promise.all([
    generateContent({ ...request, language: 'en' }),
    generateContent({ ...request, language: 'vi' })
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### 3. Video Generation with Remotion

Convert content to video format:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformConfigs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const { width, height, fps } = platformConfigs[config.platform];
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: webpackOverride
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      theme: 'modern'
    }
  });
  
  // Render video
  const outputLocation = path.join(outputPath, `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content
    }
  });
  
  return outputLocation;
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  theme?: 'modern' | 'minimal';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  theme = 'modern'
}) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  const contentOpacity = interpolate(frame, [30, 60], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  // Split content into segments for animation
  const contentSegments = content.split('\n\n').filter(Boolean);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#0a0a0a' }}>
      <Sequence from={0} durationInFrames={30}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity
          }}
        >
          <h1 style={{
            color: 'white',
            fontSize: 80,
            fontWeight: 'bold',
            textAlign: 'center',
            padding: '0 100px',
            fontFamily: 'Arial, sans-serif'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {contentSegments.map((segment, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: '0 100px',
              opacity: contentOpacity
            }}
          >
            <p style={{
              color: 'white',
              fontSize: 48,
              lineHeight: 1.5,
              textAlign: 'center',
              fontFamily: 'Arial, sans-serif'
            }}>
              {segment}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 5. Full Pipeline Integration

```typescript
// lib/pipeline/content-pipeline.ts
import { fetchLatestNews, analyzeResearch } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/content/generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function runContentPipeline(
  keyword: string,
  format: ContentFormat,
  platforms: ('reels' | 'tiktok' | 'shorts')[]
) {
  console.log(`Starting pipeline for keyword: ${keyword}`);
  
  // Step 1: Research
  console.log('Fetching latest news...');
  const sources = await fetchLatestNews(keyword);
  
  console.log('Analyzing research...');
  const insights = await analyzeResearch(
    sources,
    process.env.ANTHROPIC_API_KEY!
  );
  
  // Step 2: Generate Content
  console.log('Generating bilingual content...');
  const content = await generateBilingualContent({
    topic: keyword,
    research: insights,
    format: format,
    tone: 'expert'
  });
  
  // Step 3: Render Videos
  console.log('Rendering videos for platforms...');
  const videoPromises = platforms.flatMap(platform => [
    renderContentVideo({
      content: content.en,
      title: keyword,
      platform
    }, './output/videos/en'),
    renderContentVideo({
      content: content.vi,
      title: keyword,
      platform
    }, './output/videos/vi')
  ]);
  
  const videos = await Promise.all(videoPromises);
  
  return {
    research: insights,
    content,
    videos
  };
}
```

## API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, platforms } = await request.json();
    
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(
      keyword,
      format,
      platforms || ['reels', 'tiktok']
    );
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Usage Patterns

### Basic Content Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

// Generate AI content with video
const result = await runContentPipeline(
  'AI Marketing Trends 2024',
  'toplist',
  ['reels', 'tiktok', 'shorts']
);

console.log('Content (EN):', result.content.en);
console.log('Content (VI):', result.content.vi);
console.log('Videos:', result.videos);
```

### Custom Research Sources

```typescript
import { fetchLatestNews } from '@/lib/research/crawler';

const news = await fetchLatestNews(
  'ChatGPT',
  ['techcrunch', 'twitter']
);
```

### Video-Only Generation

```typescript
import { renderContentVideo } from '@/lib/video/renderer';

const videoPath = await renderContentVideo({
  content: 'Your pre-written content here',
  title: 'Video Title',
  platform: 'reels'
}, './output');
```

## Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) to access the UI.

## Remotion Studio

Preview and customize video compositions:

```bash
npm run remotion
# or
npx remotion studio
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting:

```typescript
// Add delay between requests
async function fetchWithDelay(keyword: string) {
  const results = [];
  for (const source of sources) {
    results.push(await fetchFromSource(source, keyword));
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
  }
  return results;
}
```

### Video Rendering Memory Issues

For large videos, increase Node memory:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run render
```

### Missing Environment Variables

Validate configuration at startup:

```typescript
// lib/config/validate.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing env variables: ${missing.join(', ')}`);
  }
}
```

### Content Quality Issues

Adjust prompts for better results:

```typescript
// Add more context to prompts
const enhancedPrompt = `${basePrompt}

Additional Requirements:
- Include at least 3 statistics with sources
- Add actionable takeaways
- Use storytelling elements
- Optimize for SEO with natural keyword placement`;
```

## Best Practices

1. **Cache Research Data**: Store crawled data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue for video generation to prevent timeout
3. **Content Review**: Implement human review before auto-publishing
4. **Error Handling**: Wrap API calls in try-catch with retries
5. **Rate Limiting**: Implement exponential backoff for API requests
