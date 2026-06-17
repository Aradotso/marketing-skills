---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese/English marketing content
triggers:
  - how do I automate content creation with AI research
  - generate marketing content from trending topics automatically
  - create video content from text using Remotion
  - set up AI content pipeline with Claude and OpenAI
  - automate social media content research and generation
  - build automated marketing workflow with video rendering
  - scrape trending news and generate blog posts with AI
  - create multilingual content pipeline for marketing
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based content automation system creates a complete pipeline from research (crawling news sources), through AI content generation (using Claude/OpenAI), to video rendering (via Remotion). It generates Vietnamese and English marketing content in multiple formats (listicles, POV, case studies, how-tos) and automatically creates videos for social media platforms.

## What This Project Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls trending news from TechCrunch, a16z, Twitter, LinkedIn within the last 24 hours
2. **AI Content Generation**: Creates multi-format content (blog posts, social media) in Vietnamese and English using Claude 3 or OpenAI
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-Platform Export**: Outputs content optimized for Reels, TikTok, Shorts

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

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration (optional)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Crawl trending news for a topic
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });
  
  return {
    articles: research.articles,
    trends: research.trends,
    insights: research.insights,
    stats: research.statistics
  };
}

// Example usage
const data = await gatherResearch('AI automation marketing');
console.log(data.articles); // Latest articles
console.log(data.insights); // Key insights extracted
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  researchData: any
) {
  const prompts = {
    toplist: `Create a top 10 list about ${topic} based on this research data: ${JSON.stringify(researchData)}`,
    pov: `Write a POV (point of view) article about ${topic} with insights from: ${JSON.stringify(researchData)}`,
    'case-study': `Develop a case study on ${topic} using these data points: ${JSON.stringify(researchData)}`,
    'how-to': `Create a how-to guide for ${topic} incorporating: ${JSON.stringify(researchData)}`
  };

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `${prompts[format]}. Write in ${language === 'vi' ? 'Vietnamese' : 'English'}. Make it engaging and data-backed.`
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Example usage
const content = await generateContentWithClaude(
  'AI Content Marketing Trends 2026',
  'toplist',
  'vi',
  researchData
);
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(
  topic: string,
  tone: 'expert' | 'friendly' | 'humorous',
  researchData: any
) {
  const tonePrompts = {
    expert: 'Use professional, authoritative language with industry terminology',
    friendly: 'Use conversational, approachable language that builds connection',
    humorous: 'Use light humor and entertaining analogies while staying informative'
  };

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content marketer. ${tonePrompts[tone]}.`
      },
      {
        role: 'user',
        content: `Create compelling content about "${topic}" using this research: ${JSON.stringify(researchData)}`
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
interface BilingualContent {
  en: string;
  vi: string;
  metadata: {
    topic: string;
    format: string;
    generatedAt: Date;
  };
}

async function generateBilingualContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  researchData: any
): Promise<BilingualContent> {
  const [enContent, viContent] = await Promise.all([
    generateContentWithClaude(topic, format, 'en', researchData),
    generateContentWithClaude(topic, format, 'vi', researchData)
  ]);

  return {
    en: enContent,
    vi: viContent,
    metadata: {
      topic,
      format,
      generatedAt: new Date()
    }
  };
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  content: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const platformSpecs = {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 }
  };

  const specs = platformSpecs[platform];
  
  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      content,
      ...specs
    }
  });

  // Render video
  const outputPath = path.join(
    process.cwd(), 
    'public', 
    'videos', 
    `${Date.now()}-${platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content,
      ...specs
    }
  });

  return outputPath;
}

