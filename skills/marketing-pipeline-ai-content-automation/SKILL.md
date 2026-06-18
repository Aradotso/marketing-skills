---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - "automate content creation with AI research"
  - "generate blog posts from trending news"
  - "create video content from articles automatically"
  - "set up AI content pipeline with Claude"
  - "crawl news and generate content"
  - "build automated marketing content system"
  - "render videos from blog content"
  - "scrape tech news and write articles with AI"
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research (web scraping), content generation (using Claude/OpenAI), and video rendering (Remotion). It automatically crawls news sources like TechCrunch, Twitter, and LinkedIn, generates articles in multiple formats and languages, and renders them into videos for social media.

## What It Does

The Ultimate AI Content Pipeline automates:
- **Research**: Crawls and analyzes fresh data from major news sources within 24h
- **Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) in English and Vietnamese
- **Video Generation**: Automatically renders infographics and short videos from written content using Remotion
- **Multi-platform Output**: Exports videos optimized for Reels, TikTok, and Shorts

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

## Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if using)
DATABASE_URL=your_database_url

# Optional: Social Media Auto-posting
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run render
```

## Key Architecture Components

### 1. Research/Crawling Module

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  selector: string;
  type: 'techcrunch' | 'twitter' | 'linkedin';
}

export async function crawlNews(keyword: string, sources: NewsSource[]) {
  const articles = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(source.url, {
        params: { q: keyword },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        }
      });
      
      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to crawl ${source.type}:`, error);
    }
  }
  
  return articles;
}

export function extractInsights(articles: any[]) {
  // Analyze articles for key insights, trends, and data points
  const insights = {
    trendingTopics: [],
    keyStatistics: [],
    expertQuotes: [],
    recentDevelopments: []
  };
  
  // Processing logic here
  return insights;
}
```

### 2. Content Generation with Claude/OpenAI

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  length: 'short' | 'medium' | 'long';
}

export async function generateContent(
  insights: any,
  config: ContentConfig
) {
  const prompt = buildPrompt(insights, config);
  
  // Using Claude for content generation
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].text;
}

function buildPrompt(insights: any, config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and actionable insights',
    'how-to': 'Provide step-by-step instructions with clear actions'
  };
  
  return `
    You are a ${config.tone} content writer creating a ${config.format} article in ${config.language}.
    
    ${formatInstructions[config.format]}
    
    Research insights:
    ${JSON.stringify(insights, null, 2)}
    
    Requirements:
    - Include specific data points and statistics
    - Reference recent developments (within 24h)
    - Write in ${config.language === 'vi' ? 'Vietnamese' : 'English'}
    - Tone: ${config.tone}
    - Length: ${config.length}
    
    Generate the complete article now.
  `;
}
```

### 3. Dual AI Strategy (Claude + OpenAI)

```typescript
// lib/ai/dual-generation.ts
export async function generateWithFallback(
  insights: any,
  config: ContentConfig
) {
  try {
    // Try Claude first (better for creative content)
    return await generateContent(insights, config);
  } catch (error) {
    console.warn('Claude failed, falling back to OpenAI:', error);
    
    // Fallback to OpenAI
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: buildPrompt(insights, config)
      }],
      temperature: 0.7,
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content;
  }
}
```

### 4. Video Rendering with Remotion

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface VideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ContentVideo: React.FC<VideoProps> = ({ 
  title, 
  points, 
  duration 
}) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center' 
        }}>
          <h1 style={{ 
            color: 'white', 
            fontSize: 60,
            opacity: frame / 30 
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence 
          key={index} 
          from={90 + (index * 120)} 
          durationInFrames={120}
        >
          <AbsoluteFill style={{ 
            padding: 60,
            justifyContent: 'center' 
          }}>
            <div style={{ 
              color: 'white', 
              fontSize: 40,
              lineHeight: 1.5 
            }}>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  article: any,
  outputPath: string
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: article.title,
      points: article.keyPoints,
      duration: 30
    },
  });
  
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

## Complete Pipeline Workflow

