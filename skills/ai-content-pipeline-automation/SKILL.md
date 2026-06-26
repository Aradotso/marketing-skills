---
name: ai-content-pipeline-automation
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - help me set up an automated marketing pipeline
  - how to generate videos from written content automatically
  - create a content automation system with Claude and OpenAI
  - automate research and content generation workflow
  - build an AI-powered content pipeline
  - generate social media content automatically with AI
  - set up remotion video rendering for marketing content
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive system for automated content creation from research through video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to transform keywords into complete multi-format content pieces including articles, social posts, and videos.

## What This Project Does

The AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes recent content from TechCrunch, a16z, Twitter, LinkedIn
2. **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically converts content to videos using Remotion for social platforms
5. **Content Scheduling**: Integrates with publishing workflows for automated posting

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

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if using)
DATABASE_URL=your_database_url

# Next.js Config
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Content crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### Content Research & Crawling

```typescript
import { crawlRecentContent } from '@/lib/crawler';

// Crawl recent content by keyword
async function researchTopic(keyword: string) {
  const sources = {
    techcrunch: true,
    twitter: true,
    linkedin: true,
    a16z: true
  };

  const results = await crawlRecentContent({
    keyword,
    sources,
    timeframe: '24h',
    maxResults: 20
  });

  return results;
}

// Example usage
const insights = await researchTopic('AI marketing automation');
console.log(insights.articles); // Array of crawled articles
console.log(insights.trends); // Extracted trends
console.log(insights.statistics); // Data points
```

### AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = `Generate a ${format} article about ${topic} in ${language} with a ${tone} tone. 
  Include data-backed insights and recent trends.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].text;
}

// Generate bilingual content
const englishContent = await generateContent('AI trends', 'toplist', 'en', 'expert');
const vietnameseContent = await generateContent('AI trends', 'toplist', 'vi', 'friendly');
```

### OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function enhanceContent(baseContent: string, researchData: any[]) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a content enhancement expert. Add data-backed insights and improve structure.'
      },
      {
        role: 'user',
        content: `Enhance this content with the following research data:\n\nContent: ${baseContent}\n\nResearch: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(
  contentData: {
    title: string;
    points: string[];
    statistics: any[];
  },
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.tsx'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: contentData,
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${contentData.title.replace(/\s+/g, '-')}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: contentData,
  });

  return outputLocation;
}

// Example usage
const videoPath = await generateVideo({
  title: 'Top 5 AI Marketing Trends 2024',
  points: [
    'AI-powered personalization',
    'Predictive analytics',
    'Automated content creation',
    'Chatbot evolution',
    'Voice search optimization'
  ],
  statistics: [
    { metric: 'ROI Increase', value: '300%' },
    { metric: 'Time Saved', value: '15 hours/week' }
  ]
}, 'reels');
```

## Complete Content Pipeline

```typescript
import { crawlRecentContent } from '@/lib/crawler';
import { generateContentWithAI } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { schedulePost } from '@/lib/publisher';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  autoPublish: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await crawlRecentContent({
      keyword: config.keyword,
      timeframe: '24h',
      sources: { techcrunch: true, twitter: true, linkedin: true }
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const contents = await Promise.all(
      config.languages.map(lang =>
        generateContentWithAI({
          topic: config.keyword,
          format: config.format,
          language: lang,
          researchData: research,
          tone: 'expert'
        })
      )
    );

    // Step 3: Generate Videos (if enabled)
    let videos: string[] = [];
    if (config.generateVideo) {
      console.log('🎬 Rendering videos...');
      videos = await Promise.all(
        contents.map(content =>
          renderContentVideo({
            title: content.title,
            points: content.keyPoints,
            statistics: research.statistics,
            platform: 'reels'
          })
        )
      );
    }

    // Step 4: Schedule Publishing (if enabled)
    if (config.autoPublish) {
      console.log('📅 Scheduling posts...');
      await Promise.all(
        contents.map((content, idx) =>
          schedulePost({
            content: content.body,
            title: content.title,
            videoUrl: videos[idx],
            scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
          })
        )
      );
    }

    return {
      success: true,
      contents,
      videos,
      research
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation 2024',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  autoPublish: false
});
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo || false,
      autoPublish: body.autoPublish || false
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
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
import { crawlRecentContent } from '@/lib/crawler';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }

  const research = await crawlRecentContent({
    keyword,
    timeframe: searchParams.get('timeframe') || '24h',
    sources: {
      techcrunch: true,
      twitter: true,
      linkedin: true,
      a16z: true
    }
  });

  return NextResponse.json(research);
}
```

## React Component Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate() {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          generateVideo: true,
          autoPublish: false
        })
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="max-w-2xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Generator</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter topic keyword..."
          className="w-full p-3 border rounded-lg"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white py-3 rounded-lg disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result && (
          <div className="mt-6 p-4 bg-gray-50 rounded-lg">
            <h3 className="font-semibold mb-2">Result:</h3>
            <pre className="text-sm overflow-auto">
              {JSON.stringify(result, null, 2)}
            </pre>
          </div>
        )}
      </div>
    </div>
  );
}
```

