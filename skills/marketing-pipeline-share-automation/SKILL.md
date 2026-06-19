---
name: marketing-pipeline-share-automation
description: Full-stack AI content pipeline for automated research, script generation, and video production using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline with Claude and OpenAI
  - generate videos from content automatically
  - build an AI content research and writing system
  - create automated social media content pipeline
  - use Remotion to render videos from scripts
  - integrate Claude API for content generation
  - scrape news and generate content automatically
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation pipeline built with TypeScript, Next.js, and Remotion. It automates the entire content creation workflow: from research (scraping news sources), to script generation (using Claude/OpenAI), to video rendering. The system can produce multilingual content (English/Vietnamese) in various formats (toplist, POV, case study, how-to) and automatically generate videos optimized for TikTok, Reels, and Shorts.

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

## Environment Configuration

Required environment variables in `.env.local`:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Remotion Studio (for video preview/editing)
npm run remotion
```

## Core Architecture

The pipeline follows this flow:

1. **Research Module**: Scrapes news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Uses Claude/OpenAI to generate scripts based on research
3. **Video Rendering**: Uses Remotion to create videos from generated content
4. **Publishing**: Outputs ready-to-post content for social platforms

## Key Code Patterns

### 1. Research/Scraping Module

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
  source: string;
}

export async function scrapeNews(keyword: string, hours: number = 24): Promise<NewsArticle[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  try {
    const response = await axios.get('https://api.rapidapi.com/news/search', {
      params: {
        q: keyword,
        time_range: `${hours}h`,
        lang: 'en'
      },
      headers: {
        'X-RapidAPI-Key': rapidApiKey,
        'X-RapidAPI-Host': 'news-api.rapidapi.com'
      }
    });
    
    return response.data.articles.map((article: any) => ({
      title: article.title,
      url: article.url,
      publishedAt: article.publishedAt,
      content: article.content,
      source: article.source.name
    }));
  } catch (error) {
    console.error('Error scraping news:', error);
    return [];
  }
}
```

### 2. Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  keywords: string[];
  researchData: string;
}

export async function generateContent(config: ContentConfig): Promise<string> {
  const systemPrompt = `You are an expert content creator specializing in ${config.format} format. 
Write in ${config.language === 'en' ? 'English' : 'Vietnamese'} with a ${config.tone} tone.
Use the provided research data to create data-backed, trendy content.`;

  const userPrompt = `Create a ${config.format} article about ${config.keywords.join(', ')} 
using this research data:

${config.researchData}

Requirements:
- Include specific data points and statistics
- Make it engaging and shareable
- Optimize for social media
- Length: 800-1200 words`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: userPrompt
    }]
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. Content Generation with OpenAI

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentOpenAI(config: ContentConfig): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${config.format} format. 
Write in ${config.language === 'en' ? 'English' : 'Vietnamese'} with a ${config.tone} tone.`
      },
      {
        role: 'user',
        content: `Create a ${config.format} article about ${config.keywords.join(', ')} 
using this research data:\n\n${config.researchData}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoProps {
  title: string;
  keyPoints: string[];
  backgroundColor: string;
}

