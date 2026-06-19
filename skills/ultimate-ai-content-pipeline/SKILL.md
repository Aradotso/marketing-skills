---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from articles automatically
  - create content from research to video
  - automate social media content with AI
  - build AI content generation workflow
  - use remotion for marketing video automation
  - scrape news and generate content automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - a complete automated content creation system that handles research, scriptwriting, article generation, and video rendering. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and automatically renders videos using Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-scans research sources** (TechCrunch, a16z, Twitter, LinkedIn) for recent news
- **Generates content** in multiple formats (toplist, POV, case study, how-to)
- **Supports bilingual output** (English & Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

Built with TypeScript, Next.js, and integrated with Claude 3, OpenAI, and RapidAPI.

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

Create a `.env.local` file in the root directory:

```env
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key

# RapidAPI for news scraping
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration (if using separate rendering service)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Database (if persistence is needed)
DATABASE_URL=your_database_connection_string
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # News scraping & research
│   │   └── video/       # Remotion video generation
│   ├── api/             # API routes
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & News Scraping

```typescript
import { scrapeRecentNews } from '@/lib/research/scraper';

async function gatherResearch(keyword: string) {
  const newsData = await scrapeRecentNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    apiKey: process.env.RAPIDAPI_KEY!
  });

  return newsData;
}

// Example response structure
interface NewsData {
  articles: Array<{
    title: string;
    url: string;
    publishedAt: string;
    summary: string;
    source: string;
  }>;
  insights: string[];
  trends: string[];
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(research: NewsData, format: string) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY!,
    prompt: {
      research,
      format, // 'toplist', 'pov', 'case-study', 'how-to'
      language: 'both', // 'en', 'vi', or 'both'
      tone: 'professional' // 'expert', 'friendly', 'humorous'
    }
  });

  return content;
}

