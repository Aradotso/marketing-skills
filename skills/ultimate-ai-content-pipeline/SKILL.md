---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline with AI research, script generation, and video rendering using Claude, OpenAI, and Remotion
triggers:
  - "set up content automation pipeline"
  - "generate AI content with research"
  - "create video from text automatically"
  - "automate content creation workflow"
  - "scrape news and generate articles"
  - "render video with remotion integration"
  - "build marketing content pipeline"
  - "generate multilingual content with AI"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline crawls news sources, generates articles in multiple formats and languages using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** recent content from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates articles** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Produces bilingual content** (English and Vietnamese) with customizable tone
- **Renders videos** and infographics automatically using Remotion
- **Optimizes output** for multiple platforms (Reels, TikTok, Shorts)

The system is built with Next.js and integrates with OpenAI, Anthropic Claude, RapidAPI, and Remotion.

## Installation

### Prerequisites

```bash
# Ensure you have Node.js 18+ and pnpm installed
node --version  # Should be 18+
pnpm --version  # Install via: npm install -g pnpm
```

### Clone and Install

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Config
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
pnpm dev
# Server runs on http://localhost:3000
```

### Build for Production

```bash
pnpm build
pnpm start
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # Content scraping modules
│   │   ├── generator/   # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Modules

### 1. Content Crawler

The crawler module fetches recent content from various sources:

```typescript
// src/lib/crawler/news-scraper.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  selector?: string;
}

export class NewsScraper {
  private sources: NewsSource[];
  
  constructor(sources: NewsSource[]) {
    this.sources = sources;
  }
  
  async scrapeAll(keyword: string, hours: number = 24): Promise<Article[]> {
    const articles: Article[] = [];
    
    for (const source of this.sources) {
      try {
        const data = await this.scrapeSource(source, keyword, hours);
        articles.push(...data);
      } catch (error) {
        console.error(`Failed to scrape ${source.name}:`, error);
      }
    }
    
    return articles;
  }
  
  private async scrapeSource(
    source: NewsSource, 
    keyword: string, 
    hours: number
  ): Promise<Article[]> {
    const response = await axios.get(source.url, {
      params: { q: keyword, hours },
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        'X-RapidAPI-Host': 'news-api.com'
      }
    });
    
    return response.data.articles.map((article: any) => ({
      title: article.title,
      url: article.url,
      source: source.name,
      publishedAt: article.publishedAt,
      content: article.content,
      summary: article.description
    }));
  }
}

// Usage
const scraper = new NewsScraper([
  { name: 'TechCrunch', url: 'https://api.techcrunch.com/search' },
  { name: 'a16z', url: 'https://api.a16z.com/posts' }
]);

const articles = await scraper.scrapeAll('AI automation', 24);
```

### 2. AI Content Generator

Generate content using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerationConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  keyword: string;
  researchData: Article[];
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateWithClaude(config: GenerationConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const message = await this.anthropic.messages.create({
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
  
  async generateWithOpenAI(config: GenerationConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content || '';
  }
  
  private buildPrompt(config: GenerationConfig): string {
    const { format, language, tone, keyword, researchData } = config;
    
    const researchSummary = researchData
      .map(article => `- ${article.title}: ${article.summary}`)
      .join('\n');
    
    return `
You are a content creator specializing in ${format} articles.

Target keyword: ${keyword}
Language: ${language === 'en' ? 'English' : 'Vietnamese'}
Tone: ${tone}
Format: ${format}

Research data from the last 24 hours:
${researchSummary}

Create a comprehensive ${format} article about "${keyword}" that:
1. Uses insights from the research data above
2. Includes relevant statistics and examples
3. Maintains a ${tone} tone throughout
4. Is structured for ${language} readers
5. Is optimized for social media sharing

${format === 'toplist' ? 'Format as a numbered list with explanations for each item.' : ''}
${format === 'how-to' ? 'Provide step-by-step instructions with clear action items.' : ''}
${format === 'pov' ? 'Present a unique perspective or opinion backed by data.' : ''}
${format === 'case-study' ? 'Analyze a real example with lessons learned.' : ''}
`.trim();
  }
}

// Usage
const generator = new ContentGenerator();

const content = await generator.generateWithClaude({
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  keyword: 'AI Marketing Tools 2024',
  researchData: articles
});
```

### 3. Bilingual Content Generation

Generate content in multiple languages simultaneously:

```typescript
// src/lib/ai/bilingual-generator.ts
import { ContentGenerator } from './content-generator';

export class BilingualGenerator {
  private generator: ContentGenerator;
  
  constructor() {
    this.generator = new ContentGenerator();
  }
  
  async generateBoth(
    config: Omit<GenerationConfig, 'language'>
  ): Promise<{ en: string; vi: string }> {
    const [enContent, viContent] = await Promise.all([
      this.generator.generateWithClaude({ ...config, language: 'en' }),
      this.generator.generateWithClaude({ ...config, language: 'vi' })
    ]);
    
    return { en: enContent, vi: viContent };
  }
}

// Usage
const bilingualGen = new BilingualGenerator();

const { en, vi } = await bilingualGen.generateBoth({
  format: 'how-to',
  tone: 'friendly',
  keyword: 'Content Marketing Automation',
  researchData: articles
});
```

### 4. Video Rendering with Remotion

Create videos from generated content:

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  platform: 'reels' | 'tiktok' | 'youtube-shorts';
}

export class VideoRenderer {
  private getDimensions(platform: string) {
    const dimensions = {
      'reels': { width: 1080, height: 1920 },
      'tiktok': { width: 1080, height: 1920 },
      'youtube-shorts': { width: 1080, height: 1920 }
    };
    
    return dimensions[platform as keyof typeof dimensions] || dimensions.reels;
  }
  
  async renderVideo(config: VideoConfig): Promise<string> {
    const compositionId = 'ContentVideo';
    const { width, height } = this.getDimensions(config.platform);
    
    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config
    });
    
    // Select composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: {
        title: config.title,
        content: config.content
      }
    });
    
    // Render video
    const outputLocation = path.join(
      process.cwd(), 
      'public/videos',
      `${Date.now()}-${config.platform}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps: {
        title: config.title,
        content: config.content
      }
    });
    
    return outputLocation;
  }
}

