---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, script generation, video creation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate AI-driven marketing content pipeline
  - create automated video content with Remotion
  - build AI content workflow with Claude and OpenAI
  - set up automated content research and publishing
  - generate multilingual marketing content automatically
  - create content automation pipeline with AI
  - automate social media content generation and video rendering
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**marketing-pipeline-share** is an end-to-end AI-powered content automation system that handles the complete content creation workflow: from automated research and data scraping, to script generation in multiple formats and languages, to automatic video rendering and publishing. Built with TypeScript, Next.js, and integrating Claude 3, OpenAI, and Remotion for video generation.

The system crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn to generate fresh, data-backed content in various formats (toplist, POV, case study, how-to) in both English and Vietnamese.

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
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for research/scraping
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if using)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
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
│   │   ├── scraper/     # Research & data crawling
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Functionality

### 1. Automated Research & Data Scraping

```typescript
// lib/scraper/research.ts
import { RapidAPIClient } from './rapidapi-client';

interface ResearchQuery {
  keyword: string;
  sources?: string[];
  timeframe?: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

export async function conductResearch(query: ResearchQuery) {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const sources = query.sources || [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const results = await Promise.all(
    sources.map(source => 
      client.fetchLatestNews({
        source,
        keyword: query.keyword,
        timeframe: query.timeframe || '24h'
      })
    )
  );
  
  return {
    keyword: query.keyword,
    sources: results.flat(),
    insights: await extractInsights(results),
    timestamp: new Date()
  };
}

async function extractInsights(data: any[]) {
  // Process and extract key insights from scraped data
  const insights = data.map(item => ({
    title: item.title,
    summary: item.summary,
    url: item.url,
    sentiment: analyzeSentiment(item.content),
    keyPoints: extractKeyPoints(item.content)
  }));
  
  return insights;
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  research: any;
}

export class ContentGenerator {
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
  
  async generateContent(config: ContentConfig) {
    const systemPrompt = this.buildSystemPrompt(config);
    const userPrompt = this.buildUserPrompt(config);
    
    // Use Claude for content generation
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: userPrompt
      }],
      system: systemPrompt
    });
    
    const content = message.content[0].text;
    
    return {
      content,
      metadata: {
        format: config.format,
        language: config.language,
        tone: config.tone,
        generatedAt: new Date()
      }
    };
  }
  
  private buildSystemPrompt(config: ContentConfig): string {
    const toneMap = {
      professional: 'expert and authoritative',
      friendly: 'conversational and approachable',
      humorous: 'witty and entertaining'
    };
    
    return `You are an expert content creator specializing in ${config.format} articles.
Write in a ${toneMap[config.tone]} tone in ${config.language === 'en' ? 'English' : 'Vietnamese'}.
Use the provided research data to create data-backed, insightful content.`;
  }
  
  private buildUserPrompt(config: ContentConfig): string {
    return `Create a ${config.format} article about "${config.keyword}" using this research:

${JSON.stringify(config.research, null, 2)}

Requirements:
- Format: ${config.format}
- Include specific data points and statistics
- Add actionable insights
- Structure with clear headings
- Include meta description and SEO title`;
  }
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reel' | 'tiktok' | 'youtube-short';
  language: 'en' | 'vi';
}

export class VideoGenerator {
  async generateVideo(config: VideoConfig) {
    const compositionId = this.getCompositionId(config.format);
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config
    });
    
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId
    });
    
    const outputLocation = path.join(
      process.cwd(),
      'public',
      'videos',
      `${Date.now()}-${config.format}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps: {
        title: config.title,
        content: this.parseContentForVideo(config.content),
        language: config.language
      }
    });
    
    return {
      videoPath: outputLocation,
      format: config.format,
      duration: composition.durationInFrames / composition.fps
    };
  }
  
  private getCompositionId(format: string): string {
    const formatMap = {
      'reel': 'InstagramReel',
      'tiktok': 'TikTokVideo',
      'youtube-short': 'YouTubeShort'
    };
    return formatMap[format] || 'DefaultComposition';
  }
  
  private parseContentForVideo(content: string) {
    // Extract key points and create video scenes
    const sections = content.split('\n\n');
    return sections.map(section => ({
      text: section.trim(),
      duration: 3 // seconds per scene
    }));
  }
}
```

### 4. Complete Pipeline Integration

```typescript
// lib/pipeline/content-pipeline.ts
import { conductResearch } from '../scraper/research';
import { ContentGenerator } from '../ai/content-generator';
import { VideoGenerator } from '../video/video-generator';

