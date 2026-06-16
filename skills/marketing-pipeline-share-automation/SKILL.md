---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline for content automation
  - generate videos from blog posts automatically
  - auto-research and write content with Claude
  - create content pipeline with remotion
  - build AI-powered content workflow
  - scrape news and generate social media content
  - automate content from research to video
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers use **Marketing Pipeline Share**, a complete AI-powered content automation system that handles research, scriptwriting, and video generation. The pipeline crawls news sources, generates multilingual content using Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

Marketing Pipeline Share is an end-to-end content automation pipeline that:

- **Auto-scrapes** trending news from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 or OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms
- **Exports optimized assets** for Reels, TikTok, Shorts

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Remotion (Video Generation)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
pnpm dev
```

The app will be available at `http://localhost:3000`

## Core Architecture

```typescript
// src/types/pipeline.ts
export interface ContentPipeline {
  keyword: string;
  language: 'en' | 'vi' | 'both';
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  includeVideo: boolean;
}

export interface ResearchResult {
  sources: Array<{
    title: string;
    url: string;
    publishedAt: string;
    excerpt: string;
  }>;
  insights: string[];
  trends: string[];
}

export interface GeneratedContent {
  title: string;
  body: string;
  language: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    keywords: string[];
  };
}
```

## Research Pipeline

### Auto-Crawl News Sources

```typescript
// src/lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

export async function crawlRecentNews(keyword: string, hours: number = 24) {
  const rapidapi = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const sources = [
    'techcrunch.com',
    'a16z.com',
    'twitter.com',
    'linkedin.com'
  ];
  
  const results = await Promise.all(
    sources.map(source => 
      rapidapi.searchNews({
        q: keyword,
        domains: source,
        from: new Date(Date.now() - hours * 3600000).toISOString(),
        language: 'en',
        sortBy: 'publishedAt'
      })
    )
  );
  
  return results.flat().map(article => ({
    title: article.title,
    url: article.url,
    publishedAt: article.publishedAt,
    excerpt: article.description,
    source: article.source.name
  }));
}
```

### Extract Insights with Claude

```typescript
// src/lib/research/insights.ts
import Anthropic from '@anthropic-ai/sdk';

export async function extractInsights(articles: any[]): Promise<ResearchResult> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const articleSummaries = articles
    .map(a => `[${a.source}] ${a.title}\n${a.excerpt}`)
    .join('\n\n');
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2048,
    messages: [{
      role: 'user',
      content: `Analyze these recent articles and extract:
1. Key insights (3-5 main points)
2. Emerging trends (2-3 patterns)
3. Data-backed conclusions

Articles:
${articleSummaries}`
    }]
  });
  
  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
  
  // Parse structured output
  return parseInsightsFromClaude(content, articles);
}

function parseInsightsFromClaude(text: string, sources: any[]): ResearchResult {
  // Extract insights and trends from Claude's response
  const insights = text.match(/(?<=Key insights:)(.*?)(?=Emerging trends:|$)/s)?.[0]
    .split('\n')
    .filter(line => line.trim().match(/^[\d\-\*]/))
    .map(line => line.replace(/^[\d\-\*\s]+/, '').trim()) || [];
  
  const trends = text.match(/(?<=Emerging trends:)(.*?)(?=Data-backed|$)/s)?.[0]
    .split('\n')
    .filter(line => line.trim().match(/^[\d\-\*]/))
    .map(line => line.replace(/^[\d\-\*\s]+/, '').trim()) || [];
  
  return { sources, insights, trends };
}
```

## Content Generation

### Generate Content with Format Templates

```typescript
// src/lib/content/generator.ts
import OpenAI from 'openai';

export async function generateContent(
  research: ResearchResult,
  config: ContentPipeline
): Promise<GeneratedContent> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const systemPrompt = buildSystemPrompt(config.format, config.tone);
  const userPrompt = buildUserPrompt(research, config.keyword, config.language);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  const content = completion.choices[0].message.content || '';
  
  return {
    title: extractTitle(content),
    body: extractBody(content),
    language: config.language,
    metadata: {
      wordCount: content.split(/\s+/).length,
      readingTime: Math.ceil(content.split(/\s+/).length / 200),
      keywords: [config.keyword, ...research.trends.slice(0, 3)]
    }
  };
}

function buildSystemPrompt(format: string, tone: string): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a unique perspective with personal insights and opinions',
    'case-study': 'Structure as a detailed case study with problem, solution, results',
    'how-to': 'Provide step-by-step instructions with actionable advice'
  };
  
  const toneInstructions = {
    'expert': 'Use authoritative, professional language with industry terminology',
    'friendly': 'Write conversationally but informatively, like talking to a colleague',
    'humorous': 'Add wit and humor while maintaining credibility'
  };
  
  return `You are an expert content writer. ${formatInstructions[format]}. 
