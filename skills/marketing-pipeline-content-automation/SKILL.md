---
name: marketing-pipeline-content-automation
description: AI-powered content pipeline for automated research, scriptwriting, video generation and multi-format content creation
triggers:
  - automate content creation with AI research and video generation
  - set up automated marketing content pipeline
  - generate videos from text using Remotion
  - create multilingual content with Claude and OpenAI
  - scrape trending news for content research
  - build automated social media content workflow
  - generate infographics and short-form videos automatically
  - schedule automated content publishing
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to use the **Ultimate AI Content Pipeline** - a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline crawls trending news, generates multi-format content in multiple languages, and renders videos/infographics automatically using Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter/X, LinkedIn for trending topics
2. **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Converts content to short-form videos and infographics via Remotion
5. **Platform Optimization**: Exports for Reels, TikTok, Shorts with proper aspect ratios

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

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=your_database_connection

# Optional: Social media auto-posting
FACEBOOK_PAGE_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

### Development Server

```bash
# Start the Next.js development server
npm run dev
# or
yarn dev

# Open http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key APIs and Usage Patterns

### 1. News Research & Crawling

```typescript
// src/lib/crawler/news-scraper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  summary: string;
}

export async function scrapeNewsForTopic(
  topic: string,
  timeRange: '24h' | '7d' = '24h'
): Promise<NewsArticle[]> {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  const articles: NewsArticle[] = [];

  for (const source of sources) {
    const response = await axios.get(
      `https://api.rapidapi.com/v1/news/${source}`,
      {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'X-RapidAPI-Host': 'news-api.rapidapi.com',
        },
        params: {
          q: topic,
          timeRange,
          language: 'en',
        },
      }
    );

    articles.push(...response.data.articles);
  }

  return articles;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const systemPrompt = buildSystemPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Generate a ${request.format} article about "${request.topic}" 
                  in ${request.language} with a ${request.tone} tone.
                  Use this research data: ${JSON.stringify(request.researchData)}`,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear benefits and examples',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Analyze a real example with data and insights',
    'how-to': 'Provide step-by-step instructions with actionable tips',
  };

  return `You are an expert content creator specializing in ${request.format} articles.
${formatInstructions[request.format]}
Always include recent data, statistics, and credible sources.
Write in ${request.language} with a ${request.tone} tone.`;
}
```

### 3. OpenAI Integration (Alternative)

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentOpenAI(
  topic: string,
  researchData: any[]
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a marketing content expert who creates engaging, data-driven articles.',
      },
      {
        role: 'user',
        content: `Create an article about "${topic}" using this research: 
                  ${JSON.stringify(researchData)}`,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function generateVideo(config: VideoConfig): Promise<string> {
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
      platform: config.platform,
    },
  });

  const dimensions = getPlatformDimensions(config.platform);
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition: {
      ...composition,
      width: dimensions.width,
      height: dimensions.height,
      durationInFrames: config.duration * 30, // 30 fps
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps,
  });

  return outputLocation;
}

function getPlatformDimensions(platform: string) {
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
    default: { width: 1920, height: 1080 },
  };
  return dimensions[platform] || dimensions.default;
}
```

### 5. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { scrapeNewsForTopic } from '../crawler/news-scraper';
import { generateContent } from '../ai/content-generator';
import { generateVideo } from '../video/render-video';

interface PipelineConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const researchData = await scrapeNewsForTopic(config.topic, '24h');

    // Step 2: Generate content for each language
    console.log('✍️ Generating content...');
    const contents = {};
    
    for (const lang of config.languages) {
      const content = await generateContent({
        topic: config.topic,
        format: config.format,
        language: lang,
        tone: 'professional',
        researchData,
      });
      contents[lang] = content;
    }

    // Step 3: Generate video if requested
    let videoPath = null;
    if (config.generateVideo && config.platform) {
      console.log('🎬 Rendering video...');
      videoPath = await generateVideo({
        content: contents['en'],
        title: config.topic,
        platform: config.platform,
        duration: 30,
      });
    }

    console.log('✅ Pipeline complete!');
    return {
      research: researchData,
      contents,
      videoPath,
    };
  } catch (error) {
    console.error('❌ Pipeline failed:', error);
    throw error;
  }
}
```

