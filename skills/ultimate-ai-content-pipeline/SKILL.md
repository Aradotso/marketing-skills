---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation with AI-powered writing and Remotion rendering
triggers:
  - how do I generate content with the AI content pipeline
  - set up automated content creation with research and video
  - create AI-powered articles with automatic video generation
  - use Remotion to render content videos automatically
  - configure Claude and OpenAI for content automation
  - crawl news sources and generate multilingual content
  - automate content pipeline from research to publication
  - generate videos from articles using the content pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that handles research (web scraping), AI-powered article generation (Claude/OpenAI), and automatic video rendering (Remotion). Create multilingual content in multiple formats from a single keyword.

## What It Does

- **Auto Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for recent data
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual Output**: Generates English and Vietnamese versions simultaneously
- **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
- **Multi-platform Optimization**: Exports video formats for Reels, TikTok, Shorts

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
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research API
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Config
NEXT_PUBLIC_API_URL=http://localhost:3000

# Video Rendering
REMOTION_RENDER_PATH=/tmp/remotion-renders
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Video rendering (Remotion)
npm run render
```

## Key API Routes & Usage

### 1. Research Endpoint

Crawls news sources for recent content on a topic:

```typescript
// app/api/research/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const { keyword, sources, timeRange } = await request.json();
  
  const headers = {
    'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
    'X-RapidAPI-Host': 'news-api.rapidapi.com'
  };
  
  const response = await fetch(
    `https://news-api.rapidapi.com/search?q=${encodeURIComponent(keyword)}&time=${timeRange}`,
    { headers }
  );
  
  const data = await response.json();
  
  return NextResponse.json({
    articles: data.articles,
    insights: extractInsights(data.articles)
  });
}
```

### 2. Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function generateContent(
  prompt: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi' = 'en'
) {
  const systemPrompt = getFormatPrompt(format, language);
  
  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function getFormatPrompt(
  format: string, 
  language: string
): string {
  const prompts = {
    toplist: {
      en: 'Create a comprehensive top 10 list article with detailed explanations for each item. Include data and examples.',
      vi: 'Tạo bài viết top 10 chi tiết với giải thích rõ ràng cho từng mục. Bao gồm số liệu và ví dụ cụ thể.'
    },
    pov: {
      en: 'Write an opinion piece with a unique perspective. Use first-person narrative and strong arguments backed by data.',
      vi: 'Viết bài quan điểm với góc nhìn độc đáo. Sử dụng ngôi thứ nhất và lập luận mạnh mẽ có số liệu chứng minh.'
    },
    'case-study': {
      en: 'Create a detailed case study with problem, solution, and results. Include specific metrics and learnings.',
      vi: 'Tạo case study chi tiết với vấn đề, giải pháp và kết quả. Bao gồm các chỉ số cụ thể và bài học.'
    },
    'how-to': {
      en: 'Write a step-by-step how-to guide. Make it actionable with clear instructions and examples.',
      vi: 'Viết hướng dẫn từng bước cụ thể. Làm cho nó dễ thực hiện với chỉ dẫn rõ ràng và ví dụ.'
    }
  };
  
  return prompts[format][language];
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentOpenAI(
  prompt: string,
  format: string,
  language: string = 'en'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: getFormatPrompt(format, language)
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content;
}
```

## Video Generation with Remotion

### Video Composition Setup

```typescript
// remotion/compositions/ContentVideo.tsx
import { Composition } from 'remotion';
import { VideoTemplate } from './templates/VideoTemplate';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={VideoTemplate}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Article Title',
          points: [],
          backgroundImage: null
        }}
      />
    </>
  );
};
```

### Video Template Component

