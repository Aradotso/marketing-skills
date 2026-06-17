---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, scriptwriting, social posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate automated social media content and videos
  - create content workflow with AI research and video rendering
  - build automated marketing content system
  - implement AI-powered content generation pipeline
  - configure Remotion video generation for content
  - set up automated content research and publishing
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Master the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates content creation from research scanning to video generation. This pipeline crawls news sources, generates multi-format content with AI (Claude/OpenAI), and renders videos automatically using Remotion.

## What It Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-scans research sources** - Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data (last 24h)
- **Generates diverse content formats** - Creates toplists, POV articles, case studies, how-tos in multiple languages
- **Renders videos automatically** - Converts text content into infographics and short-form videos via Remotion
- **Multi-platform optimization** - Exports content optimized for Reels, TikTok, Shorts
- **Integrated workflow** - Next.js interface for managing the entire pipeline

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

## Configuration

Create a `.env` file with the following variables:

```env
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

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Core Architecture

The pipeline follows this workflow:

1. **Research Phase** - Data collection from sources
2. **Content Generation** - AI-powered writing with Claude/OpenAI
3. **Video Rendering** - Remotion video generation
4. **Publishing** - Distribution to platforms

## Key Components

### 1. Research Crawler

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  name: string;
  url: string;
  selector?: string;
}

export class ContentCrawler {
  private sources: ResearchSource[];
  
  constructor(sources: ResearchSource[]) {
    this.sources = sources;
  }
  
  async crawlAll(keyword: string): Promise<ArticleData[]> {
    const results = await Promise.all(
      this.sources.map(source => this.crawlSource(source, keyword))
    );
    
    return results.flat();
  }
  
  async crawlSource(source: ResearchSource, keyword: string): Promise<ArticleData[]> {
    try {
      const response = await axios.get(source.url, {
        params: { q: keyword, timeRange: '24h' },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'X-RapidAPI-Host': source.name
        }
      });
      
      return this.parseArticles(response.data);
    } catch (error) {
      console.error(`Error crawling ${source.name}:`, error);
      return [];
    }
  }
  
  private parseArticles(data: any): ArticleData[] {
    // Parse and extract relevant data
    return data.articles?.map((article: any) => ({
      title: article.title,
      url: article.url,
      publishedAt: article.publishedAt,
      source: article.source,
      content: article.description
    })) || [];
  }
}

interface ArticleData {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  keywords: string[];
}

export class AIContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }
  
  async generateWithClaude(
    researchData: ArticleData[],
    config: ContentConfig
  ): Promise<string> {
    const prompt = this.buildPrompt(researchData, config);
    
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
  
  async generateWithOpenAI(
    researchData: ArticleData[],
    config: ContentConfig
  ): Promise<string> {
    const prompt = this.buildPrompt(researchData, config);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'system',
        content: 'You are an expert content writer specializing in marketing and technology.'
      }, {
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
      max_tokens: 4096
    });
    
    return completion.choices[0]?.message?.content || '';
  }
  
  private buildPrompt(data: ArticleData[], config: ContentConfig): string {
    const formatInstructions = this.getFormatInstructions(config.format);
    const toneInstructions = this.getToneInstructions(config.tone);
    const languageInstruction = config.language === 'vi' 
      ? 'Write in Vietnamese.' 
      : 'Write in English.';
    
    return `
Based on the following recent research data, create a ${config.format} article.

Research Data:
${data.map(article => `
Title: ${article.title}
Source: ${article.source}
Published: ${article.publishedAt}
Content: ${article.content}
URL: ${article.url}
`).join('\n---\n')}

Keywords to include: ${config.keywords.join(', ')}

${formatInstructions}
${toneInstructions}
${languageInstruction}

