---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI for Vietnamese and English marketing content
triggers:
  - "generate automated marketing content with AI"
  - "create video content from blog posts automatically"
  - "set up content automation pipeline with Claude"
  - "scrape trending news and generate content"
  - "build AI-powered content research system"
  - "automate social media video creation"
  - "create multilingual content with AI agents"
  - "generate toplist and case study articles automatically"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use the Ultimate AI Content Pipeline, a TypeScript-based automation system that handles the complete content creation workflow: from research (crawling news sources), through AI-powered content generation (using Claude 3/OpenAI), to automated video rendering (via Remotion). The system supports multiple content formats, bilingual output (Vietnamese/English), and platform-optimized video generation.

## What This Project Does

The Ultimate AI Content Pipeline is a comprehensive content automation system that:

- **Auto-scans research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **AI content generation**: Creates diverse formats (toplists, POV articles, case studies, how-tos) using Claude 3 or OpenAI
- **Multilingual support**: Generates parallel Vietnamese and English versions with customizable tone
- **Video automation**: Converts written content into infographics and short-form videos using Remotion
- **Multi-platform optimization**: Exports videos for Reels, TikTok, Shorts with proper aspect ratios

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

## Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Database (if needed)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run Remotion video preview
npm run remotion:preview

# Render video
npm run remotion:render
```

## Core Architecture

### 1. Research Module (Data Crawling)

The research module scrapes trending content from multiple sources:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  name: string;
  url: string;
  parser: (data: any) => Article[];
}

export async function crawlTechCrunch(keyword: string): Promise<Article[]> {
  const options = {
    method: 'GET',
    url: 'https://techcrunch-articles.p.rapidapi.com/search',
    params: { query: keyword, limit: '10' },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'techcrunch-articles.p.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles.map(parseArticle);
  } catch (error) {
    console.error('TechCrunch crawl error:', error);
    return [];
  }
}

function parseArticle(raw: any): Article {
  return {
    title: raw.title,
    content: raw.content,
    url: raw.url,
    publishedAt: new Date(raw.published_at),
    source: 'TechCrunch'
  };
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with customizable formats:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Tone = 'expert' | 'friendly' | 'humorous';
export type Language = 'en' | 'vi';

interface GenerateContentParams {
  keyword: string;
  researchData: Article[];
  format: ContentFormat;
  tone: Tone;
  language: Language;
  provider?: 'claude' | 'openai';
}

export async function generateContent({
  keyword,
  researchData,
  format,
  tone,
  language,
  provider = 'claude'
}: GenerateContentParams): Promise<string> {
  const prompt = buildPrompt(keyword, researchData, format, tone, language);

  if (provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

async function generateWithOpenAI(prompt: string): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer and marketer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    max_tokens: 4096,
  });

  return completion.choices[0]?.message?.content || '';
}

function buildPrompt(
  keyword: string,
  researchData: Article[],
  format: ContentFormat,
  tone: Tone,
  language: Language
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a specific point of view, presenting a unique angle or opinion',
    'case-study': 'Develop a detailed case study with problem, solution, and results',
    'how-to': 'Create a step-by-step tutorial or guide'
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Write in a conversational, approachable tone',
    'humorous': 'Include light humor and engaging storytelling'
  };

  const languageInstruction = language === 'vi' 
    ? 'Write the entire article in Vietnamese.'
    : 'Write the entire article in English.';

  const researchSummary = researchData
    .map(article => `- ${article.title} (${article.source})`)
    .join('\n');

  return `
Create a ${format} article about "${keyword}".

Research data from the last 24 hours:
${researchSummary}

Format: ${formatInstructions[format]}
Tone: ${toneInstructions[tone]}
Language: ${languageInstruction}

Requirements:
- Use data-backed insights from the research
- Include specific examples and statistics
- Make it SEO-friendly with proper headings
- Add actionable takeaways
- Length: 1500-2000 words
`;
}
```

### 3. Video Generation with Remotion

Create videos automatically from content:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  brand: {
    logo?: string;
    primaryColor: string;
  };
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  points,
  brand 
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleDuration = fps * 2; // 2 seconds
  const pointDuration = fps * 3; // 3 seconds per point

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title sequence */}
      <Sequence from={0} durationInFrames={titleDuration}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          padding: 40 
        }}>
          <h1 style={{ 
            color: brand.primaryColor, 
            fontSize: 60,
            textAlign: 'center',
            opacity: Math.min(1, frame / 15)
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Point sequences */}
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={titleDuration + index * pointDuration}
          durationInFrames={pointDuration}
        >
          <PointSlide 
            point={point} 
            index={index + 1}
            color={brand.primaryColor}
          />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ 
  point: string; 
  index: number;
  color: string;
}> = ({ point, index, color }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ 
      justifyContent: 'center', 
      alignItems: 'center',
      padding: 60 
    }}>
      <div style={{ 
        fontSize: 48, 
        color: '#fff',
        opacity: Math.min(1, frame / 15)
      }}>
        <span style={{ color }}>{index}.</span> {point}
      </div>
    </AbsoluteFill>
  );
};
```

Render the video:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: {
    title: string;
    points: string[];
  },
  outputPath: string
): Promise<string> {
  const bundled = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.points,
      brand: {
        primaryColor: '#FF6B6B'
      }
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
  });

  return outputPath;
}
```