// Using OpenAI alternative
async function createWithOpenAI(research: NewsData) {
  const content = await generateContent({
    provider: 'openai',
    model: 'gpt-4-turbo-preview',
    apiKey: process.env.OPENAI_API_KEY!,
    prompt: {
      research,
      format: 'toplist',
      language: 'en',
      tone: 'friendly'
    }
  });

  return content;
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function generateVideoFromContent(content: GeneratedContent) {
  // Define video composition
  const composition = {
    id: 'ContentVideo',
    width: 1080,
    height: 1920, // Vertical format for Reels/TikTok
    fps: 30,
    durationInFrames: 300, // 10 seconds
    props: {
      title: content.title,
      keyPoints: content.keyPoints,
      bgColor: '#0066cc',
      fontFamily: 'Inter'
    }
  };

  const videoPath = await renderVideo({
    composition,
    outputLocation: `public/videos/${content.id}.mp4`,
    quality: 'high'
  });

  return videoPath;
}

// Multi-platform rendering
async function renderForAllPlatforms(content: GeneratedContent) {
  const formats = [
    { platform: 'reels', width: 1080, height: 1920 },
    { platform: 'youtube', width: 1920, height: 1080 },
    { platform: 'tiktok', width: 1080, height: 1920 }
  ];

  const videos = await Promise.all(
    formats.map(format => 
      renderVideo({
        composition: {
          ...composition,
          width: format.width,
          height: format.height
        },
        outputLocation: `public/videos/${content.id}-${format.platform}.mp4`
      })
    )
  );

  return videos;
}
```

## Complete Pipeline Example

```typescript
import { Pipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new Pipeline({
    aiProvider: 'claude',
    renderVideos: true,
    autoPublish: false
  });

  try {
    // Step 1: Research
    const research = await pipeline.research(keyword, {
      sources: ['techcrunch', 'a16z', 'twitter'],
      depth: 'deep'
    });

    // Step 2: Generate content
    const article = await pipeline.generateContent({
      research,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'expert'
    });

    // Step 3: Create visuals
    const video = await pipeline.renderVideo({
      content: article,
      template: 'modern-infographic',
      platforms: ['reels', 'tiktok', 'youtube-shorts']
    });

    // Step 4: Review and export
    return {
      article,
      video,
      metadata: {
        keyword,
        createdAt: new Date(),
        wordCount: article.content.split(' ').length
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const result = await runContentPipeline('AI automation trends 2024');
console.log('Content generated:', result);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeRecentNews } from '@/lib/research/scraper';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();

  try {
    const research = await scrapeRecentNews({
      keyword,
      sources,
      timeRange,
      apiKey: process.env.RAPIDAPI_KEY!
    });

    return NextResponse.json({ success: true, data: research });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { research, format, language, tone, provider } = await request.json();

  const apiKey = provider === 'claude' 
    ? process.env.ANTHROPIC_API_KEY 
    : process.env.OPENAI_API_KEY;

  try {
    const content = await generateContent({
      provider,
      apiKey: apiKey!,
      prompt: { research, format, language, tone }
    });

    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/remotion-renderer';

export async function POST(request: NextRequest) {
  const { content, template, platforms } = await request.json();

  try {
    const videos = await Promise.all(
      platforms.map(platform => 
        renderVideo({
          content,
          template,
          platform,
          outputDir: 'public/videos'
        })
      )
    );

    return NextResponse.json({ success: true, videos });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      // Step 1: Research
      const researchRes = await fetch('/api/research', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          sources: ['techcrunch', 'a16z'],
          timeRange: '24h'
        })
      });
      const { data: research } = await researchRes.json();

      // Step 2: Generate content
      const contentRes = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          research,
          format,
          language: 'both',
          tone: 'professional',
          provider: 'claude'
        })
      });
      const { content } = await contentRes.json();

      // Step 3: Render video
      const videoRes = await fetch('/api/render', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content,
          template: 'modern-infographic',
          platforms: ['reels', 'tiktok']
        })
      });
      const { videos } = await videoRes.json();

      setResult({ content, videos });
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
      />
      
      <select value={format} onChange={(e) => setFormat(e.target.value)}>
        <option value="toplist">Top List</option>
        <option value="pov">POV</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How To</option>
      </select>

      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="result">
          <h3>{result.content.title}</h3>
          <div dangerouslySetInnerHTML={{ __html: result.content.html }} />
          
          {result.videos.map((video, idx) => (
            <video key={idx} src={video.url} controls />
          ))}
        </div>
      )}
    </div>
  );
}
```

## Configuration Options

### AI Provider Configuration

```typescript
// config/ai-providers.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4000,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7
  }
};
```

### Research Sources Configuration

```typescript
// config/research-sources.ts
export const researchConfig = {
  sources: {
    techcrunch: {
      apiEndpoint: 'https://techcrunch.com/wp-json/wp/v2',
      enabled: true
    },
    a16z: {
      apiEndpoint: 'https://a16z.com/feed',
      enabled: true
    },
    twitter: {
      apiEndpoint: 'https://api.twitter.com/2',
      requiresAuth: true,
      enabled: true
    }
  },
  defaultTimeRange: '24h',
  maxArticles: 50
};
```

### Video Rendering Configuration

```typescript
// config/video.ts
export const videoConfig = {
  platforms: {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    youtube: { width: 1920, height: 1080, fps: 60 },
    shorts: { width: 1080, height: 1920, fps: 30 }
  },
  defaultDuration: 10, // seconds
  quality: 'high',
  codec: 'h264'
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    const content = await runContentPipeline(keyword);
    results.push(content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Scheduled Content Creation

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline(topic);
  }
});
```

### Content Versioning

```typescript
async function generateMultipleVersions(keyword: string) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'];
  const versions = [];

  for (const format of formats) {
    const content = await generateContent({
      provider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY!,
      prompt: { format, keyword }
    });
    
    versions.push({ format, content });
  }

  return versions;
}
```

## Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateConfig() {
  const requiredKeys = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = requiredKeys.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}
```

### Rate Limiting

```typescript
import { RateLimiter } from 'limiter';

const limiter = new RateLimiter({
  tokensPerInterval: 10,
  interval: 'minute'
});

async function callAIWithRateLimit(prompt: string) {
  await limiter.removeTokens(1);
  return await generateContent(prompt);
}
```

### Error Handling

```typescript
async function safeGenerateContent(params: any) {
  const maxRetries = 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      return await generateContent(params);
    } catch (error) {
      attempt++;
      
      if (attempt === maxRetries) {
        throw new Error(`Failed after ${maxRetries} attempts: ${error.message}`);
      }
      
      // Exponential backoff
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, attempt) * 1000)
      );
    }
  }
}
```

### Video Rendering Failures

```typescript
async function renderWithFallback(content: any) {
  try {
    return await renderVideo(content);
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Fallback to static image
    return await generateStaticImage(content);
  }
}
```

This skill provides comprehensive guidance for using the Ultimate AI Content Pipeline to automate content creation from research through video generation.
