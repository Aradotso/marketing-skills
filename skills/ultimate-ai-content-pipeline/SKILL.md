---
name: ultimate-ai-content-pipeline
description: Automated Vietnamese/English content creation pipeline with AI research, script generation, and video rendering using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline for automated marketing
  - generate Vietnamese content with AI research and video rendering
  - automate content creation from keyword research to video
  - use Claude and OpenAI for multilingual content generation
  - create marketing videos automatically with Remotion
  - scrape news and generate content scripts with AI
  - build automated content workflow with this pipeline
  - render videos from text content using this system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete content automation pipeline that transforms keywords into fully-researched articles and videos. It crawls real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates multilingual content (Vietnamese/English) using Claude/OpenAI, and renders videos automatically using Remotion.

## What It Does

- **Auto-Research**: Crawls fresh news and insights from major tech/business sources
- **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos)
- **Multilingual Support**: Generates parallel Vietnamese and English content
- **Video Rendering**: Converts text content into videos/infographics for social media
- **Multi-Platform Export**: Optimized for Reels, TikTok, Shorts

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

```env
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for data crawling
RAPIDAPI_KEY=your_rapidapi_key

# Remotion for video rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```typescript
// Typical structure based on Next.js + TypeScript
src/
├── app/              # Next.js app directory
├── components/       # React components
├── lib/
│   ├── ai/          # AI service integrations
│   ├── crawler/     # Web scraping modules
│   ├── video/       # Remotion video generation
│   └── utils/       # Helper functions
├── remotion/        # Remotion video templates
└── types/           # TypeScript definitions
```

## Core Usage Patterns

### 1. Initialize Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language } from '@/types';

// Generate content from keyword
const result = await generateContent({
  keyword: 'AI marketing automation',
  format: ContentFormat.LISTICLE,
  languages: [Language.VIETNAMESE, Language.ENGLISH],
  includeResearch: true,
  tone: 'professional',
});

console.log(result.articles); // { vi: '...', en: '...' }
console.log(result.metadata); // Research sources, stats
```

### 2. Research Module (Auto-Crawl)

```typescript
import { researchTopic } from '@/lib/crawler/research';

// Crawl and analyze recent news
const insights = await researchTopic({
  topic: 'generative AI trends',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  maxResults: 10,
});

// Returns structured data
interface ResearchResult {
  articles: Array<{
    title: string;
    url: string;
    summary: string;
    publishedAt: Date;
    source: string;
  }>;
  keyInsights: string[];
  statistics: Array<{ metric: string; value: string }>;
  trends: string[];
}
```