### 4. Complete Pipeline API

Create a Next.js API route that orchestrates the entire pipeline:

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlTechCrunch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone, language, generateVideo } = await request.json();

    // Step 1: Research
    console.log('🔍 Starting research...');
    const researchData = await crawlTechCrunch(keyword);

    if (researchData.length === 0) {
      return NextResponse.json(
        { error: 'No research data found' },
        { status: 404 }
      );
    }

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      keyword,
      researchData,
      format,
      tone,
      language,
      provider: 'claude'
    });

    let videoUrl = null;

    // Step 3: Generate video (optional)
    if (generateVideo) {
      console.log('🎬 Rendering video...');
      const points = extractKeyPoints(content);
      const outputPath = path.join(
        process.cwd(),
        'public/videos',
        `${Date.now()}.mp4`
      );

      await renderContentVideo(
        {
          title: keyword,
          points: points.slice(0, 5) // Top 5 points
        },
        outputPath
      );

      videoUrl = `/videos/${path.basename(outputPath)}`;
    }

    return NextResponse.json({
      success: true,
      content,
      videoUrl,
      research: {
        sources: researchData.length,
        keywords: keyword
      }
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - look for numbered lists or headings
  const lines = content.split('\n');
  const points: string[] = [];

  for (const line of lines) {
    const match = line.match(/^(?:\d+\.|[-*])\s*(.+)/);
    if (match && match[1]) {
      points.push(match[1].trim());
    }
  }

  return points;
}
```

## Common Usage Patterns

### Pattern 1: Single Content Generation

```typescript
// Generate a toplist article in Vietnamese
const result = await fetch('/api/generate-content', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI Marketing Tools 2024',
    format: 'toplist',
    tone: 'expert',
    language: 'vi',
    generateVideo: false
  })
});

const data = await result.json();
console.log(data.content);
```

### Pattern 2: Batch Content Generation

```typescript
// lib/batch/processor.ts
export async function generateBatchContent(
  keywords: string[],
  config: {
    format: ContentFormat;
    tone: Tone;
    language: Language;
  }
) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const researchData = await crawlTechCrunch(keyword);
      const content = await generateContent({
        keyword,
        researchData,
        ...config
      });
      return { keyword, content };
    })
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
}
```

### Pattern 3: Scheduled Content Pipeline

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

export function scheduleContentGeneration() {
  // Run daily at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('🚀 Running scheduled content generation...');

    const trendingKeywords = await getTrendingKeywords();

    for (const keyword of trendingKeywords) {
      await fetch('http://localhost:3000/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'pov',
          tone: 'friendly',
          language: 'en',
          generateVideo: true
        })
      });
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
  private processing = false;
  private delay: number;

  constructor(requestsPerMinute: number) {
    this.delay = 60000 / requestsPerMinute;
  }

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;

    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) {
        await fn();
        await new Promise(resolve => setTimeout(resolve, this.delay));
      }
    }

    this.processing = false;
  }
}

// Usage
const claudeLimiter = new RateLimiter(50); // 50 requests per minute

const content = await claudeLimiter.add(() =>
  generateContent({ keyword, researchData, format, tone, language })
);
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrency for video rendering
import PQueue from 'p-queue';

const videoQueue = new PQueue({ concurrency: 1 });

export async function renderMultipleVideos(contents: ContentData[]) {
  return videoQueue.addAll(
    contents.map(content => async () => {
      return renderContentVideo(content, getOutputPath(content));
    })
  );
}
```

### Research Data Quality

```typescript
// Validate and filter research results
function validateResearchData(articles: Article[]): Article[] {
  return articles.filter(article => {
    // Must have minimum content length
    if (!article.content || article.content.length < 200) return false;

    // Must be recent (last 24 hours)
    const dayAgo = new Date(Date.now() - 24 * 60 * 60 * 1000);
    if (article.publishedAt < dayAgo) return false;

    // Must have valid URL
    if (!article.url || !article.url.startsWith('http')) return false;

    return true;
  });
}
```

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline, enabling AI coding agents to effectively assist developers in setting up and using the automated content generation system.