// Usage
const renderer = new VideoRenderer();

const videoPath = await renderer.renderVideo({
  title: '5 AI Marketing Tools You Need',
  content: [
    'Tool 1: ChatGPT for content creation',
    'Tool 2: Midjourney for visuals',
    'Tool 3: Claude for analysis',
    'Tool 4: Jasper for copywriting',
    'Tool 5: Synthesia for videos'
  ],
  platform: 'reels'
});
```

### 5. Remotion Video Template

Define video composition:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate, spring } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity: titleOpacity
        }}>
          <h1 style={{ 
            color: 'white', 
            fontSize: 72, 
            textAlign: 'center',
            padding: '0 60px'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Content Items */}
      {content.map((item, index) => {
        const startFrame = 60 + (index * 90);
        
        return (
          <Sequence 
            key={index} 
            from={startFrame} 
            durationInFrames={90}
          >
            <ContentSlide text={item} index={index + 1} />
          </Sequence>
        );
      })}
    </AbsoluteFill>
  );
};

const ContentSlide: React.FC<{ text: string; index: number }> = ({ text, index }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const scale = spring({
    frame,
    fps,
    from: 0.8,
    to: 1,
    config: { damping: 100 }
  });
  
  return (
    <AbsoluteFill style={{
      justifyContent: 'center',
      alignItems: 'center',
      transform: `scale(${scale})`
    }}>
      <div style={{
        backgroundColor: '#2a2a2a',
        padding: 60,
        borderRadius: 20,
        maxWidth: '80%'
      }}>
        <div style={{ 
          color: '#00ff88', 
          fontSize: 96, 
          fontWeight: 'bold',
          marginBottom: 20
        }}>
          {index}
        </div>
        <p style={{ 
          color: 'white', 
          fontSize: 48,
          lineHeight: 1.4
        }}>
          {text}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

End-to-end content creation workflow:

```typescript
// src/lib/pipeline/content-pipeline.ts
import { NewsScraper } from '../crawler/news-scraper';
import { BilingualGenerator } from '../ai/bilingual-generator';
import { VideoRenderer } from '../video/video-renderer';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: Tone;
  generateVideo: boolean;
  platforms?: ('reels' | 'tiktok' | 'youtube-shorts')[];
}

export class ContentPipeline {
  private scraper: NewsScraper;
  private generator: BilingualGenerator;
  private videoRenderer: VideoRenderer;
  
  constructor() {
    this.scraper = new NewsScraper([
      { name: 'TechCrunch', url: 'https://api.techcrunch.com/search' },
      { name: 'a16z', url: 'https://api.a16z.com/posts' }
    ]);
    this.generator = new BilingualGenerator();
    this.videoRenderer = new VideoRenderer();
  }
  
