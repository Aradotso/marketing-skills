---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I create automated content with marketing pipeline
  - set up AI content automation from research to video
  - use marketing pipeline share for content generation
  - automate content research and scriptwriting with AI
  - generate videos automatically from content scripts
  - configure Claude and OpenAI for content pipeline
  - build automated marketing content workflow
  - create multi-language content with AI pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end content automation system that transforms keywords into complete content pieces including research, scripts, and videos. It leverages Claude 3, OpenAI, and Remotion to create a fully automated content production pipeline that can:

- Auto-crawl news from TechCrunch, a16z, Twitter/X, LinkedIn
- Generate multi-format content (toplist, POV, case studies, how-to)
- Support bilingual output (English/Vietnamese)
- Render videos and infographics automatically
- Optimize content for Reels, TikTok, and Shorts

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
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news/research
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion rendering (optional)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawlers/    # News/content crawlers
│   │   ├── generators/  # Content generators
│   │   └── remotion/    # Video rendering
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Usage Patterns

### 1. Content Research & Crawling

```typescript
import { crawlNews } from '@/lib/crawlers/news-crawler';
import { analyzeContent } from '@/lib/ai/content-analyzer';

async function researchTopic(keyword: string) {
  // Crawl news from multiple sources
  const newsData = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Analyze with AI
  const insights = await analyzeContent(newsData, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    extractInsights: true,
    generateSummary: true
  });

  return insights;
}

// Usage
const research = await researchTopic('AI automation');
console.log(research.insights);
```

### 2. Content Generation with Multiple Formats

```typescript
import { generateContent } from '@/lib/generators/content-generator';

async function createContent(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    aiProvider: 'claude',
    includeData: true,
    researchFirst: true
  });

  return content;
}

// Generate multiple formats
const topList = await createContent('AI Tools 2024', 'toplist');
const caseStudy = await createContent('Marketing Automation Success', 'case-study');
```

### 3. Bilingual Content Generation

```typescript
import { generateBilingualContent } from '@/lib/generators/bilingual-generator';

async function createBilingualPost(keyword: string) {
  const content = await generateBilingualContent({
    keyword,
    primaryLanguage: 'en',
    secondaryLanguage: 'vi',
    format: 'pov',
    tone: 'expert',
    includeStats: true
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.meta
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/remotion/video-renderer';
import { createVideoScript } from '@/lib/generators/video-script-generator';

async function generateVideoContent(content: any) {
  // Create video script from content
  const script = await createVideoScript({
    content,
    duration: 60, // seconds
    platform: 'reels', // 'reels' | 'tiktok' | 'shorts'
    style: 'infographic'
  });

  // Render video with Remotion
  const video = await renderVideo({
    compositionId: 'ContentVideo',
    script,
    props: {
      title: content.title,
      keyPoints: content.keyPoints,
      branding: {
        logo: '/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    },
    outputFormat: 'mp4',
    resolution: '1080x1920' // Vertical for social media
  });

  return video.url;
}
```

### 5. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    enableVideo: true,
    autoPublish: false
  });

  // Run complete pipeline
  const result = await pipeline.execute({
    keyword,
    steps: [
      'research',      // Crawl and analyze news
      'script',        // Generate content script
      'content',       // Create full article
      'translate',     // Translate to secondary language
      'video',         // Generate video
      'optimize'       // SEO and platform optimization
    ],
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    videoEnabled: true
  });

  return {
    research: result.research,
    content: result.content,
    video: result.video,
    metadata: result.metadata
  };
}

// Execute pipeline
const output = await runFullPipeline('AI marketing automation');
```

## API Routes

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { researchTopic } from '@/lib/crawlers/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, sources, timeframe } = req.body;

  try {
    const research = await researchTopic(keyword, sources, timeframe);
    res.status(200).json(research);
  } catch (error) {
    res.status(500).json({ error: 'Research failed' });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/generators/content-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { topic, format, languages, tone } = req.body;

  try {
    const content = await generateContent({
      topic,
      format,
      languages,
      tone,
      aiProvider: process.env.AI_PROVIDER || 'claude'
    });

    res.status(200).json(content);
  } catch (error) {
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

### Video Render Endpoint

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderVideo } from '@/lib/remotion/video-renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { script, platform, style } = req.body;

  try {
    const video = await renderVideo({
      compositionId: 'ContentVideo',
      script,
      props: {
        platform,
        style
      }
    });

    res.status(200).json({ videoUrl: video.url });
  } catch (error) {
    res.status(500).json({ error: 'Video rendering failed' });
  }
}
```

## Frontend Components

### Content Generator Component

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          languages: ['en', 'vi'],
          enableVideo: true
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
    <div className="content-generator">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="keyword-input"
      />
      
      <select value={format} onChange={(e) => setFormat(e.target.value)}>
        <option value="toplist">Top List</option>
        <option value="pov">POV</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-To</option>
      </select>

      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="result">
          <h2>{result.content.title}</h2>
          <div dangerouslySetInnerHTML={{ __html: result.content.body }} />
          {result.video && (
            <video src={result.video.url} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run content pipeline (if CLI is implemented)
npm run pipeline -- --keyword "AI automation" --format toplist --video

# Render Remotion video
npm run remotion:render -- --composition ContentVideo --output video.mp4
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import cron from 'node-cron';
import { runFullPipeline } from '@/lib/pipeline/content-pipeline';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI news', 'marketing trends', 'tech updates'];
  
  for (const keyword of keywords) {
    await runFullPipeline(keyword);
  }
});
```

### Pattern 2: Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      generateContent({
        topic: keyword,
        format: 'toplist',
        languages: ['en', 'vi']
      })
    )
  );

  return results;
}

// Usage
const batch = await generateBatchContent([
  'AI tools 2024',
  'Marketing automation',
  'Content strategy'
]);
```

### Pattern 3: Custom AI Provider

```typescript
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

class AIProvider {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }

  async generate(prompt: string, provider: 'claude' | 'openai') {
    if (provider === 'claude') {
      const response = await this.claude.messages.create({
        model: 'claude-3-opus-20240229',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }]
      });
      return response.content[0].text;
    } else {
      const response = await this.openai.chat.completions.create({
        model: 'gpt-4-turbo-preview',
        messages: [{ role: 'user', content: prompt }]
      });
      return response.choices[0].message.content;
    }
  }
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

// Apply to API routes
export const config = {
  api: {
    bodyParser: {
      sizeLimit: '10mb'
    }
  }
};
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for video rendering
export const config = {
  api: {
    externalResolver: true,
    responseLimit: false,
    bodyParser: {
      sizeLimit: '50mb'
    }
  },
  maxDuration: 300 // 5 minutes
};
```

### Issue: Memory Issues with Large Content

```typescript
// Stream large responses
import { Readable } from 'stream';

async function streamContent(content: any, res: any) {
  const stream = new Readable();
  stream.push(JSON.stringify(content));
  stream.push(null);
  
  stream.pipe(res);
}
```

### Issue: AI Provider Errors

```typescript
// Implement fallback providers
async function generateWithFallback(prompt: string) {
  try {
    return await generateWithClaude(prompt);
  } catch (error) {
    console.warn('Claude failed, falling back to OpenAI');
    return await generateWithOpenAI(prompt);
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement caching** for research data to reduce API calls
3. **Use queue systems** (Bull, BeeQueue) for video rendering
4. **Store generated content** in a database for reuse
5. **Implement proper error handling** with retry logic
6. **Monitor API usage** to stay within limits
7. **Optimize video rendering** by using cloud rendering services
8. **Version control your prompts** for consistent AI output
