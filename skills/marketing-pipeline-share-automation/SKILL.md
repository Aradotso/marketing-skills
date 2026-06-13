---
name: marketing-pipeline-share-automation
description: Automated content pipeline system for research, scripting, posting, and video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline for auto-posting
  - generate videos from blog content automatically
  - crawl news sources and create content
  - use Claude and OpenAI for content automation
  - create multilingual content with AI pipeline
  - automate social media video generation
  - build content research and video workflow
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **Marketing Pipeline Share**, an end-to-end automated content pipeline that handles research, scripting, posting, and video generation. The system crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates content in multiple formats and languages using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

Marketing Pipeline Share is a TypeScript/Next.js application that:

- **Auto-crawls** real-time data from major news sources (last 24h)
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using AI
- **Supports multilingual** content (English & Vietnamese) with tone customization
- **Renders videos** and infographics automatically using Remotion
- **Optimizes output** for multiple platforms (Reels, TikTok, Shorts)

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

### Environment Setup

Create a `.env.local` file in the root directory:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# News APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Remotion (Video Generation)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_url

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run remotion
```

## Core Architecture

### 1. Research Module (Auto-Crawling)

The research module crawls news sources and extracts insights:

```typescript
// services/research/crawler.ts
import { Article } from '@/types';

interface CrawlerConfig {
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  keywords: string[];
}

export async function crawlNews(config: CrawlerConfig): Promise<Article[]> {
  const { sources, timeRange, keywords } = config;
  
  const articles: Article[] = [];
  
  for (const source of sources) {
    const response = await fetch(`https://api.rapidapi.com/news/${source}`, {
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        'X-RapidAPI-Host': 'news-api.rapidapi.com'
      },
      method: 'GET'
    });
    
    const data = await response.json();
    articles.push(...filterByKeywords(data.articles, keywords));
  }
  
  return articles;
}

function filterByKeywords(articles: any[], keywords: string[]): Article[] {
  return articles.filter(article => 
    keywords.some(keyword => 
      article.title.toLowerCase().includes(keyword.toLowerCase()) ||
      article.description.toLowerCase().includes(keyword.toLowerCase())
    )
  ).map(article => ({
    title: article.title,
    description: article.description,
    url: article.url,
    publishedAt: article.publishedAt,
    source: article.source.name
  }));
}
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI:

```typescript
// services/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  articles: Article[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
  provider: 'claude' | 'openai';
}

export async function generateContent(options: GenerateContentOptions): Promise<string> {
  const { articles, format, language, tone, provider } = options;
  
  const prompt = buildPrompt(articles, format, language, tone);
  
  if (provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' ? message.content[0].text : '';
}

async function generateWithOpenAI(prompt: string): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'user',
      content: prompt
    }],
    max_tokens: 4000
  });
  
  return completion.choices[0].message.content || '';
}

function buildPrompt(
  articles: Article[], 
  format: ContentFormat, 
  language: Language, 
  tone: Tone
): string {
  const formatInstructions = {
    'toplist': 'Create a top 10 list article',
    'pov': 'Write a perspective/opinion piece',
    'case-study': 'Develop a detailed case study',
    'how-to': 'Create a step-by-step how-to guide'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable tone',
    'humorous': 'Include wit and engaging humor'
  };
  
  const articleSummaries = articles.map(a => 
    `Title: ${a.title}\nSummary: ${a.description}\nSource: ${a.source}`
  ).join('\n\n');
  
  return `
You are a content creator. ${formatInstructions[format]} based on these recent news articles.

${articleSummaries}

Requirements:
- Language: ${language === 'en' ? 'English' : 'Vietnamese'}
- Tone: ${toneInstructions[tone]}
- Include data and specific insights from the articles
- Make it engaging and SEO-optimized
- Length: 1000-1500 words
`;
}
```

### 3. Video Generation with Remotion