### 3. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Generate Vietnamese content
async function generateVietnameseArticle(
  research: ResearchResult,
  format: string
): Promise<string> {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Dựa trên nghiên cứu sau, viết một bài ${format} bằng tiếng Việt:
        
${JSON.stringify(research, null, 2)}

Yêu cầu:
- Giọng văn chuyên nghiệp nhưng dễ hiểu
- Có số liệu cụ thể
- Phù hợp với marketer Việt Nam`,
      },
    ],
  });

  return message.content[0].text;
}
```

### 4. OpenAI Integration Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateEnglishContent(
  prompt: string,
  format: string
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a marketing content expert. Generate ${format} content that is data-driven and engaging.`,
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Render video from content
async function renderContentVideo(
  content: string,
  outputPath: string
): Promise<string> {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.split('\n')[0],
      body: content,
      duration: 30, // seconds
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps,
  });

  return outputPath;
}
```

### 6. Remotion Video Template

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  body: string;
  duration: number;
}> = ({ title, body }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div style={{ opacity, padding: 60, maxWidth: 800 }}>
        <h1 style={{ color: 'white', fontSize: 48, marginBottom: 20 }}>
          {title}
        </h1>
        <p style={{ color: '#ccc', fontSize: 24, lineHeight: 1.6 }}>
          {body.substring(0, 300)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 7. Complete Pipeline Orchestration

```typescript
import { generateContentPipeline } from '@/lib/pipeline';

// Full end-to-end pipeline
async function runContentPipeline(keyword: string) {
  try {
    const pipeline = await generateContentPipeline({
      keyword,
      options: {
        research: true,
        languages: ['vi', 'en'],
        formats: ['listicle', 'how-to'],
        generateVideo: true,
        videoSpecs: {
          duration: 30,
          aspectRatio: '9:16', // Vertical for Reels/TikTok
          fps: 30,
        },
      },
    });

    // Returns complete output
    return {
      research: pipeline.research,
      articles: pipeline.articles, // { vi: {...}, en: {...} }
      videos: pipeline.videos, // Array of video file paths
      metadata: pipeline.metadata,
    };
  } catch (error) {
    console.error('Pipeline failed:', error);
    throw error;
  }
}

// Usage
const output = await runContentPipeline('AI content marketing 2024');
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, options } = await request.json();

    const result = await generateContentPipeline({
      keyword,
      options,
    });

    return NextResponse.json(result, { status: 200 });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render a video (if CLI tool exists)
npm run render:video -- --input content.json --output video.mp4

# Run research crawler
npm run research -- --topic "AI marketing" --sources techcrunch,a16z
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple pieces of content
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map((keyword) =>
      generateContentPipeline({
        keyword,
        options: { research: true, languages: ['vi', 'en'] },
      })
    )
  );

  return results
    .filter((r) => r.status === 'fulfilled')
    .map((r) => r.value);
}
```

### Custom Content Formats

```typescript
// Define custom content format
interface CustomFormat {
  name: string;
  template: string;
  sections: string[];
  tone: 'professional' | 'casual' | 'humorous';
}

const caseStudyFormat: CustomFormat = {
  name: 'case-study',
  template: 'problem-solution-results',
  sections: ['Challenge', 'Solution', 'Results', 'Lessons'],
  tone: 'professional',
};

// Use in generation
const content = await generateContent({
  keyword: 'marketing automation success',
  customFormat: caseStudyFormat,
  language: 'vi',
});
```

### Video Customization

```typescript
// Custom video template with branding
interface VideoBranding {
  logo: string;
  colors: {
    primary: string;
    secondary: string;
  };
  fonts: {
    heading: string;
    body: string;
  };
}

async function renderBrandedVideo(
  content: string,
  branding: VideoBranding
): Promise<string> {
  return renderContentVideo(content, './output.mp4', {
    branding,
    duration: 45,
    transitions: 'smooth',
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function generateWithRetry(
  prompt: string,
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(prompt);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise((resolve) =>
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      } else {
        throw error;
      }
    }
  }
}
```

### Memory Issues with Video Rendering

```typescript
// Process videos in chunks
async function renderVideosBatch(
  contents: string[],
  batchSize = 3
): Promise<string[]> {
  const results: string[] = [];

  for (let i = 0; i < contents.length; i += batchSize) {
    const batch = contents.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map((content, idx) =>
        renderContentVideo(content, `./output-${i + idx}.mp4`)
      )
    );
    results.push(...batchResults);
  }

  return results;
}
```

### Crawler Blocking

```typescript
// Use proxy rotation (requires proxy service)
import { researchTopic } from '@/lib/crawler/research';

const insights = await researchTopic({
  topic: 'AI trends',
  sources: ['techcrunch'],
  timeframe: '24h',
  proxyConfig: {
    enabled: true,
    provider: process.env.PROXY_PROVIDER,
    apiKey: process.env.PROXY_API_KEY,
  },
});
```

### Handling Vietnamese Diacritics

```typescript
// Ensure proper UTF-8 encoding
import { normalize } from '@/lib/utils/text';

function normalizeVietnameseText(text: string): string {
  return text.normalize('NFC'); // Canonical composition
}

// Use in content generation
const content = await generateVietnameseArticle(research, format);
const normalized = normalizeVietnameseText(content);
```

## Best Practices

1. **Always validate API keys** before starting pipeline
2. **Cache research data** to avoid redundant API calls
3. **Use queue systems** (Bull, BullMQ) for video rendering
4. **Store generated content** in database with metadata
5. **Implement content moderation** before auto-posting
6. **Monitor API usage** and costs across all providers

## Development Workflow

```typescript
// Typical development flow
import { testPipeline } from '@/lib/testing';

// 1. Test research module
const research = await testPipeline.research('test keyword');

// 2. Test content generation
const content = await testPipeline.generate(research);

// 3. Test video rendering (skip in CI)
if (process.env.NODE_ENV !== 'ci') {
  const video = await testPipeline.render(content);
}

// 4. Review and deploy
console.log('Pipeline test complete:', { research, content, video });
```
