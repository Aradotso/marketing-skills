---
name: marketing-pipeline-share-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI
  - generate video from text content
  - crawl news and research for content ideas
  - create multilingual marketing content
  - automate social media content pipeline
  - generate blog posts with AI research
  - create content from keyword to video
  - build automated content workflow
---

# Marketing Pipeline Share Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that transforms keywords into complete content packages including research, articles, and videos. It combines AI models (Claude 3, OpenAI), web scraping, and video rendering (Remotion) to create an end-to-end content production pipeline.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- Generate multi-format content (listicles, POV, case studies, how-to)
- Support bilingual output (English & Vietnamese)
- Auto-render videos and infographics via Remotion
- Next.js frontend for easy management

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

## Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for content scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Video rendering (Remotion)
REMOTION_BUNDLE_SIZE_LIMIT=50000000
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render video (Remotion)
npm run render
```

## Core Architecture

### 1. Research & Scraping Module

The pipeline starts by gathering fresh content from various sources:

```typescript
// lib/scraper/news-aggregator.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsSource {
  name: string;
  url: string;
  category: string;
}

export async function scrapeNewsForKeyword(keyword: string): Promise<Article[]> {
  const sources: NewsSource[] = [
    { name: 'TechCrunch', url: 'techcrunch.com', category: 'tech' },
    { name: 'a16z', url: 'a16z.com', category: 'startups' },
  ];

  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY);
  const articles: Article[] = [];

  for (const source of sources) {
    try {
      const results = await rapidAPI.searchNews({
        query: keyword,
        domain: source.url,
        timeRange: '24h',
      });

      articles.push(...results.map(parseArticle));
    } catch (error) {
      console.error(`Failed to scrape ${source.name}:`, error);
    }
  }

  return articles;
}

function parseArticle(raw: any): Article {
  return {
    title: raw.title,
    url: raw.url,
    excerpt: raw.description,
    publishedAt: new Date(raw.publishedDate),
    source: raw.source,
  };
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI based on scraped research:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  research: Article[];
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateWithClaude(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);

    const message = await this.anthropic.messages.create({
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

  async generateWithOpenAI(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);

    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content marketer.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      temperature: 0.7,
    });

    return completion.choices[0]?.message?.content || '';
  }

  private buildPrompt(request: ContentRequest): string {
    const researchSummary = request.research
      .map((article, i) => `[${i + 1}] ${article.title}\n${article.excerpt}`)
      .join('\n\n');

    return `
Create a ${request.format} article about "${request.keyword}" in ${request.language}.
Tone: ${request.tone}

Recent research (last 24h):
${researchSummary}

Requirements:
- Use data and insights from the research above
- Include specific examples and statistics
- Format as ${request.format}
- Write in ${request.language === 'vi' ? 'Vietnamese' : 'English'}
- Maintain ${request.tone} tone throughout
${request.format === 'toplist' ? '- Create numbered list with clear sections' : ''}
${request.format === 'how-to' ? '- Provide step-by-step instructions' : ''}

Output the complete article in markdown format.
    `.trim();
  }
}
```

### 3. Video Generation with Remotion

Transform content into video format:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

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
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title sequence */}
      <Sequence from={0} durationInFrames={fps * 2}>
        <TitleSlide title={title} color={brandColor} />
      </Sequence>

      {/* Content points */}
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <PointSlide point={point} index={index + 1} color={brandColor} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: GeneratedContent
): Promise<string> {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      brandColor: '#3B82F6',
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${content.id}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}
```

### 4. Complete Pipeline Integration

```typescript
// lib/pipeline/content-pipeline.ts
export class ContentPipeline {
  private generator: ContentGenerator;

  constructor() {
    this.generator = new ContentGenerator();
  }

  async execute(input: PipelineInput): Promise<PipelineOutput> {
    // Step 1: Research
    console.log('🔍 Scraping news...');
    const articles = await scrapeNewsForKeyword(input.keyword);

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await this.generator.generateWithClaude({
      keyword: input.keyword,
      format: input.format,
      language: input.language,
      tone: input.tone,
      research: articles,
    });

    // Step 3: Parse and structure
    const structured = this.parseContent(content);

    // Step 4: Generate video (optional)
    let videoPath: string | undefined;
    if (input.generateVideo) {
      console.log('🎬 Rendering video...');
      videoPath = await renderContentVideo(structured);
    }

    return {
      markdown: content,
      structured,
      videoPath,
      research: articles,
      generatedAt: new Date(),
    };
  }

  private parseContent(markdown: string): GeneratedContent {
    // Extract title, sections, key points
    const lines = markdown.split('\n');
    const title = lines.find(l => l.startsWith('#'))?.replace(/^#+\s*/, '') || '';
    const keyPoints = lines
      .filter(l => l.match(/^\d+\.|^-\s/))
      .map(l => l.replace(/^\d+\.\s*|-\s*/, ''));

    return {
      id: Date.now().toString(),
      title,
      body: markdown,
      keyPoints,
    };
  }
}
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, generateVideo } = body;

    const pipeline = new ContentPipeline();
    const result = await pipeline.execute({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      tone: tone || 'expert',
      generateVideo: generateVideo || false,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Frontend Usage

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (formData: FormData) => {
    setLoading(true);

    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: formData.get('keyword'),
          format: formData.get('format'),
          language: formData.get('language'),
          tone: formData.get('tone'),
          generateVideo: formData.get('generateVideo') === 'on',
        }),
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <form onSubmit={(e) => {
        e.preventDefault();
        handleGenerate(new FormData(e.currentTarget));
      }}>
        <input name="keyword" placeholder="Enter keyword..." required />
        <select name="format">
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How To</option>
        </select>
        <select name="language">
          <option value="en">English</option>
          <option value="vi">Vietnamese</option>
        </select>
        <label>
          <input type="checkbox" name="generateVideo" />
          Generate Video
        </label>
        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result?.data && (
        <div className="mt-8">
          <h2>{result.data.structured.title}</h2>
          <div dangerouslySetInnerHTML={{ __html: result.data.markdown }} />
          {result.data.videoPath && (
            <video src={result.data.videoPath} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatch(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      pipeline.execute({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        generateVideo: false,
      })
    )
  );

  return results.map((result, i) => ({
    keyword: keywords[i],
    success: result.status === 'fulfilled',
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

### Custom Format Templates

```typescript
const customFormats = {
  'product-review': `
Create a detailed product review about {keyword}.
Include: pros, cons, pricing, alternatives, verdict.
`,
  'comparison': `
Compare {keyword} with top 3 alternatives.
Create comparison table and recommendations.
`,
};

function buildCustomPrompt(format: string, keyword: string): string {
  return customFormats[format]?.replace('{keyword}', keyword) || '';
}
```

## Troubleshooting

**API rate limits:**
```typescript
// Add retry logic with exponential backoff
async function withRetry<T>(fn: () => Promise<T>, maxRetries = 3): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
    }
  }
  throw new Error('Max retries exceeded');
}
```

**Video rendering memory issues:**
```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run render
```

**Missing environment variables:**
```typescript
// lib/config/validate-env.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY',
];

requiredEnvVars.forEach(varName => {
  if (!process.env[varName]) {
    throw new Error(`Missing required environment variable: ${varName}`);
  }
});
```
