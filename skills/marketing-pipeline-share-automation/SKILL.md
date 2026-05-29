---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from text using Remotion
  - crawl news sources for content research
  - create multilingual content with Claude API
  - automate social media content generation
  - build AI content workflow with OpenAI
  - generate marketing videos automatically
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers build and use an automated content creation pipeline that handles research (web crawling), script generation (using Claude/OpenAI), and video rendering (Remotion). The system transforms keywords into complete content pieces with multilingual support.

## What This Project Does

Marketing Pipeline Share is a complete AI-powered content automation system that:

- **Auto-crawls** news sources (TechCrunch, Twitter, LinkedIn) for fresh data
- **Generates content** in multiple formats (lists, POV, case studies, how-tos) using Claude 3 or OpenAI
- **Creates bilingual content** (English + Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms
- **Provides a Next.js interface** for managing the entire workflow

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database/Storage
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Development Server

```bash
npm run dev
# or
yarn dev

# Access at http://localhost:3000
```

## Core Architecture

```typescript
// types/content.ts
interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  tone: 'expert' | 'friendly' | 'humorous';
  includeVideo?: boolean;
}

interface ResearchData {
  source: string;
  title: string;
  url: string;
  publishedAt: Date;
  summary: string;
  insights: string[];
}

interface GeneratedContent {
  title: string;
  content: string;
  language: string;
  metadata: {
    format: string;
    wordCount: number;
    sources: ResearchData[];
  };
  videoUrl?: string;
}
```

## Research & Crawling Module

```typescript
// lib/research/crawler.ts
import axios from 'axios';

export async function crawlNewsSources(keyword: string): Promise<ResearchData[]> {
  const sources = [
    'techcrunch',
    'twitter',
    'linkedin'
  ];
  
  const results: ResearchData[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/news/${source}`,
        {
          params: { q: keyword, limit: 10 },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
            'X-RapidAPI-Host': 'news-api.p.rapidapi.com'
          }
        }
      );
      
      results.push(...response.data.articles.map((article: any) => ({
        source,
        title: article.title,
        url: article.url,
        publishedAt: new Date(article.publishedAt),
        summary: article.description,
        insights: extractInsights(article.content)
      })));
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
    }
  }
  
  return results;
}

function extractInsights(content: string): string[] {
  // Extract key data points, statistics, quotes
  const insights: string[] = [];
  
  // Regex for statistics (numbers with units/percentages)
  const statsRegex = /\b\d+(?:\.\d+)?%?\s+(?:users|customers|growth|increase|decrease)\b/gi;
  const stats = content.match(statsRegex);
  if (stats) insights.push(...stats);
  
  return insights;
}
```

## Content Generation with Claude/OpenAI

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!
});

export async function generateContent(
  request: ContentRequest,
  researchData: ResearchData[]
): Promise<GeneratedContent> {
  
  const prompt = buildPrompt(request, researchData);
  
  // Use Claude for complex, nuanced content
  if (request.format === 'case-study' || request.format === 'pov') {
    return await generateWithClaude(prompt, request);
  }
  
  // Use OpenAI for structured content
  return await generateWithOpenAI(prompt, request);
}

async function generateWithClaude(
  prompt: string,
  request: ContentRequest
): Promise<GeneratedContent> {
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
  
  return parseGeneratedContent(content, request);
}

async function generateWithOpenAI(
  prompt: string,
  request: ContentRequest
): Promise<GeneratedContent> {
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and technology.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  const content = completion.choices[0].message.content || '';
  
  return parseGeneratedContent(content, request);
}

function buildPrompt(request: ContentRequest, research: ResearchData[]): string {
  const toneMap = {
    expert: 'professional and authoritative',
    friendly: 'conversational and approachable',
    humorous: 'entertaining and witty'
  };
  
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write a thought-leadership piece expressing a unique perspective',
    'case-study': 'Develop a detailed case study with problem, solution, and results',
    'how-to': 'Create a step-by-step tutorial with actionable instructions'
  };
  
  return `
Create a ${request.format} article about "${request.keyword}" in ${request.language === 'both' ? 'English and Vietnamese' : request.language}.

Tone: ${toneMap[request.tone]}

${formatInstructions[request.format]}

Use the following research data:
${research.map(r => `- ${r.title} (${r.source}): ${r.summary}\n  Insights: ${r.insights.join(', ')}`).join('\n')}

