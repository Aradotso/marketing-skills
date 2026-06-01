---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline that researches, generates scripts, and produces videos using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I set up the AI content pipeline for automated marketing
  - generate video content from blog posts automatically
  - research and create content using Claude and OpenAI APIs
  - automate content creation workflow with AI
  - create TikTok and Reels videos from articles
  - build an automated content research and writing system
  - integrate Remotion video rendering with AI content generation
  - set up multilingual content generation pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript/Next.js project automates the entire content creation workflow: from researching trending topics, generating multilingual articles, to rendering videos for social media. It combines web scraping, AI content generation (Claude 3, OpenAI), and video rendering (Remotion) into a single pipeline.

## What It Does

- **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for trending topics
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
- **Multilingual Output**: Generates content in both English and Vietnamese simultaneously
- **Video Rendering**: Automatically converts articles into videos/infographics for Reels, TikTok, Shorts
- **Flexible Architecture**: Modular design with API integrations for OpenAI, Anthropic, RapidAPI

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
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   ├── components/        # React components
│   └── pages/            # Page components
├── lib/
│   ├── ai/               # AI integration modules
│   │   ├── claude.ts     # Claude API client
│   │   └── openai.ts     # OpenAI API client
│   ├── research/         # Content research modules
│   │   ├── scraper.ts    # Web scraping logic
│   │   └── analyzer.ts   # Content analysis
│   └── video/           # Video rendering modules
│       └── remotion.ts   # Remotion integration
├── remotion/            # Remotion video templates
└── scripts/            # Utility scripts
```

## Core Components

### 1. Content Research

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  content: string;
  url: string;
  publishedAt: Date;
  source: string;
}

export async function researchTopic(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const response = await axios.get(
      `https://news-api.rapidapi.com/search`,
      {
        params: { q: keyword, source },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          'X-RapidAPI-Host': 'news-api.rapidapi.com'
        }
      }
    );
    
    results.push(...response.data.articles);
  }
  
  return results;
}

export async function analyzeInsights(
  articles: ResearchResult[]
): Promise<string> {
  // Extract key insights and trends from articles
  const combined = articles.map(a => a.content).join('\n\n');
  return combined;
}
```

### 2. AI Content Generation

```typescript
// lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  research: string;
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);
  
  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
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

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with actionable items',
    'pov': 'Write from a unique perspective or opinion',
    'case-study': 'Analyze a specific example with data and outcomes',
    'how-to': 'Provide step-by-step instructions'
  };
  
  return `
You are a content marketing expert. Create a ${request.format} article about "${request.keyword}".

Format: ${formatInstructions[request.format]}
Tone: ${request.tone}
Language: ${request.language}

Research data:
${request.research}

Requirements:
- Use data and statistics from the research
- Include actionable insights
- Optimize for ${request.language === 'en' ? 'English' : 'Vietnamese'} readers
- Length: 800-1200 words
`.trim();
}
```

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!
});

export async function generateContentOpenAI(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer and writer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 2500
  });
  
  return completion.choices[0].message.content || '';
}
```

### 3. Video Rendering

```typescript
// lib/video/remotion.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  style: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function renderVideo(
  config: VideoConfig
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
      content: config.content,
      style: config.style
    }
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.props
  });
  
  return outputLocation;
}
```

### 4. Complete Pipeline

```typescript
// lib/pipeline/content-pipeline.ts
import { researchTopic, analyzeInsights } from '../research/scraper';
import { generateContent } from '../ai/claude';
import { renderVideo } from '../video/remotion';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
}

export async function runContentPipeline(
  config: PipelineConfig
) {
  // Step 1: Research
  console.log('🔍 Researching topic:', config.keyword);
  const articles = await researchTopic(config.keyword);
  const insights = await analyzeInsights(articles);
  
  // Step 2: Generate content in multiple languages
  const contents: Record<string, string> = {};
  
  for (const language of config.languages) {
    console.log(`✍️ Generating ${language} content...`);
    contents[language] = await generateContent({
      keyword: config.keyword,
      format: config.format,
      tone: config.tone,
      language,
      research: insights
    });
  }
  
  // Step 3: Generate video (optional)
  let videoPath: string | null = null;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderVideo({
      title: config.keyword,
      content: contents['en'] || contents['vi'],
      style: 'reels',
      duration: 60
    });
  }
  
  return {
    contents,
    videoPath,
    research: articles
  };
}
```

