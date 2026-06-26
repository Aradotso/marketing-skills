---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, script generation, and video creation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create content using Claude and OpenAI APIs
  - build automated marketing content workflow
  - how to use the content automation system
  - set up Remotion video generation for content
  - crawl news and generate content automatically
  - automate content research and video creation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript/Next.js system that automates the entire content creation workflow: from research (crawling news sources), to script generation (using Claude/OpenAI), to video rendering (using Remotion).

## What This Project Does

The Ultimate AI Content Pipeline is a complete content automation system that:

1. **Auto-Research**: Crawls real-time data from news sources (TechCrunch, a16z, Twitter/X, LinkedIn)
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Generation**: Automatically renders infographics and short videos using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following environment variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if used)
DATABASE_URL=your_database_connection_string

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── crawler/          # News crawling logic
│   ├── content/          # Content generation pipeline
│   └── video/            # Remotion video rendering
├── remotion/             # Remotion video templates
├── public/               # Static assets
└── types/                # TypeScript definitions
```

## Key Components & Usage

### 1. Content Research & Crawling

```typescript
// lib/crawler/newsCrawler.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}

export async function crawlNews(keyword: string, sources: string[] = ['techcrunch', 'twitter']): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  const options = {
    method: 'GET',
    url: 'https://news-api.rapidapi.com/search',
    params: {
      q: keyword,
      lang: 'en',
      sources: sources.join(','),
      from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString() // Last 24h
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'news-api.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles.map((article: any) => ({
      title: article.title,
      url: article.url,
      publishedAt: article.publishedAt,
      source: article.source.name,
      content: article.description || article.content
    }));
  } catch (error) {
    console.error('Error crawling news:', error);
    return [];
  }
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claudeGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

interface ContentGenerationOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContent(options: ContentGenerationOptions): Promise<string> {
  const { keyword, format, language, tone, researchData } = options;
  
  const systemPrompt = `You are an expert content creator specializing in ${format} articles. 
Write in ${language === 'en' ? 'English' : 'Vietnamese'} with a ${tone} tone.
Use the provided research data to create data-backed, insightful content.`;

  const userPrompt = `Create a ${format} article about "${keyword}" using this research data:

${researchData}

Requirements:
- Use real data and statistics from the research
- Include actionable insights
- Make it engaging and shareable
- Optimize for ${language === 'en' ? 'English' : 'Vietnamese'} audience`;

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

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openaiGenerator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export async function generateContentWithOpenAI(options: ContentGenerationOptions): Promise<string> {
  const { keyword, format, language, tone, researchData } = options;
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${format} articles. 
Write in ${language === 'en' ? 'English' : 'Vietnamese'} with a ${tone} tone.`
      },
      {
        role: 'user',
        content: `Create a ${format} article about "${keyword}" using this research:\n\n${researchData}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Complete Content Pipeline

```typescript
// lib/content/contentPipeline.ts
import { crawlNews } from '../crawler/newsCrawler';
import { generateContent } from '../ai/claudeGenerator';

interface PipelineOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  sources?: string[];
}

export async function runContentPipeline(options: PipelineOptions) {
  const { keyword, format, language, tone, sources } = options;
  
  console.log(`🔍 Step 1: Researching "${keyword}"...`);
  const articles = await crawlNews(keyword, sources);
  
  if (articles.length === 0) {
    throw new Error('No articles found for the given keyword');
  }
  
  console.log(`📰 Found ${articles.length} articles`);
  
  const researchData = articles.map(article => 
    `Title: ${article.title}\nSource: ${article.source}\nDate: ${article.publishedAt}\nContent: ${article.content}\n---`
  ).join('\n\n');
  
  console.log(`✍️ Step 2: Generating ${format} content in ${language}...`);
  const content = await generateContent({
    keyword,
    format,
    language,
    tone,
    researchData
  });
  
  console.log(`✅ Content generated successfully!`);
  
  return {
    content,
    metadata: {
      keyword,
      format,
      language,
      tone,
      sourcesUsed: articles.length,
      generatedAt: new Date().toISOString()
    }
  };
}
```

### 5. Remotion Video Generation

```typescript
// lib/video/videoRenderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  contentTitle: string;
  contentBody: string;
  outputPath: string;
  format: 'reels' | 'tiktok' | 'shorts'; // All use 9:16
}

export async function renderContentVideo(options: VideoRenderOptions) {
  const { contentTitle, contentBody, outputPath, format } = options;
  
  console.log('🎬 Bundling Remotion project...');
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const compositionId = 'ContentVideo';
  
  console.log('🎥 Rendering video...');
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: contentTitle,
      body: contentBody,
      format
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: contentTitle,
      body: contentBody,
      format
    }
  });

  console.log(`✅ Video rendered: ${outputPath}`);
  return outputPath;
}
```

### 6. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  body: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, body }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  const bodyOpacity = interpolate(frame, [30, 60], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{
      backgroundColor: '#1a1a1a',
      justifyContent: 'center',
      alignItems: 'center',
      padding: 40
    }}>
      <div style={{
        opacity: titleOpacity,
        fontSize: 48,
        fontWeight: 'bold',
        color: 'white',
        textAlign: 'center',
        marginBottom: 30
      }}>
        {title}
      </div>
      
      <div style={{
        opacity: bodyOpacity,
        fontSize: 24,
        color: '#e0e0e0',
        textAlign: 'center',
        maxWidth: 600,
        lineHeight: 1.5
      }}>
        {body.substring(0, 200)}...
      </div>
    </AbsoluteFill>
  );
};
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/content/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, sources } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      tone: tone || 'expert',
      sources: sources || ['techcrunch', 'twitter']
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Render Video Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/videoRenderer';
import path from 'path';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { title, content, format } = body;

    if (!title || !content) {
      return NextResponse.json(
        { error: 'Title and content are required' },
        { status: 400 }
      );
    }

    const outputPath = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);
    
    const videoPath = await renderContentVideo({
      contentTitle: title,
      contentBody: content,
      outputPath,
      format: format || 'reels'
    });

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(videoPath)}`
    });
  } catch (error) {
    console.error('Video rendering error:', error);
    return NextResponse.json(
      { error: 'Failed to render video' },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Full Workflow Example

```typescript
// Example: Complete content generation workflow
import { runContentPipeline } from '@/lib/content/contentPipeline';
import { renderContentVideo } from '@/lib/video/videoRenderer';

