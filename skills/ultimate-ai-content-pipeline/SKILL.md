---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline with Claude and OpenAI
  - generate videos from blog posts automatically
  - crawl news sources and create content with AI
  - build automated marketing content workflow
  - create multilingual content with AI and render videos
  - research trending topics and generate social media content
  - develop end-to-end content automation system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end automated content creation pipeline that transforms keywords into polished articles and videos. It crawls recent news from sources like TechCrunch, Twitter/X, and LinkedIn, uses Claude/OpenAI to generate content in multiple formats and languages, and leverages Remotion to render videos optimized for social platforms.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major news sources (last 24 hours)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual Support**: Generates content in both English and Vietnamese with customizable tone
- **Video Generation**: Automatically renders infographics and short videos from articles using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Database (if using)
DATABASE_URL=your_database_url
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering (if separate)
npm run remotion
```

## Project Structure

```typescript
/app                 # Next.js app directory
  /api              # API routes for content generation
  /components       # React components
  /lib              # Utility functions and API clients
/remotion           # Remotion video templates
/public             # Static assets
```

## Core Features & Usage

### 1. Research & Content Scraping

```typescript
// lib/research/scraper.ts
import { RapidAPIClient } from './clients';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  summary: string;
}

export async function scrapeRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'twitter', 'linkedin']
): Promise<NewsArticle[]> {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    const results = await rapidAPI.search({
      query: keyword,
      source: source,
      timeRange: '24h'
    });
    
    articles.push(...results);
  }
  
  return articles.sort((a, b) => 
    new Date(b.publishedAt).getTime() - new Date(a.publishedAt).getTime()
  );
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: NewsArticle[];
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const systemPrompt = buildSystemPrompt(request.format, request.tone, request.language);
  const userPrompt = buildUserPrompt(request.keyword, request.researchData);
  
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

function buildSystemPrompt(format: string, tone: string, language: string): string {
  const toneMap = {
    expert: 'professional and authoritative',
    friendly: 'conversational and approachable',
    humorous: 'witty and entertaining'
  };
  
  return `You are an expert content writer. Create a ${format} article in ${language} with a ${toneMap[tone]} tone. 
  Use the provided research data to back up your points with real examples and statistics.`;
}

function buildUserPrompt(keyword: string, research: NewsArticle[]): string {
  const researchSummary = research.map(article => 
    `- ${article.title} (${article.source}, ${article.publishedAt}): ${article.summary}`
  ).join('\n');
  
  return `Topic: ${keyword}\n\nRecent Research:\n${researchSummary}\n\nGenerate a comprehensive article.`;
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithOpenAI(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: buildSystemPrompt(request.format, request.tone, request.language)
      },
      {
        role: 'user',
        content: buildUserPrompt(request.keyword, request.researchData)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0]?.message?.content || '';
}
```

### 4. API Route Example

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeRecentNews } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone } = body;
    
    // Step 1: Research
    const researchData = await scrapeRecentNews(keyword);
    
    if (researchData.length === 0) {
      return NextResponse.json(
        { error: 'No recent data found for this keyword' },
        { status: 404 }
      );
    }
    
    // Step 2: Generate content
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      researchData
    });
    
    return NextResponse.json({
      success: true,
      content,
      sources: researchData.length,
      generatedAt: new Date().toISOString()
    });
    
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 5. Remotion Video Generation

```typescript
// remotion/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ArticleVideoProps {
  title: string;
  keyPoints: string[];
  brandColor: string;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  keyPoints,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: 'white' }}>
      <div style={{ 
        opacity,
        padding: 60,
        display: 'flex',
        flexDirection: 'column',
        gap: 40
      }}>
        <h1 style={{ 
          fontSize: 72, 
          color: brandColor,
          margin: 0 
        }}>
          {title}
        </h1>
        
        <ul style={{ fontSize: 48, lineHeight: 1.6 }}>
          {keyPoints.map((point, i) => {
            const pointFrame = frame - (i * fps);
            const pointOpacity = Math.max(0, Math.min(1, pointFrame / fps));
            
            return (
              <li key={i} style={{ opacity: pointOpacity }}>
                {point}
              </li>
            );
          })}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Video Rendering Endpoint