Make sure to:
- Include data-backed insights
- Reference specific sources
- Stay current and trend-focused
- Optimize for engagement
`;
  }
  
  private getFormatInstructions(format: ContentFormat): string {
    const formats = {
      'toplist': 'Create a numbered list format with clear headings, explanations, and examples for each point.',
      'pov': 'Write from a personal perspective, sharing unique insights and opinions backed by the research.',
      'case-study': 'Structure as a case study with Problem, Solution, Results, and Key Takeaways sections.',
      'how-to': 'Create a step-by-step guide with actionable instructions and clear outcomes.'
    };
    
    return formats[format];
  }
  
  private getToneInstructions(tone: Tone): string {
    const tones = {
      'expert': 'Use professional, authoritative language with industry terminology.',
      'friendly': 'Write in a conversational, approachable style that\'s easy to understand.',
      'humorous': 'Inject wit and humor while maintaining credibility.'
    };
    
    return tones[tone];
  }
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpack } from '@remotion/bundler';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts';
}

export class RemotionVideoRenderer {
  private bundleLocation: string | null = null;
  
  async initialize() {
    const bundled = await bundle(
      path.join(process.cwd(), 'remotion/index.ts'),
      () => undefined,
      {
        webpackOverride: (config) => config,
      }
    );
    
    this.bundleLocation = bundled;
  }
  
  async renderVideo(config: VideoConfig): Promise<string> {
    if (!this.bundleLocation) {
      await this.initialize();
    }
    
    const dimensions = this.getDimensions(config.format);
    const compositionId = 'ContentVideo';
    
    const composition = await selectComposition({
      serveUrl: this.bundleLocation!,
      id: compositionId,
      inputProps: {
        title: config.title,
        content: config.content,
        duration: config.duration
      },
    });
    
    const outputLocation = path.join(
      process.cwd(),
      'public',
      'videos',
      `${Date.now()}-${config.format}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: this.bundleLocation!,
      codec: 'h264',
      outputLocation,
      inputProps: {
        title: config.title,
        content: config.content,
      },
      ...dimensions
    });
    
    return outputLocation;
  }
  
  private getDimensions(format: 'reels' | 'tiktok' | 'shorts') {
    const dimensions = {
      'reels': { width: 1080, height: 1920 },
      'tiktok': { width: 1080, height: 1920 },
      'shorts': { width: 1080, height: 1920 }
    };
    
    return dimensions[format];
  }
}
```

### 4. Remotion Composition

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  const titleScale = interpolate(
    frame,
    [0, 30],
    [0.8, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <div
        style={{
          display: 'flex',
          flexDirection: 'column',
          justifyContent: 'center',
          alignItems: 'center',
          height: '100%',
          padding: '60px',
        }}
      >
        <h1
          style={{
            fontSize: '72px',
            fontWeight: 'bold',
            color: '#fff',
            textAlign: 'center',
            opacity: titleOpacity,
            transform: `scale(${titleScale})`,
            marginBottom: '40px',
          }}
        >
          {title}
        </h1>
        
        {content.map((point, index) => {
          const startFrame = 60 + (index * 90);
          const opacity = interpolate(
            frame,
            [startFrame, startFrame + 20],
            [0, 1],
            { extrapolateRight: 'clamp', extrapolateLeft: 'clamp' }
          );
          
          return (
            <div
              key={index}
              style={{
                fontSize: '36px',
                color: '#eee',
                marginBottom: '30px',
                opacity,
                maxWidth: '900px',
              }}
            >
              {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { ContentCrawler } from '../research/crawler';
import { AIContentGenerator } from '../ai/content-generator';
import { RemotionVideoRenderer } from '../video/remotion-renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: ContentFormat;
  language: Language;
  tone: Tone;
  includeVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export class ContentPipelineOrchestrator {
  private crawler: ContentCrawler;
  private aiGenerator: AIContentGenerator;
  private videoRenderer: RemotionVideoRenderer;
  
  constructor() {
    this.crawler = new ContentCrawler([
      { name: 'techcrunch', url: 'https://techcrunch.com/api/search' },
      { name: 'a16z', url: 'https://a16z.com/api/posts' },
    ]);
    
    this.aiGenerator = new AIContentGenerator();
    this.videoRenderer = new RemotionVideoRenderer();
  }
  
  async runPipeline(config: PipelineConfig) {
    console.log('🔍 Step 1: Crawling research data...');
    const researchData = await this.crawler.crawlAll(config.keyword);
    
    if (researchData.length === 0) {
      throw new Error('No research data found');
    }
    
    console.log(`✅ Found ${researchData.length} articles`);
    
    console.log('🤖 Step 2: Generating content with AI...');
    const content = await this.aiGenerator.generateWithClaude(researchData, {
      format: config.contentFormat,
      language: config.language,
      tone: config.tone,
      keywords: [config.keyword]
    });
    
    console.log('✅ Content generated');
    
    let videoPath: string | null = null;
    
    if (config.includeVideo && config.videoFormat) {
      console.log('🎬 Step 3: Rendering video...');
      
      const contentPoints = this.extractKeyPoints(content);
      
      videoPath = await this.videoRenderer.renderVideo({
        title: config.keyword,
        content: contentPoints,
        duration: 30,
        format: config.videoFormat
      });
      
      console.log('✅ Video rendered:', videoPath);
    }
    
    return {
      researchData,
      content,
      videoPath,
      metadata: {
        keyword: config.keyword,
        format: config.contentFormat,
        language: config.language,
        createdAt: new Date().toISOString()
      }
    };
  }
  
  private extractKeyPoints(content: string): string[] {
    // Extract main points from content for video
    const lines = content.split('\n').filter(line => line.trim());
    const points = lines
      .filter(line => line.match(/^[\d\-\*]/) || line.length > 50)
      .slice(0, 5);
    
    return points.map(point => point.replace(/^[\d\-\*\.]\s*/, '').trim());
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
// pages/api/generate-content.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipelineOrchestrator } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, language, includeVideo } = req.body;
  
  try {
    const orchestrator = new ContentPipelineOrchestrator();
    
    const result = await orchestrator.runPipeline({
      keyword,
      contentFormat: format || 'toplist',
      language: language || 'en',
      tone: 'expert',
      includeVideo: includeVideo || false,
      videoFormat: includeVideo ? 'reels' : undefined
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Failed to generate content',
      message: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

### Client-Side Integration

```typescript
// components/ContentGenerator.tsx
import React, { useState } from 'react';

