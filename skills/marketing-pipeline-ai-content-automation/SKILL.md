---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - "help me automate content creation with AI"
  - "set up the marketing pipeline content automation"
  - "generate AI content from research to video"
  - "create automated content with Claude and OpenAI"
  - "build an AI content generation pipeline"
  - "automate blog posts and video content"
  - "use the ultimate AI content pipeline"
  - "set up AI-powered content research and generation"
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates content creation from research gathering, script writing, to video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to create a complete content production workflow.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter, LinkedIn
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) in multiple languages using Claude/OpenAI
3. **Video Rendering**: Automatically generates infographics and short videos using Remotion
4. **Multi-Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

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

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Research/crawling modules
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API & Usage

### 1. Research Module - Auto-Crawl Content

```typescript
// src/lib/research/crawler.ts
import { ResearchCrawler } from '@/lib/research/crawler';

interface CrawlConfig {
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  keywords: string[];
  timeRange: '24h' | '7d' | '30d';
  maxResults: number;
}

async function gatherResearch(topic: string): Promise<ResearchData[]> {
  const crawler = new ResearchCrawler({
    apiKey: process.env.RAPIDAPI_KEY!
  });

  const config: CrawlConfig = {
    sources: ['techcrunch', 'twitter', 'linkedin'],
    keywords: [topic],
    timeRange: '24h',
    maxResults: 20
  };

  const results = await crawler.crawl(config);
  
  // Filter and rank by relevance
  const insights = await crawler.extractInsights(results);
  
  return insights;
}

// Usage
const data = await gatherResearch('AI automation');
console.log(`Found ${data.length} relevant articles`);
```

### 2. AI Content Generation with Claude/OpenAI

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchData[];
}

class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!
    });
  }

  async generateWithClaude(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const message = await this.claude.messages.create({
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

  async generateWithOpenAI(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'system',
        content: 'You are an expert content writer and marketer.'
      }, {
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
      max_tokens: 4096
    });

    return completion.choices[0].message.content || '';
  }

  private buildPrompt(config: ContentConfig): string {
    const { format, language, tone, researchData } = config;
    
    const researchContext = researchData
      .map(r => `- ${r.title}: ${r.summary}`)
      .join('\n');

    return `
Create a ${format} article in ${language} with a ${tone} tone.

Research Data:
${researchContext}

Requirements:
- Use data-backed insights from the research
- Include specific examples and statistics
- Structure for readability
- Optimize for SEO
- Target audience: marketers and content creators
    `.trim();
  }
}

// Usage
const generator = new ContentGenerator();
const content = await generator.generateWithClaude({
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: await gatherResearch('AI marketing')
});
```

### 3. Remotion Video Generation

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'vertical' | 'square' | 'horizontal';
  duration: number; // in seconds
}

class VideoRenderer {
  async renderVideo(config: VideoConfig): Promise<string> {
    const { content, title, format, duration } = config;
    
    // Bundle Remotion composition
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config
    });

    // Get composition dimensions
    const dimensions = this.getDimensions(format);
    
    // Select composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: 'ContentVideo',
      inputProps: {
        title,
        content: this.parseContentForVideo(content),
        ...dimensions
      }
    });

    // Render video
    const outputPath = path.join(
      process.cwd(), 
      'public/videos',
      `${Date.now()}-${format}.mp4`
    );

    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        title,
        content: this.parseContentForVideo(content),
        duration: duration * 30 // fps
      }
    });

    return outputPath;
  }

  private getDimensions(format: VideoConfig['format']) {
    switch (format) {
      case 'vertical': // TikTok, Reels, Shorts
        return { width: 1080, height: 1920 };
      case 'square': // Instagram feed
        return { width: 1080, height: 1080 };
      case 'horizontal': // YouTube
        return { width: 1920, height: 1080 };
    }
  }

  private parseContentForVideo(content: string) {
    // Extract key points for video slides
    const lines = content.split('\n').filter(l => l.trim());
    const keyPoints = lines
      .filter(l => l.startsWith('- ') || l.startsWith('## '))
      .slice(0, 5)
      .map(l => l.replace(/^[-#\s]+/, ''));
    
    return keyPoints;
  }
}

// Usage
const renderer = new VideoRenderer();
const videoPath = await renderer.renderVideo({
  content: generatedContent,
  title: 'Top 5 AI Marketing Tools',
  format: 'vertical',
  duration: 30
});
```