### 6. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { topic, format, languages, generateVideo, platform } = body;

    if (!topic || !format) {
      return NextResponse.json(
        { error: 'Topic and format are required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      topic,
      format,
      languages: languages || ['en', 'vi'],
      generateVideo: generateVideo || false,
      platform,
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
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
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.currentTarget);
    const payload = {
      topic: formData.get('topic'),
      format: formData.get('format'),
      languages: ['en', 'vi'],
      generateVideo: formData.get('generateVideo') === 'on',
      platform: formData.get('platform'),
    };

    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
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
    <form onSubmit={handleGenerate} className="space-y-4">
      <input
        name="topic"
        type="text"
        placeholder="Enter topic (e.g., AI Marketing Tools)"
        required
        className="w-full p-2 border rounded"
      />
      
      <select name="format" required className="w-full p-2 border rounded">
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-to Guide</option>
      </select>

      <label className="flex items-center gap-2">
        <input type="checkbox" name="generateVideo" />
        Generate Video
      </label>

      <select name="platform" className="w-full p-2 border rounded">
        <option value="reels">Instagram Reels</option>
        <option value="tiktok">TikTok</option>
        <option value="shorts">YouTube Shorts</option>
      </select>

      <button
        type="submit"
        disabled={loading}
        className="w-full bg-blue-600 text-white p-2 rounded disabled:bg-gray-400"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-4 p-4 bg-gray-100 rounded">
          <h3 className="font-bold">Results:</h3>
          <pre className="mt-2 text-sm overflow-auto">
            {JSON.stringify(result, null, 2)}
          </pre>
        </div>
      )}
    </form>
  );
}
```

## Configuration Files

### TypeScript Configuration

Ensure `tsconfig.json` includes:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### Remotion Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function generateBatchContent(topics: string[]) {
  const results = await Promise.all(
    topics.map(topic =>
      runContentPipeline({
        topic,
        format: 'toplist',
        languages: ['en', 'vi'],
        generateVideo: false,
      })
    )
  );

  return results;
}
```

### Pattern 2: Scheduled Content Creation

```typescript
// Using node-cron for scheduling
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics.slice(0, 3)) {
    await runContentPipeline({
      topic: topic.name,
      format: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true,
      platform: 'reels',
    });
  }
});
```

### Pattern 3: Content Variation Testing

```typescript
async function generateVariations(topic: string) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'] as const;
  const variations = {};

  for (const format of formats) {
    variations[format] = await generateContent({
      topic,
      format,
      language: 'en',
      tone: 'professional',
      researchData: [],
    });
  }

  return variations;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  items.map(item => limit(() => apiCall(item)))
);
```

### Video Rendering Memory Issues

```typescript
// Use smaller chunks and cleanup
Config.setConcurrency(2); // Reduce concurrent renders
Config.setChromiumDisableWebSecurity(true);

// Clean up after rendering
import { cleanupArtifacts } from '@remotion/renderer';
await cleanupArtifacts();
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at startup
validateEnv();
```

### Error Handling Best Practices

```typescript
async function safeGenerateContent(config: ContentRequest) {
  const maxRetries = 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      return await generateContent(config);
    } catch (error) {
      attempt++;
      if (attempt >= maxRetries) throw error;
      
      // Exponential backoff
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, attempt) * 1000)
      );
    }
  }
}
```

## Build and Deployment

```bash
# Build for production
npm run build

# Start production server
npm run start

# Build Remotion compositions
npx remotion bundle remotion/index.ts public/bundle

# Render specific video
npx remotion render public/bundle ContentVideo output.mp4
```

This skill provides comprehensive coverage of the marketing content automation pipeline, enabling AI agents to help developers implement automated content creation workflows with research, AI generation, and video rendering capabilities.