## Common Patterns

### Custom Content Format Templates

```typescript
// lib/templates/content-formats.ts
export const contentTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    aiPrompt: (topic: string, itemCount: number) => 
      `Create a top ${itemCount} list about ${topic}. Include specific examples and data.`
  },
  
  pov: {
    structure: ['hook', 'perspective', 'supporting-points', 'call-to-action'],
    aiPrompt: (topic: string, angle: string) =>
      `Write a POV article about ${topic} from the angle of ${angle}. Be opinionated but data-driven.`
  },
  
  caseStudy: {
    structure: ['challenge', 'solution', 'implementation', 'results'],
    aiPrompt: (topic: string, company: string) =>
      `Create a case study about ${topic} featuring ${company}. Include metrics and outcomes.`
  },
  
  howTo: {
    structure: ['problem', 'overview', 'steps', 'tips', 'conclusion'],
    aiPrompt: (topic: string) =>
      `Write a comprehensive how-to guide for ${topic}. Include step-by-step instructions.`
  }
};
```

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['en'],
        generateVideo: false,
        autoPublish: false
      })
    )
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);

  const failed = results
    .filter(r => r.status === 'rejected')
    .map(r => (r as PromiseRejectedResult).reason);

  return { successful, failed };
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion studio (for video template editing)
npm run remotion:studio

# Render a single video
npm run remotion:render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
  private processing = false;
  private lastCall = 0;
  private minInterval: number;

  constructor(callsPerMinute: number) {
    this.minInterval = 60000 / callsPerMinute;
  }

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const now = Date.now();
          const timeSinceLastCall = now - this.lastCall;
          
          if (timeSinceLastCall < this.minInterval) {
            await new Promise(r => 
              setTimeout(r, this.minInterval - timeSinceLastCall)
            );
          }
          
          this.lastCall = Date.now();
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
    }
    this.processing = false;
  }
}

// Usage
const openaiLimiter = new RateLimiter(50); // 50 calls per minute

const result = await openaiLimiter.add(() =>
  openai.chat.completions.create({...})
);
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering for large videos
const renderConfig = {
  concurrency: 1, // Reduce parallel rendering
  enforceAudioTrack: false,
  muted: false,
  pixelFormat: 'yuv420p',
  codec: 'h264',
  videoBitrate: '2M', // Reduce bitrate if needed
};
```

### Content Quality Validation

```typescript
function validateContentQuality(content: string): {
  valid: boolean;
  issues: string[];
} {
  const issues: string[] = [];
  
  if (content.length < 500) {
    issues.push('Content too short (minimum 500 characters)');
  }
  
  if (content.split('\n\n').length < 3) {
    issues.push('Insufficient paragraph breaks');
  }
  
  const hasNumbers = /\d+%|\d+x|\$\d+/.test(content);
  if (!hasNumbers) {
    issues.push('No statistics or data points found');
  }
  
  return {
    valid: issues.length === 0,
    issues
  };
}
```

This skill provides comprehensive coverage of the AI Content Pipeline system, enabling AI agents to effectively assist developers in setting up and using automated content creation workflows with Claude, OpenAI, and Remotion.
