---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - "help me set up the AI content pipeline"
  - "how do I auto-generate content with Claude and OpenAI"
  - "create automated video content from text"
  - "scrape news and generate social media posts"
  - "build content automation workflow"
  - "generate multilingual content with AI"
  - "render videos from content with Remotion"
  - "automate content research and publishing"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - a comprehensive TypeScript-based system that automates content creation from research/scraping through scriptwriting to video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to create a fully automated content factory.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

1. **Auto-scrapes** trending news from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
3. **Creates bilingual output** (English & Vietnamese) with customizable tone
4. **Renders videos & graphics** automatically using Remotion
5. **Optimizes for platforms** (Reels, TikTok, Shorts) with proper aspect ratios

## Installation

### Prerequisites

```bash
# Required
node >= 18.0.0
npm or yarn or pnpm
```

### Setup Steps

```bash
# Clone repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
# or
pnpm install

# Create environment file
cp .env.example .env
```

### Environment Configuration

Create a `.env` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Content Scraping (RapidAPI or custom)
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# Application runs on http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── scrapers/    # Content scraping modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── services/        # Business logic services
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API & Usage Patterns

### 1. Content Research/Scraping

```typescript
// src/lib/scrapers/newsScaper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}

export async function scrapeRecentNews(
  keyword: string,
  hours: number = 24
): Promise<NewsArticle[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const response = await axios.get('https://api.rapidapi.com/news/search', {
    headers: {
      'X-RapidAPI-Key': rapidApiKey,
      'X-RapidAPI-Host': 'news-api.rapidapi.com'
    },
    params: {
      q: keyword,
      lang: 'en',
      from: new Date(Date.now() - hours * 3600 * 1000).toISOString()
    }
  });

  return response.data.articles.map((article: any) => ({
    title: article.title,
    url: article.url,
    publishedAt: article.publishedAt,
    source: article.source.name,
    content: article.description || article.content
  }));
}

// Usage
const articles = await scrapeRecentNews('AI automation', 24);
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claudeGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: string[];
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const formatPrompts = {
    'toplist': 'Create a numbered list article with actionable items',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze with data, examples, and lessons learned',
    'how-to': 'Step-by-step tutorial with clear instructions'
  };

  const toneAdjustments = {
    'expert': 'Use professional language with industry terminology',
    'friendly': 'Write conversationally with relatable examples',
    'humorous': 'Add wit and entertaining observations'
  };

  const prompt = `
You are an expert content creator. Generate a ${request.format} article about "${request.topic}".

Format Style: ${formatPrompts[request.format]}
Tone: ${toneAdjustments[request.tone]}
Language: ${request.language === 'en' ? 'English' : 'Vietnamese'}

Research Data:
${request.researchData.join('\n\n')}

Requirements:
- Use data from research to support claims
- Include specific examples and statistics
- Create engaging headlines and subheadings
- Optimize for social media sharing
- Length: 1000-1500 words
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openaiGenerator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export async function generateWithGPT(
  request: ContentRequest
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer specializing in engaging, data-driven articles.'
      },
      {
        role: 'user',
        content: `Generate a ${request.format} about ${request.topic} in ${request.language}...`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Bilingual Content Generation

```typescript
// src/lib/generators/bilingualGenerator.ts
export async function generateBilingualContent(
  topic: string,
  researchData: string[]
): Promise<{ english: string; vietnamese: string }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      topic,
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData
    }),
    generateContent({
      topic,
      format: 'toplist',
      tone: 'expert',
      language: 'vi',
      researchData
    })
  ]);

  return {
    english: englishContent,
    vietnamese: vietnameseContent
  };
}
```

### 5. Video Rendering with Remotion

```typescript
// src/lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoContent {
  title: string;
  points: string[];
  brandColor: string;
}

export async function renderContentVideo(
  content: VideoContent,
  outputPath: string
): Promise<string> {
  // Bundle Remotion video
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content
  });

  return outputPath;
}
```

### 6. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  brandColor
}) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title sequence - 0-60 frames (2 seconds at 30fps) */}
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 40
          }}
        >
          <h1
            style={{
              fontSize: 60,
              color: brandColor,
              textAlign: 'center',
              opacity: interpolate(frame, [0, 20], [0, 1])
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Point sequences */}
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60
            }}
          >
            <div
              style={{
                fontSize: 40,
                color: '#fff',
                textAlign: 'center',
                backgroundColor: brandColor,
                padding: 30,
                borderRadius: 20
              }}
            >
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 7. Complete Pipeline Orchestration