export const ContentVideo: React.FC<VideoProps> = ({ title, keyPoints, backgroundColor }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity 
        }}>
          <h1 style={{ 
            fontSize: 60, 
            color: 'white',
            textAlign: 'center',
            padding: '0 50px'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {keyPoints.map((point, index) => (
        <Sequence 
          key={index} 
          from={fps * (3 + index * 2)} 
          durationInFrames={fps * 2}
        >
          <AbsoluteFill style={{ 
            justifyContent: 'center', 
            alignItems: 'center' 
          }}>
            <p style={{ 
              fontSize: 40, 
              color: 'white',
              textAlign: 'center',
              padding: '0 80px'
            }}>
              {point}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// remotion/Root.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './VideoComposition';

export const RemotionRoot: React.FC = () => {
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
          title: 'AI Content Revolution',
          keyPoints: [
            'Automate research in seconds',
            'Generate engaging content',
            'Create videos automatically',
          ],
          backgroundColor: '#4f46e5',
        }}
      />
    </>
  );
};
```

### 5. Render Video Programmatically

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderConfig {
  title: string;
  keyPoints: string[];
  outputPath: string;
}

export async function renderVideo(config: RenderConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public/videos',
    config.outputPath
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
      backgroundColor: '#4f46e5',
    },
  });

  return outputLocation;
}
```

### 6. Full Pipeline API Route

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNews } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderVideo } from '@/lib/video/render';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone } = await request.json();

    // Step 1: Research
    const articles = await scrapeNews(keyword, 24);
    const researchData = articles
      .map(a => `${a.title}\n${a.content}`)
      .join('\n\n');

    // Step 2: Generate Content
    const content = await generateContent({
      format,
      language,
      tone,
      keywords: [keyword],
      researchData,
    });

    // Step 3: Extract key points for video
    const keyPoints = content
      .split('\n')
      .filter(line => line.startsWith('-') || line.startsWith('•'))
      .slice(0, 5)
      .map(line => line.replace(/^[-•]\s*/, ''));

    // Step 4: Render Video
    const videoPath = await renderVideo({
      title: keyword,
      keyPoints,
      outputPath: `${Date.now()}.mp4`,
    });

    return NextResponse.json({
      success: true,
      content,
      videoUrl: videoPath.replace(process.cwd() + '/public', ''),
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Integration

```typescript
// app/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [language, setLanguage] = useState<'en' | 'vi'>('en');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language,
          tone: 'expert',
        }),
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
    <div className="p-6 max-w-4xl mx-auto">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 w-full mb-4"
      />
      
      <select value={format} onChange={(e) => setFormat(e.target.value as any)}>
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How To</option>
      </select>

      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-4 py-2 rounded ml-4"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h2 className="text-2xl font-bold mb-4">Generated Content</h2>
          <div className="prose mb-6">{result.content}</div>
          
          {result.videoUrl && (
            <video controls className="w-full">
              <source src={result.videoUrl} type="video/mp4" />
            </video>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Workflows

### Workflow 1: Generate Content Only

```typescript
import { scrapeNews } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/claude-generator';

const articles = await scrapeNews('AI marketing', 24);
const researchData = articles.map(a => a.content).join('\n\n');

const content = await generateContent({
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  keywords: ['AI marketing', 'automation'],
  researchData,
});

console.log(content);
```

### Workflow 2: Batch Video Generation

```typescript
import { renderVideo } from '@/lib/video/render';

const topics = ['AI Trends', 'Marketing Automation', 'Content Creation'];

for (const topic of topics) {
  await renderVideo({
    title: topic,
    keyPoints: [
      'Key insight 1',
      'Key insight 2',
      'Key insight 3',
    ],
    outputPath: `${topic.toLowerCase().replace(/\s+/g, '-')}.mp4`,
  });
}
```

## Troubleshooting

### API Rate Limits

If you hit rate limits with Claude or OpenAI:

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(config: ContentConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(config);
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Remotion Rendering Issues

Ensure ffmpeg is installed:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows (via Chocolatey)
choco install ffmpeg
```

### Memory Issues with Video Rendering

For large videos, increase Node.js memory:

```bash
NODE_OPTIONS=--max-old-space-size=4096 npm run dev
```

## Project Structure

```
marketing-pineline-share/
├── app/
│   ├── api/
│   │   └── generate-content/
│   │       └── route.ts
│   ├── components/
│   │   └── ContentGenerator.tsx
│   └── page.tsx
├── lib/
│   ├── ai/
│   │   ├── claude-generator.ts
│   │   └── openai-generator.ts
│   ├── research/
│   │   └── scraper.ts
│   └── video/
│       └── render.ts
├── remotion/
│   ├── VideoComposition.tsx
│   ├── Root.tsx
│   └── index.ts
├── public/
│   └── videos/
├── .env.local
└── package.json
```

This skill enables AI agents to effectively utilize the marketing-pipeline-share project for automated content generation, research automation, and video production at scale.