interface PipelineConfig {
  keyword: string;
  formats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'>;
  languages: Array<'en' | 'vi'>;
  generateVideo: boolean;
  videoFormats?: Array<'reel' | 'tiktok' | 'youtube-short'>;
}

export class ContentPipeline {
  private contentGenerator: ContentGenerator;
  private videoGenerator: VideoGenerator;
  
  constructor() {
    this.contentGenerator = new ContentGenerator();
    this.videoGenerator = new VideoGenerator();
  }
  
  async execute(config: PipelineConfig) {
    console.log(`🔍 Starting research for: ${config.keyword}`);
    
    // Step 1: Research
    const research = await conductResearch({
      keyword: config.keyword,
      timeframe: '24h'
    });
    
    console.log(`✅ Research complete: ${research.sources.length} sources`);
    
    // Step 2: Generate content in all formats and languages
    const contents = [];
    for (const format of config.formats) {
      for (const language of config.languages) {
        console.log(`📝 Generating ${format} in ${language}...`);
        
        const content = await this.contentGenerator.generateContent({
          keyword: config.keyword,
          format,
          language,
          tone: 'professional',
          research
        });
        
        contents.push({
          ...content,
          format,
          language
        });
      }
    }
    
    console.log(`✅ Generated ${contents.length} content pieces`);
    
    // Step 3: Generate videos (if enabled)
    const videos = [];
    if (config.generateVideo && config.videoFormats) {
      for (const content of contents) {
        for (const videoFormat of config.videoFormats) {
          console.log(`🎬 Rendering ${videoFormat} video...`);
          
          const video = await this.videoGenerator.generateVideo({
            content: content.content,
            title: config.keyword,
            format: videoFormat,
            language: content.language
          });
          
          videos.push(video);
        }
      }
      
      console.log(`✅ Rendered ${videos.length} videos`);
    }
    
    return {
      research,
      contents,
      videos,
      summary: {
        keyword: config.keyword,
        totalContent: contents.length,
        totalVideos: videos.length,
        completedAt: new Date()
      }
    };
  }
}
```

## Usage Examples

### API Route Example

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: Request) {
  try {
    const body = await request.json();
    const { keyword, formats, languages, generateVideo } = body;
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute({
      keyword,
      formats: formats || ['toplist', 'how-to'],
      languages: languages || ['en', 'vi'],
      generateVideo: generateVideo || false,
      videoFormats: generateVideo ? ['reel', 'tiktok'] : undefined
    });
    
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

### React Component Example

```typescript
// components/ContentGenerator.tsx
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
          formats: ['toplist', 'pov'],
          languages: ['en', 'vi'],
          generateVideo: true
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
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="mt-4 bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-6">
          <h3 className="font-bold">Results:</h3>
          <pre className="bg-gray-100 p-4 rounded mt-2 overflow-auto">
            {JSON.stringify(result.summary, null, 2)}
          </pre>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.all(
    keywords.map(keyword => 
      pipeline.execute({
        keyword,
        formats: ['toplist'],
        languages: ['en'],
        generateVideo: false
      })
    )
  );
  
  return results;
}
```

### Scheduled Content Creation

```typescript
// Using cron or scheduled job
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function scheduledContentJob() {
  const trendingKeywords = await fetchTrendingKeywords();
  const pipeline = new ContentPipeline();
  
  for (const keyword of trendingKeywords) {
    await pipeline.execute({
      keyword,
      formats: ['pov', 'how-to'],
      languages: ['en', 'vi'],
      generateVideo: true,
      videoFormats: ['reel', 'tiktok', 'youtube-short']
    });
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting and retry logic
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Issues

- Ensure sufficient memory for Remotion rendering
- Check that FFmpeg is installed on the system
- Use lower resolution for faster rendering during development

### Content Quality

- Adjust AI prompts in `buildSystemPrompt()` for better output
- Increase `max_tokens` for longer content
- Use different AI models for different content types (Claude for creative, GPT-4 for technical)

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npm run remotion render
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Queue video rendering** for resource-intensive operations
3. **Validate input** before passing to AI models
4. **Store generated content** in a database for tracking and reuse
5. **Monitor API usage** to manage costs effectively
