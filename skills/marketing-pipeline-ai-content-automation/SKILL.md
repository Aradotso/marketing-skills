---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I generate automated marketing content with AI
  - set up the marketing pipeline for content automation
  - create AI-powered content with research and video generation
  - automate content from research to video using this pipeline
  - configure Claude and OpenAI for automated content creation
  - generate videos automatically from content using Remotion
  - build an AI content pipeline with auto-research
  - use marketing-pipeline-share for content automation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research, scriptwriting, posting, and video generation. It automatically crawls news from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates content in multiple formats using Claude/OpenAI, and renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major tech news sources
- **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese
- **Video Rendering**: Automatically converts content to infographics and short-form videos
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Utility functions and API integrations
│   │   ├── ai/          # AI service integrations (Claude, OpenAI)
│   │   ├── research/    # News crawling and research
│   │   └── video/       # Video generation with Remotion
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Collection

```typescript
// lib/research/crawler.ts
import { NewsSource } from '@/types';

interface ResearchConfig {
  sources: NewsSource[];
  keywords: string[];
  timeRange: '24h' | '7d' | '30d';
}

export async function crawlNews(config: ResearchConfig) {
  const { sources, keywords, timeRange } = config;
  
  const apiKey = process.env.RAPIDAPI_KEY;
  if (!apiKey) throw new Error('RAPIDAPI_KEY not configured');

  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await fetch(`https://news-api.example.com/v1/search`, {
        headers: {
          'X-RapidAPI-Key': apiKey,
          'X-RapidAPI-Host': 'news-api.example.com',
        },
        method: 'POST',
        body: JSON.stringify({
          source,
          keywords,
          timeRange,
        }),
      });
      
      return response.json();
    })
  );

  return results.flat();
}

// Usage
const newsData = await crawlNews({
  sources: ['techcrunch', 'a16z', 'twitter'],
  keywords: ['AI', 'marketing', 'automation'],
  timeRange: '24h',
});
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentGenerationParams {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateContent(params: ContentGenerationParams) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(params);

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

function buildPrompt(params: ContentGenerationParams): string {
  const { topic, format, language, tone, researchData } = params;
  
  return `
You are an expert content creator. Generate a ${format} article about ${topic}.

Language: ${language}
Tone: ${tone}

Research Data:
${JSON.stringify(researchData, null, 2)}

Requirements:
- Use data-backed insights from the research
- Make it engaging and actionable
- Include specific examples and numbers
- Optimize for ${language === 'vi' ? 'Vietnamese' : 'English'} readers
- Structure with clear headings and bullet points
`;
}
```

### 3. OpenAI Integration (Alternative)

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateWithOpenAI(params: ContentGenerationParams) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content creator.',
      },
      {
        role: 'user',
        content: buildPrompt(params),
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  outputPath: string;
}

export async function renderContentVideo(config: VideoConfig) {
  const { title, content, format, outputPath } = config;

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content,
      format,
    },
  });

  // Render video
  await renderMedia({
    composition: compositions,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      content,
      format,
    },
  });

  return outputPath;
}
```

### 5. Complete Pipeline Example

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/render-video';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    // Step 1: Research
    const researchData = await crawlNews({
      sources: ['techcrunch', 'a16z', 'twitter'],
      keywords: [keyword],
      timeRange: '24h',
    });

    // Step 2: Generate Content
    const content = await generateContent({
      topic: keyword,
      format: format || 'toplist',
      language: language || 'en',
      tone: 'expert',
      researchData,
    });

    // Step 3: Generate Video (optional)
    const videoPath = await renderContentVideo({
      title: keyword,
      content,
      format: 'reels',
      outputPath: `./public/videos/${keyword}-${Date.now()}.mp4`,
    });

    return NextResponse.json({
      success: true,
      content,
      videoPath,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Remotion Video Template Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  format,
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ color: '#fff', fontSize: 60, marginBottom: 20 }}>
          {title}
        </h1>
        <p style={{ color: '#fff', fontSize: 30, lineHeight: 1.5 }}>
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Multi-language Content Generation

```typescript
async function generateBilingualContent(topic: string, researchData: any[]) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      topic,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData,
    }),
    generateContent({
      topic,
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      researchData,
    }),
  ]);

  return { en: englishContent, vi: vietnameseContent };
}
```

### Batch Video Rendering

```typescript
async function renderMultipleFormats(content: string, title: string) {
  const formats: Array<'reels' | 'tiktok' | 'shorts'> = ['reels', 'tiktok', 'shorts'];
  
  const videos = await Promise.all(
    formats.map((format) =>
      renderContentVideo({
        title,
        content,
        format,
        outputPath: `./public/videos/${title}-${format}-${Date.now()}.mp4`,
      })
    )
  );

  return videos;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits with AI providers:

```typescript
// lib/utils/rate-limiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  retries = 3,
  delay = 1000
): Promise<T> {
  try {
    return await fn();
  } catch (error: any) {
    if (error.status === 429 && retries > 0) {
      await new Promise((resolve) => setTimeout(resolve, delay));
      return withRateLimit(fn, retries - 1, delay * 2);
    }
    throw error;
  }
}

// Usage
const content = await withRateLimit(() =>
  generateContent(params)
);
```

### Video Rendering Memory Issues

For large video renders:

```typescript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "remotion": "NODE_OPTIONS='--max-old-space-size=4096' remotion"
  }
}
```

### Missing Environment Variables

```typescript
// lib/utils/validate-env.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter((key) => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

## Best Practices

1. **Cache Research Data**: Store crawled data to avoid redundant API calls
2. **Use Streaming**: For long content generation, use streaming responses
3. **Error Handling**: Always wrap AI calls in try-catch with proper logging
4. **Content Validation**: Verify generated content meets quality standards before rendering videos
5. **Asset Management**: Store generated videos in cloud storage (S3, Cloudinary) for production use