```typescript
// lib/pipeline/orchestrator.ts
import { crawlNews, extractInsights } from '../research/crawler';
import { generateWithFallback } from '../ai/dual-generation';
import { renderContentVideo } from '../video/renderer';

export async function runContentPipeline(keyword: string) {
  console.log(`Starting pipeline for keyword: ${keyword}`);
  
  // Step 1: Research
  const sources = [
    { url: 'https://api.techcrunch.com/search', type: 'techcrunch' },
    { url: 'https://api.twitter.com/v2/tweets/search', type: 'twitter' },
  ];
  
  const articles = await crawlNews(keyword, sources);
  const insights = extractInsights(articles);
  
  // Step 2: Generate Content (Multiple Formats)
  const formats: ContentConfig[] = [
    { format: 'toplist', tone: 'expert', language: 'en', length: 'medium' },
    { format: 'pov', tone: 'friendly', language: 'vi', length: 'medium' },
  ];
  
  const generatedContent = [];
  
  for (const config of formats) {
    const content = await generateWithFallback(insights, config);
    generatedContent.push({
      config,
      content,
      metadata: {
        keyword,
        generatedAt: new Date(),
      }
    });
  }
  
  // Step 3: Render Videos
  const videos = [];
  
  for (const item of generatedContent) {
    const videoPath = `./output/${keyword}-${item.config.format}-${Date.now()}.mp4`;
    await renderContentVideo(
      {
        title: extractTitle(item.content),
        keyPoints: extractKeyPoints(item.content),
      },
      videoPath
    );
    videos.push(videoPath);
  }
  
  return {
    articles: generatedContent,
    videos,
    insights,
  };
}

function extractTitle(content: string): string {
  const lines = content.split('\n');
  return lines[0].replace(/^#\s*/, '');
}

function extractKeyPoints(content: string): string[] {
  const lines = content.split('\n');
  return lines
    .filter(line => line.match(/^[\d\-\*]/))
    .slice(0, 5);
}
```

## API Route Example (Next.js)

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '../../lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword } = req.body;
  
  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }
  
  try {
    const result = await runContentPipeline(keyword);
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline failed', 
      details: error.message 
    });
  }
}
```

## Frontend Usage Example

```typescript
// components/ContentGenerator.tsx
import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword }),
      });
      
      const data = await response.json();
      setResults(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="container">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword (e.g., AI, blockchain)"
      />
      
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {results && (
        <div className="results">
          <h2>Generated Articles: {results.articles.length}</h2>
          <h2>Videos: {results.videos.length}</h2>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Scheduling Automated Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/orchestrator';

// Run every day at 8 AM
export function scheduleDaily() {
  cron.schedule('0 8 * * *', async () => {
    const keywords = ['AI', 'blockchain', 'marketing automation'];
    
    for (const keyword of keywords) {
      try {
        await runContentPipeline(keyword);
        console.log(`✅ Generated content for: ${keyword}`);
      } catch (error) {
        console.error(`❌ Failed for ${keyword}:`, error);
      }
    }
  });
}
```

### Customizing Video Templates

```typescript
// remotion/templates/MinimalTemplate.tsx
export const MinimalTemplate: React.FC<VideoProps> = ({ title, points }) => {
  return (
    <AbsoluteFill style={{ 
      backgroundColor: 'white',
      fontFamily: 'Inter, sans-serif' 
    }}>
      {/* Custom template implementation */}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits
```typescript
// lib/utils/rate-limiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  delayMs: number = 1000
): Promise<T> {
  await new Promise(resolve => setTimeout(resolve, delayMs));
  return fn();
}

// Usage
const content = await withRateLimit(
  () => generateContent(insights, config),
  2000 // 2 second delay
);
```

### Video Rendering Memory Issues
```bash
# Increase Node memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run render
```

### Handling AI API Errors
```typescript
const MAX_RETRIES = 3;

async function generateWithRetry(insights: any, config: ContentConfig) {
  for (let i = 0; i < MAX_RETRIES; i++) {
    try {
      return await generateContent(insights, config);
    } catch (error) {
      if (i === MAX_RETRIES - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

### Debugging Crawled Data
```typescript
// Add logging to understand what's being scraped
console.log('Crawled articles:', JSON.stringify(articles, null, 2));
console.log('Extracted insights:', insights);
```

## Performance Tips

- Use caching for frequently crawled sources
- Run video rendering in background jobs/queues
- Batch API requests when possible
- Store generated content in a database for reuse
- Use CDN for rendered video files