### 4. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/content-pipeline.ts
export class ContentPipeline {
  private crawler: ResearchCrawler;
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.crawler = new ResearchCrawler({
      apiKey: process.env.RAPIDAPI_KEY!
    });
    this.generator = new ContentGenerator();
    this.renderer = new VideoRenderer();
  }

  async execute(topic: string, config: PipelineConfig) {
    console.log(`🔍 Starting pipeline for topic: ${topic}`);

    // Step 1: Research
    console.log('📡 Gathering research...');
    const research = await this.crawler.crawl({
      sources: ['techcrunch', 'twitter', 'linkedin'],
      keywords: [topic],
      timeRange: '24h',
      maxResults: 20
    });

    const insights = await this.crawler.extractInsights(research);
    console.log(`✅ Found ${insights.length} insights`);

    // Step 2: Generate Content
    console.log('🧠 Generating content...');
    const content = await this.generator.generateWithClaude({
      format: config.contentFormat,
      language: config.language,
      tone: config.tone,
      researchData: insights
    });
    console.log('✅ Content generated');

    // Step 3: Create Video (optional)
    let videoPath: string | null = null;
    if (config.generateVideo) {
      console.log('🎬 Rendering video...');
      videoPath = await this.renderer.renderVideo({
        content,
        title: config.title,
        format: config.videoFormat,
        duration: 30
      });
      console.log(`✅ Video rendered: ${videoPath}`);
    }

    return {
      content,
      videoPath,
      metadata: {
        topic,
        researchCount: insights.length,
        generatedAt: new Date().toISOString()
      }
    };
  }
}

// API Route Example: src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(req: NextRequest) {
  try {
    const { topic, config } = await req.json();
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(topic, config);
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

### React Component Example

```typescript
// src/components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export function ContentPipelineForm() {
  const [topic, setTopic] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          topic,
          config: {
            contentFormat: 'toplist',
            language: 'en',
            tone: 'expert',
            generateVideo: true,
            videoFormat: 'vertical',
            title: `Top AI Insights: ${topic}`
          }
        })
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
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        type="text"
        value={topic}
        onChange={(e) => setTopic(e.target.value)}
        placeholder="Enter topic (e.g., AI Marketing)"
        className="w-full px-4 py-2 border rounded"
        required
      />
      
      <button
        type="submit"
        disabled={loading}
        className="px-6 py-2 bg-blue-600 text-white rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-4">
          <h3 className="font-bold">Generated Content:</h3>
          <div className="prose">{result.content}</div>
          {result.videoPath && (
            <video src={result.videoPath} controls />
          )}
        </div>
      )}
    </form>
  );
}
```

## Configuration Patterns

### Custom AI Prompts

```typescript
// src/config/prompts.ts
export const CONTENT_PROMPTS = {
  toplist: {
    system: 'You are an expert at creating engaging top 10/top 5 lists.',
    template: (topic: string, data: string) => `
      Create a compelling top list about ${topic}.
      Use this research: ${data}
      Include numbers, statistics, and real examples.
    `
  },
  
  caseStudy: {
    system: 'You are a business analyst writing case studies.',
    template: (topic: string, data: string) => `
      Write a detailed case study about ${topic}.
      Research data: ${data}
      Include problem, solution, and measurable results.
    `
  }
};
```

### Multi-Language Support

```typescript
// src/lib/ai/translator.ts
export async function generateBilingual(
  topic: string,
  research: ResearchData[]
): Promise<{ en: string; vi: string }> {
  const generator = new ContentGenerator();
  
  const [english, vietnamese] = await Promise.all([
    generator.generateWithClaude({
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research
    }),
    generator.generateWithClaude({
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      researchData: research
    })
  ]);

  return { en: english, vi: vietnamese };
}
```

## Troubleshooting

### API Key Issues

**Problem**: `Error: API key not found`

**Solution**: Ensure all required environment variables are set:

```bash
# Check if keys are loaded
console.log('Keys check:', {
  openai: !!process.env.OPENAI_API_KEY,
  anthropic: !!process.env.ANTHROPIC_API_KEY,
  rapidapi: !!process.env.RAPIDAPI_KEY
});
```

### Rate Limiting

**Problem**: Too many API requests

**Solution**: Implement request queuing:

```typescript
// src/lib/utils/rate-limiter.ts
import pQueue from 'p-queue';

const queue = new pQueue({ 
  concurrency: 2,
  interval: 1000,
  intervalCap: 2
});

export async function queuedRequest<T>(
  fn: () => Promise<T>
): Promise<T> {
  return queue.add(fn);
}

// Usage
const content = await queuedRequest(() => 
  generator.generateWithClaude(config)
);
```

### Remotion Rendering Fails

**Problem**: Video rendering times out or fails

**Solution**: Increase timeout and use local rendering:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    headless: true,
    gl: 'swiftshader'
  }
});
```

### Memory Issues with Large Content

**Problem**: Node.js heap out of memory

**Solution**: Increase Node memory limit:

```bash
# In package.json
"scripts": {
  "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
  "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
}
```

## Best Practices

1. **Cache Research Data**: Store crawled data to avoid redundant API calls
2. **Batch Processing**: Generate multiple content pieces in parallel
3. **Error Handling**: Wrap AI calls with try-catch and fallback providers
4. **Cost Optimization**: Use GPT-3.5 for drafts, GPT-4/Claude for final content
5. **Video Optimization**: Pre-render templates, only update dynamic content

This skill enables comprehensive automation of content marketing workflows from research to publication-ready assets.