  async execute(config: PipelineConfig) {
    console.log('🔍 Step 1: Researching content...');
    const articles = await this.scraper.scrapeAll(config.keyword, 24);
    
    console.log(`✅ Found ${articles.length} articles`);
    console.log('✍️ Step 2: Generating content...');
    
    const { en, vi } = await this.generator.generateBoth({
      format: config.format,
      tone: config.tone,
      keyword: config.keyword,
      researchData: articles
    });
    
    console.log('✅ Content generated in both languages');
    
    const result = {
      research: articles,
      content: { en, vi },
      videos: [] as string[]
    };
    
    if (config.generateVideo) {
      console.log('🎬 Step 3: Rendering videos...');
      
      const contentPoints = this.extractKeyPoints(en);
      const platforms = config.platforms || ['reels'];
      
      for (const platform of platforms) {
        const videoPath = await this.videoRenderer.renderVideo({
          title: config.keyword,
          content: contentPoints,
          platform
        });
        
        result.videos.push(videoPath);
        console.log(`✅ Video rendered for ${platform}: ${videoPath}`);
      }
    }
    
    console.log('🎉 Pipeline completed!');
    return result;
  }
  
  private extractKeyPoints(content: string): string[] {
    // Simple extraction - in production, use better NLP
    const lines = content.split('\n').filter(line => 
      line.match(/^\d+\./) || line.match(/^-/) || line.match(/^•/)
    );
    
    return lines.slice(0, 5).map(line => 
      line.replace(/^\d+\.\s*/, '').replace(/^[-•]\s*/, '').trim()
    );
  }
}

// Usage
const pipeline = new ContentPipeline();

const result = await pipeline.execute({
  keyword: 'AI Content Marketing 2024',
  format: 'toplist',
  tone: 'expert',
  generateVideo: true,
  platforms: ['reels', 'tiktok', 'youtube-shorts']
});

console.log('English content:', result.content.en);
console.log('Vietnamese content:', result.content.vi);
console.log('Videos:', result.videos);
```

## API Routes (Next.js)

Create API endpoints for the pipeline:

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, generateVideo, platforms } = body;
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute({
      keyword,
      format: format || 'toplist',
      tone: tone || 'expert',
      generateVideo: generateVideo || false,
      platforms: platforms || ['reels']
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

## Frontend Integration

React component for the UI:

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<ContentFormat>('toplist');
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
          format,
          tone: 'expert',
          generateVideo: true,
          platforms: ['reels', 'tiktok']
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
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">
        AI Content Generator
      </h1>
      
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
          onChange={(e) => setFormat(e.target.value as ContentFormat)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>
      
      {result && (
        <div className="mt-8 space-y-6">
          <div>
            <h2 className="text-xl font-bold mb-2">English Content</h2>
            <div className="bg-gray-50 p-4 rounded whitespace-pre-wrap">
              {result.content.en}
            </div>
          </div>
          
          <div>
            <h2 className="text-xl font-bold mb-2">Vietnamese Content</h2>
            <div className="bg-gray-50 p-4 rounded whitespace-pre-wrap">
              {result.content.vi}
            </div>
          </div>
          
          {result.videos.length > 0 && (
            <div>
              <h2 className="text-xl font-bold mb-2">Generated Videos</h2>
              {result.videos.map((path: string, i: number) => (
                <video key={i} controls className="w-full mb-2">
                  <source src={path} type="video/mp4" />
                </video>
              ))}
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Troubleshooting

### Common Issues

**API Key Errors**
```typescript
// Check if environment variables are loaded
if (!process.env.OPENAI_API_KEY) {
  throw new Error('OPENAI_API_KEY not found in environment');
}
```

**Rate Limiting**
```typescript
// Add rate limiting middleware
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10 // limit each IP to 10 requests per windowMs
});

// Apply to API routes
export const config = {
  api: {
    bodyParser: {
      sizeLimit: '1mb'
    }
  }
};
```

**Remotion Rendering Fails**
```bash
# Ensure ffmpeg is installed
brew install ffmpeg  # macOS
apt-get install ffmpeg  # Linux

# Check Remotion license
echo $REMOTION_LICENSE_KEY
```

**Memory Issues with Large Videos**
```typescript
// Increase Node memory limit
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev"
  }
}
```

### Debug Mode

Enable detailed logging:

```typescript
// src/lib/logger.ts
export const logger = {
  debug: (message: string, data?: any) => {
    if (process.env.DEBUG === 'true') {
      console.log(`[DEBUG] ${message}`, data);
    }
  },
  error: (message: string, error?: any) => {
    console.error(`[ERROR] ${message}`, error);
  }
};

// Use in pipeline
logger.debug('Starting content generation', { keyword, format });
```

## Best Practices

1. **Cache Research Data**: Store scraped articles to avoid redundant API calls
2. **Implement Queue System**: Use Bull or BullMQ for video rendering jobs
3. **Error Handling**: Wrap all async operations in try-catch blocks
4. **Type Safety**: Use TypeScript interfaces for all data structures
5. **Rate Limit AI APIs**: Implement exponential backoff for API calls
6. **Optimize Video Rendering**: Render videos in background workers
7. **Monitor Costs**: Track API usage for OpenAI and Anthropic

This skill enables comprehensive automation of content creation workflows, from research through publication-ready video assets.
