---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for Vietnamese and English marketing content
triggers:
  - how do I automate content creation with AI
  - generate marketing content from research to video
  - create content pipeline with Claude and OpenAI
  - automate Vietnamese and English content generation
  - build AI content workflow with Remotion
  - set up automated marketing content system
  - research and generate content automatically
  - create videos from written content automatically
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation pipeline that handles research, scriptwriting, and video generation. It automatically crawls news sources (TechCrunch, a16z, Twitter/X, LinkedIn), generates content in multiple formats (toplist, POV, case study, how-to), and renders videos using Remotion. Built with Next.js and TypeScript, supporting both Vietnamese and English content.

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

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research API Keys (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research crawlers
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── config/          # Configuration files
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Content Research Pipeline

The research module automatically crawls and analyzes content from multiple sources:

```typescript
// src/lib/research/crawler.ts
import { ResearchCrawler } from '@/lib/research/crawler';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeframe: '24h' | '7d' | '30d';
  language: 'en' | 'vi' | 'both';
}

async function researchTopic(config: ResearchConfig) {
  const crawler = new ResearchCrawler({
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  const results = await crawler.scan({
    keyword: config.keyword,
    sources: config.sources,
    timeframe: config.timeframe,
  });

  return results;
}

// Example usage
const insights = await researchTopic({
  keyword: 'AI marketing automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'both',
});
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!,
    });
  }

  async generateWithClaude(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);

    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async generateWithOpenAI(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);

    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator specializing in marketing.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      max_tokens: 4096,
    });

    return completion.choices[0].message.content || '';
  }

  private buildPrompt(config: ContentConfig): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with data-backed insights',
      'pov': 'Write a perspective piece with unique viewpoint',
      'case-study': 'Analyze a real-world example with results',
      'how-to': 'Create step-by-step instructional content',
    };

    return `
Create ${config.language === 'vi' ? 'Vietnamese' : 'English'} content in ${config.format} format.
Tone: ${config.tone}
Format: ${formatInstructions[config.format]}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Generate comprehensive, engaging content with:
- Attention-grabbing headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure
`;
  }
}

// Usage example
const generator = new ContentGenerator();

const content = await generator.generateWithClaude({
  format: 'toplist',
  tone: 'expert',
  language: 'vi',
  researchData: insights,
});
```

### 3. Video Generation with Remotion

Transform written content into videos:

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
  duration: number;
  template: 'reels' | 'tiktok' | 'shorts';
}

class VideoRenderer {
  async renderVideo(config: VideoConfig): Promise<string> {
    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config,
    });

    // Get composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: config.template,
      inputProps: {
        title: config.title,
        content: config.content,
        aspectRatio: config.aspectRatio,
      },
    });

    // Render video
    const outputPath = path.join(
      process.cwd(),
      'public',
      'videos',
      `${Date.now()}.mp4`
    );

    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        title: config.title,
        content: config.content,
      },
    });

    return outputPath;
  }
}

// Usage
const renderer = new VideoRenderer();

const videoPath = await renderer.renderVideo({
  content: generatedContent,
  title: 'Top 5 AI Marketing Trends',
  aspectRatio: '9:16',
  duration: 60,
  template: 'reels',
});
```

### 4. Complete Pipeline API Route

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchCrawler } from '@/lib/research/crawler';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, generateVideo } = body;

    // Step 1: Research
    const crawler = new ResearchCrawler({
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });

    const research = await crawler.scan({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h',
    });

    // Step 2: Generate Content
    const generator = new ContentGenerator();
    
    const content = await generator.generateWithClaude({
      format,
      tone: 'expert',
      language,
      researchData: research,
    });

    let videoUrl = null;

    // Step 3: Generate Video (optional)
    if (generateVideo) {
      const renderer = new VideoRenderer();
      videoUrl = await renderer.renderVideo({
        content,
        title: keyword,
        aspectRatio: '9:16',
        duration: 60,
        template: 'reels',
      });
    }

    return NextResponse.json({
      success: true,
      data: {
        research,
        content,
        videoUrl,
      },
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

### 5. Frontend Component Example

```typescript
// src/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

interface PipelineResult {
  research: any;
  content: string;
  videoUrl?: string;
}

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [language, setLanguage] = useState<'en' | 'vi'>('vi');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<PipelineResult | null>(null);

  const runPipeline = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language,
          generateVideo: true,
        }),
      });

      const data = await response.json();
      if (data.success) {
        setResult(data.data);
      }
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4 mb-6">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI marketing)"
          className="w-full px-4 py-2 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To Guide</option>
        </select>

        <select
          value={language}
          onChange={(e) => setLanguage(e.target.value as any)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="vi">Tiếng Việt</option>
          <option value="en">English</option>
        </select>

        <button
          onClick={runPipeline}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white px-6 py-3 rounded disabled:bg-gray-400"
        >
          {loading ? 'Processing...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="space-y-6">
          <div className="bg-white p-6 rounded shadow">
            <h2 className="text-xl font-semibold mb-4">Generated Content</h2>
            <div className="prose max-w-none">
              {result.content}
            </div>
          </div>

          {result.videoUrl && (
            <div className="bg-white p-6 rounded shadow">
              <h2 className="text-xl font-semibold mb-4">Generated Video</h2>
              <video controls className="w-full">
                <source src={result.videoUrl} type="video/mp4" />
              </video>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Bilingual Content Generation

```typescript
async function generateBilingualContent(keyword: string) {
  const generator = new ContentGenerator();
  
  const [englishContent, vietnameseContent] = await Promise.all([
    generator.generateWithClaude({
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData: research,
    }),
    generator.generateWithClaude({
      format: 'toplist',
      tone: 'expert',
      language: 'vi',
      researchData: research,
    }),
  ]);

  return { en: englishContent, vi: vietnameseContent };
}
```

### Batch Video Generation

```typescript
async function generateMultipleVideos(contents: string[]) {
  const renderer = new VideoRenderer();
  const aspectRatios = ['9:16', '16:9', '1:1'] as const;

  const videos = await Promise.all(
    contents.flatMap((content) =>
      aspectRatios.map((ratio) =>
        renderer.renderVideo({
          content,
          title: 'Marketing Content',
          aspectRatio: ratio,
          duration: 60,
          template: 'reels',
        })
      )
    )
  );

  return videos;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      if (error.status === 429) {
        await new Promise((resolve) => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

For large video renders, use chunked processing:

```typescript
// Reduce memory usage with streaming
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(config: VideoConfig) {
  const frames = await renderFrames({
    composition,
    serveUrl: bundleLocation,
    onFrameUpdate: (frame) => {
      console.log(`Rendering frame ${frame}`);
    },
    parallelism: 2, // Reduce parallelism for lower memory
  });
}
```

### Character Encoding for Vietnamese

Ensure proper UTF-8 encoding:

```typescript
// src/lib/utils/encoding.ts
export function ensureUtf8(text: string): string {
  return Buffer.from(text, 'utf-8').toString('utf-8');
}

// Use in content generation
const content = ensureUtf8(await generator.generateWithClaude(config));
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npm run remotion:render

# Preview Remotion compositions
npm run remotion:preview
```

## Best Practices

1. **Cache research results** to avoid redundant API calls
2. **Use streaming responses** for long-form content generation
3. **Implement queue system** for video rendering to handle multiple requests
4. **Store generated videos** in cloud storage (S3, Cloudinary) instead of local filesystem
5. **Monitor AI token usage** to optimize costs
6. **Validate content** before video generation to ensure quality
