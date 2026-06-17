---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate videos from research automatically
  - create multi-format content with Claude
  - crawl news sources for content ideas
  - build automated marketing content system
  - render videos from text using Remotion
  - setup AI content generation workflow
  - create bilingual content automatically
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Marketing Pipeline Share**, an end-to-end automated content creation system that handles research, scriptwriting, multi-format content generation, and video rendering using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

Marketing Pipeline Share automates the entire content creation pipeline:

1. **Auto-Research**: Crawls fresh content from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates multiple content formats (listicles, POV, case studies, how-tos) in Vietnamese and English
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-Platform Export**: Outputs optimized video formats for Reels, TikTok, Shorts

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

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News/content crawlers
│   │   └── video/       # Remotion video generation
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Video rendering (Remotion)
npm run video:preview
npm run video:render
```

## Core APIs and Usage Patterns

### 1. Content Research & Crawling

```typescript
// src/lib/crawlers/newsScanner.ts
import { RapidAPI } from '@/lib/api/rapid';

interface NewsSource {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
}

export async function scanLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsSource[]> {
  const rapidAPI = new RapidAPI(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    sources.map(source => 
      rapidAPI.searchNews({
        query: keyword,
        source,
        timeRange: '24h'
      })
    )
  );
  
  return results.flat();
}

// Usage
const newsData = await scanLatestNews('AI marketing automation');
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'vi' | 'en';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContent(
  keyword: string,
  options: ContentOptions
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!,
  });

  const prompt = buildPrompt(keyword, options);

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

function buildPrompt(keyword: string, options: ContentOptions): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article format',
    'pov': 'Write from a unique perspective or opinion',
    'case-study': 'Analyze a specific real-world example',
    'how-to': 'Provide step-by-step instructions'
  };

  return `
You are a ${options.tone} content writer. 
Create a ${options.format} article about "${keyword}" in ${options.language}.

Format: ${formatInstructions[options.format]}

Research Data:
${options.researchData}

Requirements:
- Use data-backed insights from the research
- ${options.language === 'vi' ? 'Write in Vietnamese' : 'Write in English'}
- Tone: ${options.tone}
- Include statistics and examples
- Optimize for social media engagement
`;
}
```

### 3. Bilingual Content Generation

```typescript
// src/lib/ai/bilingualGenerator.ts
export async function generateBilingualContent(
  keyword: string,
  researchData: string,
  format: ContentOptions['format']
): Promise<{ vi: string; en: string }> {
  const [vietnamese, english] = await Promise.all([
    generateContent(keyword, {
      format,
      language: 'vi',
      tone: 'friendly',
      researchData
    }),
    generateContent(keyword, {
      format,
      language: 'en',
      tone: 'expert',
      researchData
    })
  ]);

  return { vi: vietnamese, en: english };
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/videoRenderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundled = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      title: config.title,
      ...compositions[config.format]
    }
  });

  const outputPath = `output/${Date.now()}-${config.format}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
  });

  return outputPath;
}
```

### 5. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/contentPipeline.ts
export class ContentPipeline {
  async execute(keyword: string, format: ContentOptions['format']) {
    // Step 1: Research
    console.log('🔍 Scanning latest news...');
    const newsData = await scanLatestNews(keyword);
    const researchSummary = newsData
      .map(n => `${n.title}: ${n.content}`)
      .join('\n\n');

    // Step 2: Generate content
    console.log('✍️ Generating bilingual content...');
    const content = await generateBilingualContent(
      keyword,
      researchSummary,
      format
    );

    // Step 3: Render videos
    console.log('🎬 Rendering videos...');
    const videos = await Promise.all([
      renderContentVideo({
        content: content.vi,
        title: keyword,
        format: 'reels'
      }),
      renderContentVideo({
        content: content.en,
        title: keyword,
        format: 'shorts'
      })
    ]);

    return {
      content,
      videos,
      research: newsData
    };
  }
}

// Usage
const pipeline = new ContentPipeline();
const result = await pipeline.execute('AI automation trends', 'toplist');
```

## Next.js API Routes

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format } = await request.json();

    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(keyword, format);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: (error as Error).message },
      { status: 500 }
    );
  }
}
```

## Frontend Components

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
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
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-6">
          <div>
            <h2 className="text-2xl font-bold mb-2">Vietnamese Content</h2>
            <div className="prose">{result.content.vi}</div>
          </div>
          <div>
            <h2 className="text-2xl font-bold mb-2">English Content</h2>
            <div className="prose">{result.content.en}</div>
          </div>
          <div>
            <h2 className="text-2xl font-bold mb-2">Generated Videos</h2>
            {result.videos.map((video: string, i: number) => (
              <a key={i} href={video} className="block text-blue-600">
                Video {i + 1}: {video}
              </a>
            ))}
          </div>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Custom Tone Configuration

```typescript
// src/lib/ai/tones.ts
export const TONE_PRESETS = {
  expert: {
    systemPrompt: 'You are an industry expert with deep technical knowledge',
    style: 'formal, data-driven, authoritative'
  },
  friendly: {
    systemPrompt: 'You are a helpful friend sharing valuable insights',
    style: 'conversational, warm, accessible'
  },
  humorous: {
    systemPrompt: 'You are a witty content creator who makes learning fun',
    style: 'entertaining, light-hearted, engaging'
  }
};
```

### Content Scheduling

```typescript
// src/lib/scheduler/contentScheduler.ts
export async function scheduleContent(
  content: string,
  platform: 'facebook' | 'twitter' | 'linkedin',
  scheduledTime: Date
) {
  // Integration with social media APIs
  // Store in database for scheduled posting
  return {
    id: generateId(),
    content,
    platform,
    scheduledFor: scheduledTime,
    status: 'pending'
  };
}
```

## Troubleshooting

### API Key Issues

If you get authentication errors:
- Verify all API keys are correctly set in `.env.local`
- Check API key permissions and quota limits
- Ensure no trailing spaces in environment variables

```typescript
// Validate env vars on startup
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY is required');
}
```

### Video Rendering Fails

If Remotion rendering fails:
- Ensure ffmpeg is installed: `brew install ffmpeg` (macOS) or `apt-get install ffmpeg` (Linux)
- Check disk space for output files
- Verify composition ID matches your Remotion setup

### Rate Limiting

Handle API rate limits gracefully:

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries reached');
}
```

### Memory Issues with Large Content

For processing large volumes:

```typescript
// Use streaming for large content
import { Stream } from '@anthropic-ai/sdk/streaming';

async function streamContent(prompt: string) {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: prompt }],
  });

  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta') {
      process.stdout.write(chunk.delta.text);
    }
  }
}
```

This skill enables AI agents to effectively work with the Marketing Pipeline Share system for automated, AI-powered content creation and distribution.