export const ContentGenerator: React.FC = () => {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate-content', {
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
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full mb-4"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-6">
          <h3 className="text-xl font-bold mb-2">Generated Content:</h3>
          <div className="bg-gray-100 p-4 rounded whitespace-pre-wrap">
            {result.content}
          </div>
          
          {result.videoPath && (
            <div className="mt-4">
              <h4 className="font-bold mb-2">Video:</h4>
              <video src={result.videoPath} controls className="w-full max-w-md" />
            </div>
          )}
        </div>
      )}
    </div>
  );
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const orchestrator = new ContentPipelineOrchestrator();
  
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      orchestrator.runPipeline({
        keyword,
        contentFormat: 'toplist',
        language: 'en',
        tone: 'expert',
        includeVideo: false
      })
    )
  );
  
  return results
    .filter((r): r is PromiseFulfilledResult<any> => r.status === 'fulfilled')
    .map(r => r.value);
}
```

### Scheduled Content Generation

```typescript
// lib/scheduler/content-scheduler.ts
import cron from 'node-cron';
import { ContentPipelineOrchestrator } from '../pipeline/orchestrator';

export class ContentScheduler {
  private orchestrator: ContentPipelineOrchestrator;
  
  constructor() {
    this.orchestrator = new ContentPipelineOrchestrator();
  }
  
  startDailyGeneration(keywords: string[]) {
    // Run every day at 9 AM
    cron.schedule('0 9 * * *', async () => {
      console.log('Running scheduled content generation...');
      
      for (const keyword of keywords) {
        try {
          await this.orchestrator.runPipeline({
            keyword,
            contentFormat: 'toplist',
            language: 'en',
            tone: 'expert',
            includeVideo: true,
            videoFormat: 'reels'
          });
        } catch (error) {
          console.error(`Failed to generate for ${keyword}:`, error);
        }
      }
    });
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
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

// Usage in AI generator
const rateLimiter = new RateLimiter(10); // 10 calls per minute

async generateWithRateLimit(data: ArticleData[], config: ContentConfig) {
  await rateLimiter.throttle();
  return this.generateWithClaude(data, config);
}
```

### Error Handling

```typescript
class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'video',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

// In orchestrator
try {
  const researchData = await this.crawler.crawlAll(config.keyword);
} catch (error) {
  throw new PipelineError(
    'Failed to crawl research data',
    'research',
    error as Error
  );
}
```

### Video Rendering Issues

If video rendering fails:

1. **Check Remotion installation**:
```bash
npm install @remotion/bundler @remotion/renderer @remotion/cli
```

2. **Verify FFmpeg**:
```bash
# Install FFmpeg if missing
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg
```

3. **Memory issues** - Increase Node.js memory:
```bash
NODE_OPTIONS=--max-old-space-size=4096 npm run dev
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Remotion preview (for video development)
npm run remotion:preview
```

Access the application at `http://localhost:3000`