```typescript
// app/api/render-video/route.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  try {
    const { title, keyPoints, format } = await request.json();
    
    // Bundle Remotion project
    const bundleLocation = await bundle(
      path.join(process.cwd(), 'remotion/index.ts')
    );
    
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: 'ArticleVideo',
      inputProps: { title, keyPoints, brandColor: '#0066FF' }
    });
    
    const outputPath = path.join(
      process.cwd(),
      'public/videos',
      `${Date.now()}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: { title, keyPoints, brandColor: '#0066FF' }
    });
    
    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(outputPath)}`
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

### 7. Complete Workflow Component

```typescript
// components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [language, setLanguage] = useState<'en' | 'vi'>('en');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      // Step 1: Generate content
      const contentRes = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language,
          tone: 'expert'
        })
      });
      
      const contentData = await contentRes.json();
      
      // Step 2: Extract key points for video
      const keyPoints = extractKeyPoints(contentData.content);
      
      // Step 3: Render video
      const videoRes = await fetch('/api/render-video', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          title: keyword,
          keyPoints,
          format: 'vertical' // For Reels/TikTok/Shorts
        })
      });
      
      const videoData = await videoRes.json();
      
      setResult({
        content: contentData.content,
        videoUrl: videoData.videoUrl,
        sources: contentData.sources
      });
      
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4 mb-6">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full px-4 py-2 border rounded"
        />
        
        <select 
          value={format} 
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How To</option>
        </select>
        
        <select 
          value={language} 
          onChange={(e) => setLanguage(e.target.value as any)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="en">English</option>
          <option value="vi">Vietnamese</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white py-3 rounded font-semibold disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content & Video'}
        </button>
      </div>
      
      {result && (
        <div className="space-y-6">
          <div className="bg-gray-50 p-6 rounded">
            <h2 className="text-xl font-bold mb-4">Generated Content</h2>
            <div className="prose max-w-none">{result.content}</div>
            <p className="text-sm text-gray-500 mt-4">
              Based on {result.sources} recent sources
            </p>
          </div>
          
          {result.videoUrl && (
            <div className="bg-gray-50 p-6 rounded">
              <h2 className="text-xl font-bold mb-4">Generated Video</h2>
              <video controls className="w-full rounded">
                <source src={result.videoUrl} type="video/mp4" />
              </video>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - find lines starting with numbers or bullets
  const lines = content.split('\n');
  return lines
    .filter(line => /^(\d+\.|[-*•])/.test(line.trim()))
    .map(line => line.replace(/^(\d+\.|[-*•])\s*/, '').trim())
    .slice(0, 5); // Top 5 points for video
}
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch-processor.ts
export async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const research = await scrapeRecentNews(keyword);
    const content = await generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research
    });
    
    results.push({ keyword, content });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Multi-Platform Video Export

```typescript
// lib/video/multi-platform.ts
const platformConfigs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
  youtube: { width: 1920, height: 1080, fps: 60 }
};

export async function renderForAllPlatforms(composition: any) {
  const outputs = [];
  
  for (const [platform, config] of Object.entries(platformConfigs)) {
    const outputPath = `public/videos/${platform}-${Date.now()}.mp4`;
    
    await renderMedia({
      ...composition,
      width: config.width,
      height: config.height,
      fps: config.fps,
      outputLocation: outputPath
    });
    
    outputs.push({ platform, path: outputPath });
  }
  
  return outputs;
}
```

## Troubleshooting

### API Rate Limits
- Implement exponential backoff for research APIs
- Cache research results for 1 hour to avoid duplicate calls
- Use queue systems (Bull, BullMQ) for batch processing

### Video Rendering Performance
- Use Remotion's Lambda rendering for faster cloud-based processing
- Pre-bundle Remotion project to reduce startup time
- Consider using lower quality settings for previews

### AI Token Limits
- Split long research data into chunks
- Use streaming responses for real-time content generation
- Implement token counting before API calls

### Memory Issues
- Process videos one at a time, not in parallel
- Clean up temporary files after rendering
- Use serverless functions for heavy operations

## Advanced Usage

### Custom AI Prompts

```typescript
// lib/ai/prompt-templates.ts
export const promptTemplates = {
  viral: `Create highly shareable content optimized for social engagement...`,
  seo: `Write SEO-optimized content with target keywords naturally integrated...`,
  thought_leadership: `Position the author as an industry expert with unique insights...`
};

export function getPromptTemplate(type: keyof typeof promptTemplates): string {
  return promptTemplates[type];
}
```

### Scheduling & Automation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await getTrendingTopics();
  await batchGenerateContent(trendingTopics);
});
```

This skill enables AI agents to help developers build and customize automated content pipelines with real-time research, AI generation, and video rendering capabilities.