${toneInstructions[tone]}. Include data and statistics from research.`;
}

function buildUserPrompt(research: ResearchResult, keyword: string, lang: string): string {
  return `Write a comprehensive article about "${keyword}" based on this research:

Insights:
${research.insights.map((i, idx) => `${idx + 1}. ${i}`).join('\n')}

Trends:
${research.trends.map((t, idx) => `${idx + 1}. ${t}`).join('\n')}

Sources:
${research.sources.slice(0, 5).map(s => `- ${s.title} (${s.source})`).join('\n')}

Language: ${lang === 'vi' ? 'Vietnamese' : 'English'}
Include title, introduction, main sections, and conclusion.`;
}

function extractTitle(content: string): string {
  const match = content.match(/^#\s+(.+)$/m);
  return match ? match[1] : content.split('\n')[0].substring(0, 100);
}

function extractBody(content: string): string {
  return content.replace(/^#\s+.+$/m, '').trim();
}
```

### Bilingual Content Generation

```typescript
// src/lib/content/bilingual.ts
export async function generateBilingualContent(
  research: ResearchResult,
  config: Omit<ContentPipeline, 'language'>
): Promise<{ en: GeneratedContent; vi: GeneratedContent }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, { ...config, language: 'en' }),
    generateContent(research, { ...config, language: 'vi' })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

## Video Generation with Remotion

### Setup Remotion Composition

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

export interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = Math.min(1, frame / (fps * 0.5));
  const pointsStartFrame = fps * 2;
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Section */}
      <AbsoluteFill
        style={{
          justifyContent: 'center',
          alignItems: 'center',
          opacity: titleOpacity
        }}
      >
        <h1 style={{
          fontSize: 72,
          color: brandColor,
          textAlign: 'center',
          padding: '0 60px',
          fontWeight: 'bold'
        }}>
          {title}
        </h1>
      </AbsoluteFill>
      
      {/* Key Points */}
      {frame > pointsStartFrame && (
        <AbsoluteFill style={{ padding: 60, justifyContent: 'center' }}>
          {keyPoints.map((point, index) => {
            const pointFrame = pointsStartFrame + (index * fps * 1.5);
            const pointOpacity = Math.max(0, Math.min(1, 
              (frame - pointFrame) / (fps * 0.3)
            ));
            
            if (frame < pointFrame) return null;
            
            return (
              <div
                key={index}
                style={{
                  opacity: pointOpacity,
                  marginBottom: 40,
                  transform: `translateY(${(1 - pointOpacity) * 20}px)`
                }}
              >
                <h2 style={{
                  fontSize: 48,
                  color: '#fff',
                  marginBottom: 10
                }}>
                  {index + 1}. {point}
                </h2>
              </div>
            );
          })}
        </AbsoluteFill>
      )}
    </AbsoluteFill>
  );
};
```

### Render Video from Content

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: GeneratedContent,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Extract key points from content
  const keyPoints = extractKeyPoints(content.body);
  
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      keyPoints: keyPoints.slice(0, 5),
      brandColor: '#3b82f6'
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.props,
  });
  
  return outputPath;
}

function extractKeyPoints(body: string): string[] {
  // Extract bullet points or numbered items
  const matches = body.match(/(?:^|\n)[\d\-\*]\s*(.+?)(?=\n|$)/gm);
  return matches 
    ? matches.map(m => m.replace(/^[\d\-\*\s]+/, '').trim())
    : body.split('\n').filter(line => line.length > 20 && line.length < 150).slice(0, 5);
}
```

### Platform-Specific Video Exports

```typescript
// src/lib/video/platforms.ts
export const PLATFORM_SPECS = {
  reels: { width: 1080, height: 1920, fps: 30, duration: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30, duration: 60 },
  shorts: { width: 1080, height: 1920, fps: 30, duration: 60 },
  linkedin: { width: 1920, height: 1080, fps: 30, duration: 120 }
};