```typescript
// remotion/templates/VideoTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface VideoTemplateProps {
  title: string;
  points: string[];
  backgroundImage?: string;
}

export const VideoTemplate: React.FC<VideoTemplateProps> = ({
  title,
  points,
  backgroundImage
}) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        backgroundImage: backgroundImage 
          ? `url(${backgroundImage})` 
          : undefined,
        backgroundSize: 'cover'
      }}
    >
      <div style={{
        padding: 60,
        color: 'white',
        fontFamily: 'Inter, sans-serif'
      }}>
        <h1 style={{
          fontSize: 72,
          fontWeight: 'bold',
          opacity: titleOpacity,
          marginBottom: 40
        }}>
          {title}
        </h1>
        
        {points.map((point, index) => {
          const pointOpacity = interpolate(
            frame,
            [60 + (index * 40), 90 + (index * 40)],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <div
              key={index}
              style={{
                fontSize: 36,
                marginBottom: 30,
                opacity: pointOpacity,
                transform: `translateX(${interpolate(
                  frame,
                  [60 + (index * 40), 90 + (index * 40)],
                  [-50, 0],
                  { extrapolateRight: 'clamp' }
                )}px)`
              }}
            >
              {index + 1}. {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### Rendering Videos

```typescript
// lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  title: string,
  points: string[],
  outputPath: string
) {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      points
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      points
    }
  });
  
  return outputPath;
}
```

## Complete Content Pipeline

```typescript
// app/api/pipeline/route.ts
import { NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/render-video';

export async function POST(request: Request) {
  const { keyword, format, languages } = await request.json();
  
  try {
    // Step 1: Research
    const research = await researchTopic(keyword, {
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h'
    });
    
    // Step 2: Generate content in multiple languages
    const content = {};
    for (const lang of languages) {
      const prompt = `
        Based on this research data:
        ${JSON.stringify(research.insights)}
        
        Create a ${format} article about: ${keyword}
      `;
      
      content[lang] = await generateContent(prompt, format, lang);
    }
    
    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(content.en);
    
    // Step 4: Render video
    const videoPath = `/tmp/videos/${Date.now()}.mp4`;
    await renderContentVideo(keyword, keyPoints, videoPath);
    
    return NextResponse.json({
      success: true,
      content,
      videoPath,
      research: research.insights
    });
    
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - in production, use AI to extract
  const lines = content.split('\n')
    .filter(line => line.match(/^\d+\./))
    .slice(0, 5);
  
  return lines.map(line => 
    line.replace(/^\d+\.\s*/, '').trim()
  );
}
```

## Frontend Component Example

```typescript
// components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export function ContentPipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<string>('toplist');
  const [languages, setLanguages] = useState<string[]>(['en', 'vi']);
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format, languages })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block mb-2">Keyword</label>
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          className="w-full px-4 py-2 border rounded"
          placeholder="Enter topic keyword..."
        />
      </div>
      
      <div>
        <label className="block mb-2">Format</label>
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How To</option>
        </select>
      </div>
      
      <button
        type="submit"
        disabled={loading}
        className="px-6 py-2 bg-blue-500 text-white rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-6">
          <h3>Generated Content</h3>
          <div className="space-y-4">
            {Object.entries(result.content).map(([lang, text]) => (
              <div key={lang}>
                <h4>{lang.toUpperCase()}</h4>
                <div className="p-4 bg-gray-100 rounded">
                  {text as string}
                </div>
              </div>
            ))}
          </div>
          
          {result.videoPath && (
            <div className="mt-4">
              <h4>Generated Video</h4>
              <p>Video saved to: {result.videoPath}</p>
            </div>
          )}
        </div>
      )}
    </form>
  );
}
```

## Common Patterns

### Research with Multiple Sources

```typescript
// lib/research/crawler.ts
export async function researchTopic(
  keyword: string,
  options: {
    sources: string[];
    timeRange: string;
  }
) {
  const results = await Promise.all(
    options.sources.map(source => 
      fetchFromSource(source, keyword, options.timeRange)
    )
  );
  
  return {
    articles: results.flat(),
    insights: analyzeArticles(results.flat())
  };
}

async function fetchFromSource(
  source: string,
  keyword: string,
  timeRange: string
) {
  const apiMap = {
    techcrunch: 'techcrunch-api.rapidapi.com',
    a16z: 'a16z-news.rapidapi.com',
    twitter: 'twitter135.p.rapidapi.com'
  };
  
  const response = await fetch(
    `https://${apiMap[source]}/search?q=${keyword}&time=${timeRange}`,
    {
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        'X-RapidAPI-Host': apiMap[source]
      }
    }
  );
  
  return response.json();
}
```

### Batch Content Generation

```typescript
// lib/batch-generator.ts
export async function generateBatchContent(
  keywords: string[],
  format: string
) {
  const results = [];
  
  for (const keyword of keywords) {
    const content = await generateContent(
      `Create a ${format} about ${keyword}`,
      format,
      'en'
    );
    
    results.push({
      keyword,
      content,
      createdAt: new Date()
    });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private requests: number[] = [];
  
  constructor(
    private maxRequests: number,
    private timeWindow: number
  ) {}
  
  async throttle() {
    const now = Date.now();
    this.requests = this.requests.filter(
      time => now - time < this.timeWindow
    );
    
    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = this.timeWindow - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    this.requests.push(Date.now());
  }
}

// Usage
const limiter = new RateLimiter(10, 60000); // 10 requests per minute
await limiter.throttle();
```

### Video Rendering Memory Issues

Add to `remotion.config.ts`:

```typescript
import { Config } from '@remotion/cli/config';

Config.setChromiumOpenGlRenderer('angle');
Config.setChromiumHeadlessMode(true);
Config.setDelayRenderTimeoutInMilliseconds(90000);
Config.setConcurrency(1); // Reduce for low memory
```

### Content Quality Issues

Improve prompts with context:

```typescript
const enhancedPrompt = `
Context: ${research.insights.join('\n')}

Task: Create a ${format} article about ${keyword}

Requirements:
- Use recent data from the research above
- Include specific examples and metrics
- Write in ${language === 'vi' ? 'Vietnamese' : 'English'}
- Tone: Professional yet engaging
- Length: 1500-2000 words
- Include actionable takeaways
`;
```
