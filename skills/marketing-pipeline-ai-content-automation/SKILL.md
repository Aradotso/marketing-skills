---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated content
  - generate videos from AI-written content automatically
  - create content pipeline with Claude and Remotion
  - automate social media content from research to video
  - build AI content automation system with TypeScript
  - integrate AI content generation with video rendering
  - set up automated marketing content workflow
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline AI Content Automation is a complete end-to-end content production system that automatically:
- **Researches** trending topics by crawling news sources (TechCrunch, a16z, Twitter, LinkedIn)
- **Generates** multi-format content (blog posts, case studies, how-tos) in multiple languages using Claude 3 or OpenAI
- **Renders** videos and infographics using Remotion for social media platforms

Built with Next.js and TypeScript, this pipeline transforms a single keyword into publication-ready content and videos optimized for Reels, TikTok, and Shorts.

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for Research/Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration (optional)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Visit `http://localhost:3000` to access the application.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   └── video/       # Remotion video generation
│   ├── services/        # API services
│   └── types/           # TypeScript definitions
├── public/              # Static assets
├── remotion/           # Remotion video templates
└── package.json
```

## Core Functionality

### 1. Content Research Module

Automatically crawl and analyze recent content from multiple sources:

```typescript
// src/lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchResult {
  title: string;
  url: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}

export async function researchTopic(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const rapidApi = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const articles = await rapidApi.searchArticles({
      query: keyword,
      source: source,
      timeRange: '24h'
    });
    
    results.push(...articles);
  }
  
  // Filter and rank by relevance
  return results.sort((a, b) => 
    calculateRelevanceScore(b, keyword) - calculateRelevanceScore(a, keyword)
  ).slice(0, 10);
}

function calculateRelevanceScore(article: ResearchResult, keyword: string): number {
  const titleMatch = article.title.toLowerCase().includes(keyword.toLowerCase()) ? 2 : 0;
  const contentMatch = article.content.toLowerCase().split(keyword.toLowerCase()).length - 1;
  return titleMatch + contentMatch;
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentGenerationOptions {
  keyword: string;
  research: ResearchResult[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
  provider?: 'claude' | 'openai';
}

export async function generateContent(options: ContentGenerationOptions): Promise<string> {
  const { keyword, research, format, language, tone, provider = 'claude' } = options;
  
  const prompt = buildPrompt(keyword, research, format, language, tone);
  
  if (provider === 'claude') {
    return await generateWithClaude(prompt);
  } else {
    return await generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!,
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' ? message.content[0].text : '';
}

async function generateWithOpenAI(prompt: string): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!,
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'user',
      content: prompt
    }],
    max_tokens: 4096,
  });
  
  return completion.choices[0]?.message?.content || '';
}

function buildPrompt(
  keyword: string,
  research: ResearchResult[],
  format: ContentFormat,
  language: Language,
  tone: Tone
): string {
  const researchSummary = research.map(r => 
    `- ${r.title} (${r.source}): ${r.insights.join(', ')}`
  ).join('\n');
  
  const formatInstructions = {
    'toplist': 'Create a numbered list article with at least 7-10 items',
    'pov': 'Write from a unique perspective or opinion angle',
    'case-study': 'Analyze with real examples, data, and outcomes',
    'how-to': 'Provide step-by-step actionable instructions'
  };
  
  const toneInstructions = {
    'expert': 'Use authoritative, professional language with industry terminology',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include light humor and engaging storytelling'
  };
  
  return `You are a content marketing expert. Create a ${format} article about "${keyword}" in ${language === 'en' ? 'English' : 'Vietnamese'}.

RESEARCH DATA:
${researchSummary}

REQUIREMENTS:
- Format: ${formatInstructions[format]}
- Tone: ${toneInstructions[tone]}
- Length: 1500-2000 words
- Include data and statistics from the research
- Add actionable takeaways
- Optimize for SEO with proper headings

Generate the complete article now:`;
}
```

### 3. Video Generation with Remotion

Transform content into videos for social media:

```typescript
// src/lib/video/generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // All are 9:16
  duration: number; // in seconds
}

export async function generateVideo(config: VideoConfig): Promise<string> {
  const { content, title, format, duration } = config;
  
  // Parse content into video segments
  const segments = parseContentToSegments(content, duration);
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const compositionId = 'ContentVideo';
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title,
      segments,
      format
    },
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${format}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      segments,
      format
    },
  });
  
  return outputLocation;
}

function parseContentToSegments(content: string, duration: number): VideoSegment[] {
  // Split content into key points
  const paragraphs = content.split('\n\n').filter(p => p.trim());
  const segmentDuration = duration / paragraphs.length;
  
  return paragraphs.slice(0, 8).map((text, index) => ({
    text: text.trim().substring(0, 150) + '...',
    startTime: index * segmentDuration,
    duration: segmentDuration,
  }));
}