Create video content from generated text:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  points, 
  brandColor 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity: titleOpacity
        }}>
          <h1 style={{ 
            color: brandColor, 
            fontSize: 80,
            textAlign: 'center',
            padding: '0 100px'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
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

const PointSlide: React.FC<{ point: string; index: number; color: string }> = ({
  point,
  index,
  color
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const slideIn = Math.min(1, frame / (fps * 0.3));
  
  return (
    <AbsoluteFill style={{
      justifyContent: 'center',
      alignItems: 'center',
      transform: `translateX(${(1 - slideIn) * 100}px)`,
      opacity: slideIn
    }}>
      <div style={{ padding: '0 100px', maxWidth: 1200 }}>
        <h2 style={{ color, fontSize: 60, marginBottom: 30 }}>
          {index}. 
        </h2>
        <p style={{ color: '#fff', fontSize: 40, lineHeight: 1.5 }}>
          {point}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

Register composition:

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Top 10 AI Trends 2024',
          points: [
            'AI agents are becoming mainstream',
            'Multimodal models dominate the market',
            'Edge AI enables real-time processing'
          ],
          brandColor: '#FF6B6B'
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

### 4. Complete Pipeline Orchestration

Combine all modules into a single workflow:

```typescript
// lib/pipeline.ts
import { crawlNews } from '@/services/research/crawler';
import { generateContent, ContentFormat, Language, Tone } from '@/services/ai/contentGenerator';
import { renderVideo } from '@/services/video/renderer';

interface PipelineConfig {
  keyword: string;
  sources: string[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
  aiProvider: 'claude' | 'openai';
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('🔍 Step 1: Crawling news sources...');
  const articles = await crawlNews({
    sources: config.sources,
    timeRange: '24h',
    keywords: [config.keyword]
  });
  
  if (articles.length === 0) {
    throw new Error('No articles found for the given keyword');
  }
  
  console.log(`✅ Found ${articles.length} articles`);
  
  console.log('🤖 Step 2: Generating content with AI...');
  const content = await generateContent({
    articles,
    format: config.format,
    language: config.language,
    tone: config.tone,
    provider: config.aiProvider
  });
  
  console.log('✅ Content generated');
  
  let videoUrl = null;
  if (config.generateVideo) {
    console.log('🎬 Step 3: Rendering video...');
    videoUrl = await renderVideo({
      content,
      format: config.format
    });
    console.log('✅ Video rendered:', videoUrl);
  }
  
  return {
    content,
    videoUrl,
    articles,
    metadata: {
      keyword: config.keyword,
      format: config.format,
      language: config.language,
      articleCount: articles.length,
      generatedAt: new Date().toISOString()
    }
  };
}
```

### 5. API Route Example

Create a Next.js API route to trigger the pipeline:

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const {
      keyword,
      sources = ['techcrunch', 'a16z'],
      format = 'toplist',
      language = 'en',
      tone = 'expert',
      aiProvider = 'claude',
      generateVideo = false
    } = body;
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      sources,
      format,
      language,
      tone,
      aiProvider,
      generateVideo
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Generate Content in Multiple Languages

```typescript
const results = await Promise.all([
  runContentPipeline({
    keyword: 'AI automation',
    sources: ['techcrunch'],
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    aiProvider: 'claude',
    generateVideo: false
  }),
  runContentPipeline({
    keyword: 'AI automation',
    sources: ['techcrunch'],
    format: 'toplist',
    language: 'vi',
    tone: 'friendly',
    aiProvider: 'openai',
    generateVideo: false
  })
]);

const [englishContent, vietnameseContent] = results;
```

### Batch Video Generation

```typescript
const formats: ContentFormat[] = ['toplist', 'how-to', 'case-study'];

for (const format of formats) {
  const result = await runContentPipeline({
    keyword: 'marketing automation',
    sources: ['techcrunch', 'a16z'],
    format,
    language: 'en',
    tone: 'friendly',
    aiProvider: 'claude',
    generateVideo: true
  });
  
  console.log(`Video for ${format}:`, result.videoUrl);
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// utils/retry.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const articles = await withRetry(() => crawlNews(config), 3, 2000);
```

### Video Rendering Fails

Check Remotion logs and ensure sufficient memory:

```bash
# Increase Node memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion
```

### Missing Environment Variables

Validate environment variables at startup:

```typescript
// lib/validateEnv.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}
```

This skill enables AI agents to guide developers through setting up and using the complete marketing automation pipeline effectively.