// Example usage
const videoPath = await renderContentVideo(
  generatedContent,
  'reels'
);
```

## Complete Content Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/content/generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface ContentPipelineResult {
  research: any;
  content: BilingualContent;
  videos: {
    reels: string;
    tiktok: string;
    shorts: string;
  };
}

async function runContentPipeline(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
): Promise<ContentPipelineResult> {
  
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });

  // Step 2: Generate bilingual content
  console.log('✍️ Generating content...');
  const content = await generateBilingualContent(
    keyword,
    format,
    research
  );

  // Step 3: Render videos for all platforms
  console.log('🎬 Rendering videos...');
  const [reelsVideo, tiktokVideo, shortsVideo] = await Promise.all([
    renderContentVideo(content.en, 'reels'),
    renderContentVideo(content.vi, 'tiktok'),
    renderContentVideo(content.en, 'shorts')
  ]);

  return {
    research,
    content,
    videos: {
      reels: reelsVideo,
      tiktok: tiktokVideo,
      shorts: shortsVideo
    }
  };
}

// Execute pipeline
const result = await runContentPipeline(
  'AI automation in marketing 2026',
  'toplist'
);

console.log('Content (EN):', result.content.en);
console.log('Content (VI):', result.content.vi);
console.log('Videos:', result.videos);
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format } = await request.json();

    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing keyword or format' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword, format);

    return NextResponse.json({
      success: true,
      data: result
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeRange } = await request.json();

    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeRange: timeRange || '24h',
      language: 'en'
    });

    return NextResponse.json({
      success: true,
      data: research
    });

  } catch (error) {
    return NextResponse.json(
      { error: 'Research failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format })
      });

      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold mb-4">AI Content Generator</h1>
      
      <input
        type="text"
        placeholder="Enter topic keyword..."
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        className="border p-2 w-full mb-4"
      />

      <select
        value={format}
        onChange={(e) => setFormat(e.target.value as any)}
        className="border p-2 w-full mb-4"
      >
        <option value="toplist">Top List</option>
        <option value="pov">POV Article</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-To Guide</option>
      </select>

      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-500 text-white px-6 py-2 rounded disabled:bg-gray-300"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h2 className="font-bold mb-2">English Version:</h2>
          <div className="bg-gray-100 p-4 mb-4 rounded">{result.content.en}</div>
          
          <h2 className="font-bold mb-2">Vietnamese Version:</h2>
          <div className="bg-gray-100 p-4 mb-4 rounded">{result.content.vi}</div>

          <h2 className="font-bold mb-2">Generated Videos:</h2>
          <ul className="list-disc pl-6">
            <li>Reels: {result.videos.reels}</li>
            <li>TikTok: {result.videos.tiktok}</li>
            <li>Shorts: {result.videos.shorts}</li>
          </ul>
        </div>
      )}
    </div>
  );
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video locally
npm run remotion:preview

# Build Remotion video
npm run remotion:render
```

## Common Patterns

### Scheduled Content Generation

```typescript
// Schedule daily content generation
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = [
    'AI marketing trends',
    'Social media automation',
    'Content creation tools'
  ];

  for (const topic of topics) {
    await runContentPipeline(topic, 'toplist');
  }
});
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runContentPipeline(keyword, 'toplist')
    )
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Custom AI Prompts

```typescript
interface CustomPrompt {
  systemRole: string;
  userTemplate: string;
  temperature: number;
}

async function generateWithCustomPrompt(
  topic: string,
  customPrompt: CustomPrompt,
  research: any
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: customPrompt.systemRole },
      { 
        role: 'user', 
        content: customPrompt.userTemplate
          .replace('{topic}', topic)
          .replace('{research}', JSON.stringify(research))
      }
    ],
    temperature: customPrompt.temperature
  });

  return completion.choices[0].message.content;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error?.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await retryWithBackoff(() => 
  generateContentWithClaude(topic, format, 'en', research)
);
```

### Video Rendering Memory Issues

```typescript
// Use concurrency limits for video rendering
import pLimit from 'p-limit';

const limit = pLimit(2); // Only render 2 videos at a time

async function renderMultipleVideos(contents: string[]) {
  const tasks = contents.map((content, index) => 
    limit(() => renderContentVideo(content, 'reels'))
  );
  
  return Promise.all(tasks);
}
```

### Environment Variable Validation

```typescript
// src/lib/config/validate.ts
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call on app startup
validateEnv();
```

### Content Quality Validation

```typescript
function validateGeneratedContent(content: string): boolean {
  return (
    content.length >= 500 && // Minimum length
    content.length <= 10000 && // Maximum length
    !content.includes('[ERROR]') &&
    !content.includes('I apologize')
  );
}

async function generateValidatedContent(...args: any[]) {
  const content = await generateContentWithClaude(...args);
  
  if (!validateGeneratedContent(content)) {
    throw new Error('Generated content failed validation');
  }
  
  return content;
}
```

This pipeline system is designed for marketers and content creators who need to scale content production while maintaining quality through AI-powered research, generation, and video rendering.