interface VideoSegment {
  text: string;
  startTime: number;
  duration: number;
}
```

### 4. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  segments: Array<{
    text: string;
    startTime: number;
    duration: number;
  }>;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, segments, format }) => {
  const { fps } = useVideoConfig();
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      {/* Title Sequence */}
      <Sequence durationInFrames={fps * 3}>
        <AbsoluteFill style={{
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          padding: 40
        }}>
          <h1 style={{
            fontSize: 48,
            color: '#ffffff',
            textAlign: 'center',
            fontWeight: 'bold'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Content Segments */}
      {segments.map((segment, index) => (
        <Sequence
          key={index}
          from={Math.floor((segment.startTime + 3) * fps)}
          durationInFrames={Math.floor(segment.duration * fps)}
        >
          <AbsoluteFill style={{
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            padding: 60,
            background: `linear-gradient(135deg, #667eea 0%, #764ba2 100%)`
          }}>
            <p style={{
              fontSize: 36,
              color: '#ffffff',
              textAlign: 'center',
              lineHeight: 1.6,
              maxWidth: '80%'
            }}>
              {segment.text}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline API Route

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, generateVideoOutput } = await request.json();
    
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic(keyword);
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      keyword,
      research,
      format,
      language,
      tone,
      provider: 'claude'
    });
    
    let videoUrl = null;
    
    // Step 3: Generate Video (optional)
    if (generateVideoOutput) {
      console.log('🎬 Generating video...');
      const videoPath = await generateVideo({
        content,
        title: keyword,
        format: 'reels',
        duration: 30
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        research: research.slice(0, 5) // Top 5 sources
      }
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

### 6. Frontend Usage Example

```typescript
// src/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const runPipeline = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
          generateVideoOutput: true
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter topic keyword..."
          className="w-full p-3 border rounded-lg"
        />
        
        <button
          onClick={runPipeline}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-3 rounded-lg disabled:opacity-50"
        >
          {loading ? 'Processing...' : 'Generate Content & Video'}
        </button>
      </div>
      
      {result?.data && (
        <div className="mt-8 space-y-6">
          <div>
            <h2 className="text-xl font-semibold mb-2">Generated Content</h2>
            <div className="prose max-w-none bg-gray-50 p-6 rounded-lg">
              {result.data.content}
            </div>
          </div>
          
          {result.data.videoUrl && (
            <div>
              <h2 className="text-xl font-semibold mb-2">Generated Video</h2>
              <video controls className="w-full rounded-lg">
                <source src={result.data.videoUrl} type="video/mp4" />
              </video>
            </div>
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
async function generateMultipleFormats(keyword: string) {
  const research = await researchTopic(keyword);
  
  const formats: ContentFormat[] = ['toplist', 'how-to', 'case-study'];
  const languages: Language[] = ['en', 'vi'];
  
  const results = [];
  
  for (const format of formats) {
    for (const language of languages) {
      const content = await generateContent({
        keyword,
        research,
        format,
        language,
        tone: 'expert'
      });
      
      results.push({ format, language, content });
    }
  }
  
  return results;
}
```

### Scheduled Content Pipeline

```typescript
// src/lib/scheduler/cron.ts
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run daily at 6 AM
  cron.schedule('0 6 * * *', async () => {
    console.log('🤖 Running scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        const research = await researchTopic(keyword);
        const content = await generateContent({
          keyword,
          research,
          format: 'toplist',
          language: 'en',
          tone: 'expert'
        });
        
        // Save to database or publish
        await saveContent({ keyword, content });
      } catch (error) {
        console.error(`Failed for keyword: ${keyword}`, error);
      }
    }
  });
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// Add retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      
      if (error.status === 429) {
        await new Promise(resolve => setTimeout(resolve, delay * Math.pow(2, i)));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => generateContent(options));
```

### Video Rendering Memory Issues

For large video generations, increase Node.js memory:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Research Crawler Blocked

If sources block your crawler, add user agent rotation:

```typescript
const userAgents = [
  'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
];

async function fetchWithRotation(url: string) {
  const randomUA = userAgents[Math.floor(Math.random() * userAgents.length)];
  
  return fetch(url, {
    headers: {
      'User-Agent': randomUA
    }
  });
}
```

## Performance Optimization

### Parallel Processing

```typescript
async function optimizedPipeline(keywords: string[]) {
  // Research in parallel
  const researchResults = await Promise.all(
    keywords.map(k => researchTopic(k))
  );
  
  // Generate content in parallel (with rate limiting)
  const contents = await Promise.all(
    keywords.map((keyword, i) => 
      generateContent({
        keyword,
        research: researchResults[i],
        format: 'toplist',
        language: 'en',
        tone: 'expert'
      })
    )
  );
  
  return contents;
}
```

This skill enables AI agents to help developers build complete automated content marketing pipelines with research, AI generation, and video production capabilities.