## API Routes

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      tone: body.tone || 'expert',
      languages: body.languages || ['en', 'vi'],
      generateVideo: body.generateVideo || false
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

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scraper';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const keyword = searchParams.get('keyword');
  
  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }
  
  const articles = await researchTopic(keyword);
  
  return NextResponse.json({
    success: true,
    articles
  });
}
```

## Frontend Usage

```typescript
// app/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          tone: 'expert',
          languages: ['en', 'vi'],
          generateVideo: true
        })
      });
      
      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-4">
        AI Content Pipeline
      </h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter topic keyword..."
          className="w-full p-3 border rounded"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="px-6 py-3 bg-blue-600 text-white rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
        
        {result && (
          <div className="mt-8 space-y-6">
            <div>
              <h2 className="text-xl font-bold mb-2">English Content</h2>
              <div className="prose max-w-none">
                {result.contents.en}
              </div>
            </div>
            
            <div>
              <h2 className="text-xl font-bold mb-2">Vietnamese Content</h2>
              <div className="prose max-w-none">
                {result.contents.vi}
              </div>
            </div>
            
            {result.videoPath && (
              <div>
                <h2 className="text-xl font-bold mb-2">Generated Video</h2>
                <video controls className="w-full max-w-md">
                  <source src={result.videoPath} type="video/mp4" />
                </video>
              </div>
            )}
          </div>
        )}
      </div>
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

# Run specific pipeline script
npm run pipeline -- --keyword "AI trends" --format toplist
```

## Common Patterns

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { runContentPipeline } from '../lib/pipeline/content-pipeline';

const topics = [
  'AI marketing trends 2024',
  'Social media automation',
  'Content creation tools'
];

async function batchGenerate() {
  for (const topic of topics) {
    console.log(`Processing: ${topic}`);
    
    const result = await runContentPipeline({
      keyword: topic,
      format: 'toplist',
      tone: 'expert',
      languages: ['en', 'vi'],
      generateVideo: true
    });
    
    // Save results to database or file system
    console.log(`✅ Completed: ${topic}`);
  }
}

batchGenerate();
```

### Custom Remotion Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

interface Props {
  title: string;
  content: string;
  style: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<Props> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center'
      }}
    >
      <div style={{ 
        color: 'white', 
        fontSize: 60,
        opacity: frame / 30 
      }}>
        {title}
      </div>
      <div style={{ 
        color: 'white', 
        fontSize: 30,
        marginTop: 40 
      }}>
        {content.substring(0, 200)}...
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (maxRetries > 0 && error.status === 429) {
      await new Promise(resolve => setTimeout(resolve, delay));
      return withRetry(fn, maxRetries - 1, delay * 2);
    }
    throw error;
  }
}

// Usage
const content = await withRetry(() => 
  generateContent(request)
);
```

### Video Rendering Memory Issues

- Reduce video resolution in Remotion config
- Use `renderFrames` instead of `renderMedia` for large videos
- Set `REMOTION_CONCURRENCY` environment variable to limit parallel rendering

### Claude API Timeouts

```typescript
// Increase timeout for long content generation
const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
  timeout: 120000 // 2 minutes
});
```

### Missing Research Data

```typescript
// Add fallback when research returns no results
const articles = await researchTopic(keyword);

if (articles.length === 0) {
  // Use AI to generate content without research context
  console.warn('No research data found, using AI knowledge only');
}
```

## Best Practices

1. **Cache Research Results**: Store scraped articles to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BeeQueue) for video generation to avoid blocking
3. **Content Validation**: Add quality checks before publishing generated content
4. **Rate Limiting**: Implement rate limiting for public API endpoints
5. **Error Handling**: Log errors to monitoring service (Sentry, LogRocket)
