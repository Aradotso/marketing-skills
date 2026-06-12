---
name: marketing-pipeline-share-automation
description: AI-powered content pipeline that automates research, scriptwriting, and video generation for marketing content
triggers:
  - how do I set up the marketing content automation pipeline
  - automate content research and video generation
  - use AI to create marketing content from research to video
  - configure the content pipeline with Claude and OpenAI
  - generate automated marketing videos with Remotion
  - create multilingual content posts automatically
  - build an AI content creation workflow
  - set up automated social media content generation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **marketing-pipeline-share**, a complete AI-powered content automation system that handles research, scriptwriting, posting, and video generation. The pipeline crawls news sources, generates content in multiple formats and languages, and automatically renders videos using Remotion.

## What This Project Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter, LinkedIn for fresh data
2. **AI Content Generation**: Uses Claude 3 and OpenAI to create content in multiple formats (toplist, POV, case studies, how-to)
3. **Multilingual Support**: Generates parallel English and Vietnamese versions
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Multi-platform Output**: Exports optimized content for Reels, TikTok, Shorts

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
```

## Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if needed)
DATABASE_URL=your_database_connection_string

# Remotion Config
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

Reference environment variables in code:

```typescript
const anthropicKey = process.env.ANTHROPIC_API_KEY;
const openaiKey = process.env.OPENAI_API_KEY;
const rapidApiKey = process.env.RAPIDAPI_KEY;
```

## Key Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Crawling

```typescript
// lib/crawler/news-scraper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}

export async function crawlTechNews(
  keyword: string,
  timeframe: string = '24h'
): Promise<NewsArticle[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const options = {
    method: 'GET',
    url: 'https://news-api.example.com/search',
    params: {
      q: keyword,
      timeframe: timeframe,
      sources: 'techcrunch,a16z,twitter'
    },
    headers: {
      'X-RapidAPI-Key': rapidApiKey,
      'X-RapidAPI-Host': 'news-api.example.com'
    }
  };

  const response = await axios.request(options);
  return response.data.articles;
}

export async function extractInsights(articles: NewsArticle[]): Promise<string[]> {
  // Extract key insights from crawled articles
  return articles.map(article => {
    return `${article.source}: ${article.title} - ${article.content.substring(0, 200)}`;
  });
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: string[];
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const systemPrompt = buildSystemPrompt(request.format, request.tone);
  const userPrompt = buildUserPrompt(request.keyword, request.researchData);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    temperature: 0.7,
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

function buildSystemPrompt(format: string, tone: string): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a unique perspective, sharing insights and opinions',
    'case-study': 'Structure as a detailed case study with problem, solution, results',
    'how-to': 'Create step-by-step tutorial with actionable instructions'
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Add wit and humor while maintaining informativeness'
  };

  return `You are an expert content creator. ${formatInstructions[format]}. ${toneInstructions[tone]}.`;
}

function buildUserPrompt(keyword: string, researchData: string[]): string {
  return `
Topic: ${keyword}

Recent Research Data:
${researchData.join('\n\n')}

Create comprehensive, engaging content based on this research data.
Include specific examples, data points, and actionable insights.
  `.trim();
}
```

### 3. OpenAI Integration for Variation

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentVariation(
  originalContent: string,
  targetLanguage: 'en' | 'vi'
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: targetLanguage === 'vi' 
          ? 'You are a Vietnamese content writer. Adapt the content while maintaining key messages.'
          : 'You are an English content writer. Refine and optimize the content.'
      },
      {
        role: 'user',
        content: originalContent
      }
    ],
    temperature: 0.8,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}

export async function generateSocialCaptions(
  content: string,
  platform: 'facebook' | 'twitter' | 'linkedin'
): Promise<string> {
  const platformGuidelines = {
    'facebook': 'Engaging, conversational, 1-2 paragraphs',
    'twitter': 'Concise, impactful, max 280 characters',
    'linkedin': 'Professional, thought-leadership style'
  };

  const completion = await openai.chat.completions.create({
    model: 'gpt-3.5-turbo',
    messages: [
      {
        role: 'system',
        content: `Create a ${platform} post caption. Style: ${platformGuidelines[platform]}`
      },
      {
        role: 'user',
        content: `Summarize this content for ${platform}:\n\n${content}`
      }
    ],
    temperature: 0.9
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  points: string[];
  duration: number;
  format: '16:9' | '9:16' | '1:1';
}

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
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
      points: config.points,
      theme: 'modern'
    }
  });

  const dimensions = getVideoDimensions(config.format);

  await renderMedia({
    composition: {
      ...composition,
      width: dimensions.width,
      height: dimensions.height,
      durationInFrames: config.duration * 30, // 30 fps
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      points: config.points,
    },
  });

  return outputPath;
}

function getVideoDimensions(format: string): { width: number; height: number } {
  const formats = {
    '16:9': { width: 1920, height: 1080 },
    '9:16': { width: 1080, height: 1920 }, // Reels, TikTok, Shorts
    '1:1': { width: 1080, height: 1080 }
  };
  return formats[format];
}
```