async function createContentWithVideo(keyword: string) {
  // Step 1: Generate content
  const result = await runContentPipeline({
    keyword,
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    sources: ['techcrunch', 'a16z', 'twitter']
  });

  console.log('Generated content:', result.content);

  // Step 2: Extract title and summary
  const lines = result.content.split('\n');
  const title = lines[0].replace(/^#\s+/, '');
  const summary = lines.slice(1, 5).join(' ');

  // Step 3: Render video
  const videoPath = await renderContentVideo({
    contentTitle: title,
    contentBody: summary,
    outputPath: `./public/videos/${keyword.replace(/\s+/g, '-')}.mp4`,
    format: 'reels'
  });

  return {
    content: result.content,
    video: videoPath,
    metadata: result.metadata
  };
}

// Usage
createContentWithVideo('AI trends 2024')
  .then(result => console.log('Complete!', result))
  .catch(error => console.error('Error:', error));
```

### Batch Content Generation

```typescript
// Generate multiple content pieces
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runContentPipeline({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert'
      })
    )
  );

  return results;
}

// Usage
const keywords = ['AI automation', 'Content marketing', 'Video creation'];
batchGenerate(keywords);
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

## Troubleshooting

### API Key Issues

```typescript
// Check if API keys are properly loaded
function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}
```

### Rate Limiting

```typescript
// Add rate limiting for API calls
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

async function crawlNewsWithRateLimit(keywords: string[]) {
  return Promise.all(
    keywords.map(keyword => 
      limit(() => crawlNews(keyword))
    )
  );
}
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Content Quality Check

```typescript
// Validate generated content
function validateContent(content: string): boolean {
  const minLength = 500;
  const hasTitle = /^#\s+.+/m.test(content);
  const hasBody = content.length > minLength;
  
  return hasTitle && hasBody;
}
```

## Best Practices

1. **Cache Research Data**: Store crawled articles to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for video processing
3. **Content Versioning**: Save multiple versions of generated content
4. **Error Handling**: Implement retry logic for API failures
5. **Monitor Costs**: Track AI API usage to manage costs

## TypeScript Types

```typescript
// types/content.ts
export interface ContentMetadata {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  sourcesUsed: number;
  generatedAt: string;
}

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';
export type VideoFormat = 'reels' | 'tiktok' | 'shorts';

export interface GeneratedContent {
  content: string;
  metadata: ContentMetadata;
}
```

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline system, enabling AI coding agents to effectively assist developers in automating content creation workflows.