```typescript
// src/services/contentPipeline.ts
import { scrapeRecentNews } from '@/lib/scrapers/newsScaper';
import { generateContent } from '@/lib/ai/claudeGenerator';
import { renderContentVideo } from '@/lib/video/renderVideo';

export interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  includeVideo: boolean;
}

export async function runContentPipeline(
  config: PipelineConfig
) {
  // Step 1: Research/Scrape
  console.log('🔍 Scraping news...');
  const articles = await scrapeRecentNews(config.keyword, 24);
  const researchData = articles.map(a => `${a.title}\n${a.content}`);

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent({
    topic: config.keyword,
    format: config.format,
    tone: 'expert',
    language: config.language,
    researchData
  });

  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(content);

  // Step 4: Render Video (if requested)
  let videoPath: string | null = null;
  if (config.includeVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo(
      {
        title: config.keyword,
        points: keyPoints,
        brandColor: '#3b82f6'
      },
      `./output/${Date.now()}.mp4`
    );
  }

  return {
    content,
    videoPath,
    articles: articles.length,
    timestamp: new Date().toISOString()
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - improve based on content structure
  const lines = content.split('\n').filter(line => line.trim());
  return lines
    .filter(line => /^\d+\./.test(line.trim()))
    .slice(0, 5)
    .map(line => line.replace(/^\d+\.\s*/, '').trim());
}
```

### 8. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/services/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, includeVideo } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      includeVideo: includeVideo ?? false
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 9. Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
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
    <div className="p-8 max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">
        AI Content Pipeline
      </h1>

      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter topic keyword..."
          className="w-full px-4 py-2 border rounded-lg"
        />

        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="px-6 py-2 bg-blue-600 text-white rounded-lg disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result && (
          <div className="mt-8 space-y-4">
            <div className="p-4 bg-gray-100 rounded-lg">
              <h2 className="font-bold mb-2">Generated Content:</h2>
              <div className="whitespace-pre-wrap">{result.content}</div>
            </div>

            {result.videoPath && (
              <div className="p-4 bg-blue-50 rounded-lg">
                <h2 className="font-bold mb-2">Video:</h2>
                <video controls src={result.videoPath} className="w-full" />
              </div>
            )}

            <div className="text-sm text-gray-600">
              Analyzed {result.articles} articles
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

## Common Workflows

### Workflow 1: Quick Content Generation

```typescript
// Generate content from keyword only
import { runContentPipeline } from '@/services/contentPipeline';

const result = await runContentPipeline({
  keyword: 'AI marketing automation',
  format: 'toplist',
  language: 'en',
  includeVideo: false
});

console.log(result.content);
```

### Workflow 2: Multilingual Content with Video

```typescript
// Generate both languages with video
const [english, vietnamese] = await Promise.all([
  runContentPipeline({
    keyword: 'content automation',
    format: 'how-to',
    language: 'en',
    includeVideo: true
  }),
  runContentPipeline({
    keyword: 'content automation',
    format: 'how-to',
    language: 'vi',
    includeVideo: true
  })
]);
```

### Workflow 3: Scheduled Content Production

```typescript
// src/lib/scheduler/cronJob.ts
import cron from 'node-cron';
import { runContentPipeline } from '@/services/contentPipeline';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI trends', 'Marketing automation', 'Social media'];
  
  for (const topic of topics) {
    await runContentPipeline({
      keyword: topic,
      format: 'toplist',
      language: 'en',
      includeVideo: true
    });
  }
});
```

## Configuration Options

### AI Provider Selection

```typescript
// src/config/ai.ts
export const aiConfig = {
  provider: process.env.AI_PROVIDER || 'claude', // 'claude' or 'openai'
  models: {
    claude: 'claude-3-5-sonnet-20241022',
    openai: 'gpt-4-turbo-preview'
  },
  temperature: 0.7,
  maxTokens: 4096
};
```

### Video Rendering Options

```typescript
// src/config/video.ts
export const videoConfig = {
  fps: 30,
  width: 1080,
  height: 1920, // 9:16 for Reels/TikTok
  codec: 'h264',
  brandColor: '#3b82f6',
  durationPerPoint: 3 // seconds
};
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rateLimiter.ts
export class RateLimiter {
  private lastCall: number = 0;
  private minInterval: number;

  constructor(callsPerMinute: number) {
    this.minInterval = 60000 / callsPerMinute;
  }

  async throttle() {
    const now = Date.now();
    const timeSinceLastCall = now - this.lastCall;
    
    if (timeSinceLastCall < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastCall)
      );
    }
    
    this.lastCall = Date.now();
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 calls per minute
await limiter.throttle();
await generateContent(request);
```

### Error Handling

```typescript
// src/lib/utils/errorHandler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      console.warn(`Attempt ${i + 1} failed:`, error);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  
  throw lastError!;
}

// Usage
const content = await withRetry(() => 
  generateContent(request)
);
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Missing Environment Variables

```typescript
// src/lib/utils/envValidator.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

## Build and Deploy

```bash
# Build for production
npm run build

# Start production server
npm start

# Build Remotion video compositions
npx remotion bundle remotion/index.ts --bundle-cache ./bundle-cache

# Render single video
npx remotion render ContentVideo output.mp4
```

## Performance Tips

1. **Cache research data** to avoid repeated API calls
2. **Use streaming responses** for real-time content generation
3. **Parallelize** bilingual generation
4. **Pre-render** common video templates
5. **Implement queue system** for video rendering (use BullMQ or similar)

This skill enables comprehensive automation of content creation workflows using cutting-edge AI and video rendering technologies.