```tsx
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  theme?: 'modern' | 'minimal';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  points,
  theme = 'modern' 
}) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{ 
          opacity: titleOpacity,
          padding: 60,
          color: 'white',
          fontSize: 72,
          fontWeight: 'bold',
          textAlign: 'center'
        }}>
          {title}
        </div>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence 
          key={index} 
          from={60 + index * 90} 
          durationInFrames={90}
        >
          <PointSlide point={point} index={index + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ point: string; index: number }> = ({ point, index }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15], [0, 1]);
  
  return (
    <div style={{ 
      opacity,
      padding: 80,
      color: 'white',
      fontSize: 48
    }}>
      <div style={{ fontSize: 96, marginBottom: 40 }}>{index}</div>
      <div>{point}</div>
    </div>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/content-pipeline.ts
import { crawlTechNews, extractInsights } from '../crawler/news-scraper';
import { generateContent } from '../ai/claude-generator';
import { generateContentVariation, generateSocialCaptions } from '../ai/openai-generator';
import { renderContentVideo } from '../video/remotion-renderer';

interface PipelineResult {
  contentEn: string;
  contentVi: string;
  captions: {
    facebook: string;
    twitter: string;
    linkedin: string;
  };
  videoPath: string;
}

export async function runContentPipeline(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
): Promise<PipelineResult> {
  
  // Step 1: Research
  console.log('🔍 Crawling news and research data...');
  const articles = await crawlTechNews(keyword);
  const insights = await extractInsights(articles);
  
  // Step 2: Generate English content with Claude
  console.log('✍️ Generating English content...');
  const contentEn = await generateContent({
    keyword,
    format,
    tone: 'expert',
    language: 'en',
    researchData: insights
  });
  
  // Step 3: Generate Vietnamese version with OpenAI
  console.log('🌏 Creating Vietnamese version...');
  const contentVi = await generateContentVariation(contentEn, 'vi');
  
  // Step 4: Generate social media captions
  console.log('📱 Generating social captions...');
  const captions = {
    facebook: await generateSocialCaptions(contentEn, 'facebook'),
    twitter: await generateSocialCaptions(contentEn, 'twitter'),
    linkedin: await generateSocialCaptions(contentEn, 'linkedin')
  };
  
  // Step 5: Extract key points for video
  const points = extractKeyPoints(contentEn);
  
  // Step 6: Render video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo({
    title: keyword,
    points,
    duration: 60,
    format: '9:16'
  }, `./output/${keyword.replace(/\s+/g, '-')}.mp4`);
  
  console.log('✅ Pipeline complete!');
  
  return {
    contentEn,
    contentVi,
    captions,
    videoPath
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - in production, use AI to extract key points
  const lines = content.split('\n').filter(line => line.trim().length > 0);
  return lines.slice(0, 5).map(line => line.replace(/^\d+\.\s*/, '').substring(0, 100));
}
```

### 6. Next.js API Route

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format } = await request.json();
    
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields: keyword, format' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword, format);
    
    return NextResponse.json({
      success: true,
      data: result
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### 7. React Component Usage

```tsx
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format })
      });
      
      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-8">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 w-full mb-4"
      />
      
      <select
        value={format}
        onChange={(e) => setFormat(e.target.value as any)}
        className="border p-2 w-full mb-4"
      >
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How To</option>
      </select>
      
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-8 space-y-4">
          <div>
            <h3 className="font-bold">English Content:</h3>
            <p className="whitespace-pre-wrap">{result.contentEn}</p>
          </div>
          <div>
            <h3 className="font-bold">Vietnamese Content:</h3>
            <p className="whitespace-pre-wrap">{result.contentVi}</p>
          </div>
          <div>
            <h3 className="font-bold">Social Captions:</h3>
            <pre>{JSON.stringify(result.captions, null, 2)}</pre>
          </div>
          <div>
            <h3 className="font-bold">Video:</h3>
            <video src={result.videoPath} controls className="w-full max-w-md" />
          </div>
        </div>
      )}
    </div>
  );
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

# Render videos (if separate script)
npm run render
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch/batch-generator.ts
export async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runContentPipeline(keyword, 'toplist')
    )
  );
  
  return results;
}
```

### Scheduled Content Generation

```typescript
// lib/scheduler/cron-jobs.ts
import cron from 'node-cron';

export function setupContentScheduler() {
  // Run daily at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('Running scheduled content generation...');
    const trendingTopics = await fetchTrendingTopics();
    await generateBatchContent(trendingTopics);
  });
}
```

## Troubleshooting

### API Rate Limits

If hitting rate limits:

```typescript
// lib/utils/rate-limiter.ts
export async function rateLimitedRequest<T>(
  fn: () => Promise<T>,
  delay: number = 1000
): Promise<T> {
  await new Promise(resolve => setTimeout(resolve, delay));
  return fn();
}
```

### Video Rendering Timeout

Increase timeout for large videos:

```typescript
await renderMedia({
  // ... other options
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Memory Issues with Large Content

Process in chunks:

```typescript
export async function processLargeContent(content: string) {
  const chunks = content.match(/.{1,2000}/g) || [];
  const results = [];
  
  for (const chunk of chunks) {
    const result = await processChunk(chunk);
    results.push(result);
  }
  
  return results.join('\n');
}
```

### Claude/OpenAI API Errors

Add retry logic:

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

This skill provides comprehensive guidance for using the marketing-pipeline-share automation system to create AI-powered content workflows.