Requirements:
- Include specific data points and statistics from the research
- Make it engaging and shareable on social media
- Add clear headings and structure
- Include actionable takeaways
${request.language === 'both' ? '- Provide both English and Vietnamese versions, separated by "---LANGUAGE-BREAK---"' : ''}
`;
}

function parseGeneratedContent(
  content: string,
  request: ContentRequest
): GeneratedContent {
  
  if (request.language === 'both') {
    const [english, vietnamese] = content.split('---LANGUAGE-BREAK---');
    return {
      title: extractTitle(english),
      content: english + '\n\n' + vietnamese,
      language: 'both',
      metadata: {
        format: request.format,
        wordCount: content.split(' ').length,
        sources: []
      }
    };
  }
  
  return {
    title: extractTitle(content),
    content,
    language: request.language,
    metadata: {
      format: request.format,
      wordCount: content.split(' ').length,
      sources: []
    }
  };
}

function extractTitle(content: string): string {
  const match = content.match(/^#\s+(.+)$/m);
  return match ? match[1] : 'Untitled';
}
```

## Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function generateVideo(
  content: GeneratedContent,
  format: 'reels' | 'tiktok' | 'shorts' = 'reels'
): Promise<string> {
  
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      content: content.content,
      format: format
    }
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${format}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: content.title,
      content: content.content,
      format: format
    }
  });
  
  return outputLocation;
}
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content, format }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const aspectRatios = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  // Extract key points from content
  const points = content
    .split('\n')
    .filter(line => line.match(/^-|\d+\./))
    .slice(0, 5);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <div style={{
          display: 'flex',
          flexDirection: 'column',
          justifyContent: 'center',
          alignItems: 'center',
          padding: 60,
          opacity
        }}>
          <h1 style={{
            color: 'white',
            fontSize: 72,
            textAlign: 'center',
            fontWeight: 'bold',
            lineHeight: 1.2
          }}>
            {title}
          </h1>
        </div>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <div style={{
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            padding: 60
          }}>
            <p style={{
              color: 'white',
              fontSize: 48,
              textAlign: 'center',
              lineHeight: 1.5
            }}>
              {point}
            </p>
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## API Routes (Next.js)

```typescript
// pages/api/content/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { crawlNewsSources } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const request: ContentRequest = req.body;
    
    // Step 1: Research
    const researchData = await crawlNewsSources(request.keyword);
    
    // Step 2: Generate content
    const content = await generateContent(request, researchData);
    
    // Step 3: Generate video (optional)
    if (request.includeVideo) {
      const videoUrl = await generateVideo(content);
      content.videoUrl = videoUrl;
    }
    
    res.status(200).json(content);
  } catch (error) {
    console.error('Content generation failed:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

## Frontend Usage Example

```typescript
// components/ContentGenerator.tsx
import { useState } from 'react';

export function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<GeneratedContent | null>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/content/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: 'AI automation',
          format: 'toplist',
          language: 'both',
          tone: 'expert',
          includeVideo: true
        })
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
    <div>
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h2>{result.title}</h2>
          <div dangerouslySetInnerHTML={{ __html: result.content }} />
          {result.videoUrl && (
            <video src={result.videoUrl} controls />
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
// scripts/batch-generate.ts
import { crawlNewsSources } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';

const keywords = ['AI marketing', 'content automation', 'video generation'];
const formats: ContentRequest['format'][] = ['toplist', 'how-to'];

async function batchGenerate() {
  for (const keyword of keywords) {
    const research = await crawlNewsSources(keyword);
    
    for (const format of formats) {
      const content = await generateContent({
        keyword,
        format,
        language: 'both',
        tone: 'expert',
        includeVideo: false
      }, research);
      
      // Save to database or file
      console.log(`Generated: ${content.title}`);
    }
  }
}

batchGenerate();
```

### Custom Video Templates

```typescript
// remotion/templates/InfographicVideo.tsx
export const InfographicVideo: React.FC<{ data: any[] }> = ({ data }) => {
  return (
    <AbsoluteFill>
      {data.map((item, i) => (
        <Sequence from={i * 90} durationInFrames={90} key={i}>
          <div className="stat-card">
            <h2>{item.label}</h2>
            <p className="stat-value">{item.value}</p>
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
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
      
      this.process();
    });
  }
  
  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift()!;
    await fn();
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    this.processing = false;
    this.process();
  }
}

export const aiLimiter = new RateLimiter();
```

### Video Rendering Memory Issues

```typescript
// Increase Node.js memory limit in package.json
{
  "scripts": {
    "render": "NODE_OPTIONS='--max-old-space-size=4096' node scripts/render-videos.js"
  }
}
```

### Claude/OpenAI Timeout Handling

```typescript
async function generateWithRetry(
  generateFn: () => Promise<GeneratedContent>,
  maxRetries = 3
): Promise<GeneratedContent> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateFn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      
      if (error.status === 429) {
        await new Promise(resolve => setTimeout(resolve, 5000 * (i + 1)));
      }
    }
  }
  
  throw new Error('Max retries exceeded');
}
```