export async function renderForPlatform(
  content: GeneratedContent,
  platform: keyof typeof PLATFORM_SPECS,
  outputDir: string
): Promise<string> {
  const spec = PLATFORM_SPECS[platform];
  const outputPath = path.join(outputDir, `${platform}-${Date.now()}.mp4`);
  
  // This would integrate with Remotion's config
  return renderContentVideo(content, outputPath);
}
```

## Complete Pipeline Execution

```typescript
// src/lib/pipeline/executor.ts
export async function executeContentPipeline(
  config: ContentPipeline
): Promise<{
  research: ResearchResult;
  content: GeneratedContent | { en: GeneratedContent; vi: GeneratedContent };
  video?: string;
}> {
  // Step 1: Research
  console.log(`🔍 Researching: ${config.keyword}`);
  const articles = await crawlRecentNews(config.keyword, 24);
  const research = await extractInsights(articles);
  
  // Step 2: Content Generation
  console.log(`✍️ Generating content in ${config.language} format`);
  const content = config.language === 'both'
    ? await generateBilingualContent(research, config)
    : await generateContent(research, config);
  
  // Step 3: Video Generation (optional)
  let videoPath: string | undefined;
  if (config.includeVideo) {
    console.log('🎬 Rendering video...');
    const contentForVideo = config.language === 'both' 
      ? (content as { en: GeneratedContent }).en 
      : content as GeneratedContent;
    videoPath = await renderContentVideo(
      contentForVideo,
      `./output/video-${Date.now()}.mp4`
    );
  }
  
  console.log('✅ Pipeline complete!');
  return { research, content, video: videoPath };
}
```

### API Route Example

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { executeContentPipeline } from '@/lib/pipeline/executor';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await executeContentPipeline({
      keyword: body.keyword,
      language: body.language || 'en',
      format: body.format || 'toplist',
      tone: body.tone || 'expert',
      includeVideo: body.includeVideo || false
    });
    
    return NextResponse.json({
      success: true,
      data: result
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

### Frontend Usage Example

```typescript
// src/app/components/PipelineForm.tsx
'use client';

import { useState } from 'react';

export default function PipelineForm() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);
    
    const formData = new FormData(e.currentTarget);
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        language: formData.get('language'),
        format: formData.get('format'),
        tone: formData.get('tone'),
        includeVideo: formData.get('includeVideo') === 'on'
      })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  };
  
  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        name="keyword"
        placeholder="Enter keyword..."
        required
        className="w-full p-2 border rounded"
      />
      
      <select name="language" className="w-full p-2 border rounded">
        <option value="en">English</option>
        <option value="vi">Vietnamese</option>
        <option value="both">Both</option>
      </select>
      
      <select name="format" className="w-full p-2 border rounded">
        <option value="toplist">Top List</option>
        <option value="pov">POV</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-To</option>
      </select>
      
      <select name="tone" className="w-full p-2 border rounded">
        <option value="expert">Expert</option>
        <option value="friendly">Friendly</option>
        <option value="humorous">Humorous</option>
      </select>
      
      <label className="flex items-center gap-2">
        <input type="checkbox" name="includeVideo" />
        Generate video
      </label>
      
      <button
        type="submit"
        disabled={loading}
        className="w-full bg-blue-500 text-white p-2 rounded"
      >
        {loading ? 'Processing...' : 'Generate Content'}
      </button>
      
      {result && (
        <pre className="mt-4 p-4 bg-gray-100 rounded overflow-auto">
          {JSON.stringify(result, null, 2)}
        </pre>
      )}
    </form>
  );
}
```

## Common Patterns

### Scheduled Content Generation

```typescript
// src/lib/scheduler/cron.ts
import cron from 'node-cron';

export function scheduleContentGeneration(
  keywords: string[],
  schedule: string = '0 9 * * *' // Daily at 9 AM
) {
  cron.schedule(schedule, async () => {
    console.log('Running scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        await executeContentPipeline({
          keyword,
          language: 'both',
          format: 'toplist',
          tone: 'expert',
          includeVideo: true
        });
      } catch (error) {
        console.error(`Failed for keyword: ${keyword}`, error);
      }
    }
  });
}
```

### Batch Processing

```typescript
// src/lib/pipeline/batch.ts
export async function batchProcessKeywords(
  keywords: string[],
  config: Omit<ContentPipeline, 'keyword'>
): Promise<Array<{ keyword: string; success: boolean; data?: any; error?: string }>> {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      executeContentPipeline({ ...config, keyword })
    )
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    success: result.status === 'fulfilled',
    data: result.status === 'fulfilled' ? result.value : undefined,
    error: result.status === 'rejected' ? result.reason.message : undefined
  }));
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// src/lib/utils/rate-limiter.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

For Remotion rendering errors, ensure ffmpeg is installed:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows (use chocolatey)
choco install ffmpeg
```

### Memory Issues with Large Batches

Use streaming for large datasets:

```typescript
// Process in chunks
const CHUNK_SIZE = 5;
for (let i = 0; i < keywords.length; i += CHUNK_SIZE) {
  const chunk = keywords.slice(i, i + CHUNK_SIZE);
  await batchProcessKeywords(chunk, config);
  // Allow garbage collection
  await new Promise(resolve => setTimeout(resolve, 1000));
}
```

### Claude/OpenAI Timeout

Increase timeout for long-running requests:

```typescript
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  timeout: 60000, // 60 seconds
  maxRetries: 2
});
```

## Best Practices

1. **Cache research results** to avoid redundant API calls
2. **Use webhooks** for long-running video renders instead of blocking
3. **Validate content quality** with a scoring system before publishing
4. **Store generated content** in a database with metadata for analytics
5. **Monitor API costs** - set up billing alerts for AI providers
