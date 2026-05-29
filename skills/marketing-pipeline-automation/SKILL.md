---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline with research crawling, multi-format content generation, and automated video rendering using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate video content from text automatically
  - crawl news and create marketing content
  - set up AI content pipeline with Remotion
  - build automated content workflow
  - create multi-language marketing content with AI
  - automate social media video generation
  - research to video content pipeline
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow from research and script writing to video generation and multi-platform publishing.

## What This Project Does

The Marketing Pipeline automates:

1. **Auto-Research Crawling**: Scrapes latest news from TechCrunch, a16z, Twitter, LinkedIn within the last 24 hours
2. **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
3. **Multi-language Support**: Generates parallel English and Vietnamese content with customizable tone
4. **Automated Video Rendering**: Converts text content to videos and infographics using Remotion
5. **Multi-platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

### Required Environment Variables

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   ├── components/        # React components
│   └── page.tsx           # Main page
├── lib/                   # Core libraries
│   ├── research/          # Research crawlers
│   ├── ai/               # AI content generation
│   └── video/            # Remotion video rendering
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Core Components

### 1. Research Crawler

```typescript
// lib/research/crawler.ts
import { NewsAPI } from '@/lib/api/news';

interface ResearchResult {
  title: string;
  source: string;
  url: string;
  publishedAt: string;
  content: string;
  insights: string[];
}

export async function crawlLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter', 'linkedin']
): Promise<ResearchResult[]> {
  const newsAPI = new NewsAPI(process.env.RAPIDAPI_KEY!);
  
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    try {
      const articles = await newsAPI.search({
        keyword,
        source,
        timeRange: '24h',
        limit: 10
      });
      
      results.push(...articles);
    } catch (error) {
      console.error(`Error crawling ${source}:`, error);
    }
  }
  
  return results;
}

// Usage in API route
export async function POST(req: Request) {
  const { keyword } = await req.json();
  const research = await crawlLatestNews(keyword);
  
  return Response.json({ research });
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchResult[];
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
  
  async generateContent(options: ContentOptions): Promise<string> {
    const { format, language, tone, research } = options;
    
    const researchContext = research
      .map(r => `${r.title}\n${r.content}\nInsights: ${r.insights.join(', ')}`)
      .join('\n\n');
    
    const prompt = this.buildPrompt(format, language, tone, researchContext);
    
    // Use Claude for content generation
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt
        }
      ]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  private buildPrompt(
    format: string, 
    language: string, 
    tone: string, 
    research: string
  ): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with rankings',
      'pov': 'Write from a unique point of view with personal insights',
      'case-study': 'Structure as a detailed case study with data',
      'how-to': 'Create step-by-step tutorial format'
    };
    
    return `
You are an expert content creator. Using the following research data, create a ${format} article.

Research Data:
${research}

Requirements:
- Format: ${formatInstructions[format as keyof typeof formatInstructions]}
- Language: ${language === 'en' ? 'English' : 'Vietnamese'}
- Tone: ${tone}
- Include specific data points and insights from the research
- Make it engaging and actionable
- Add relevant examples and statistics

Generate the complete article now:
    `.trim();
  }
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
  duration?: number;
}

const platformDimensions = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

export async function renderVideo(config: VideoConfig): Promise<string> {
  const { content, title, platform, duration = 30 } = config;
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      title
    }
  });
  
  const { width, height } = platformDimensions[platform];
  
  const outputPath = path.join(
    process.cwd(), 
    'public', 
    'videos',
    `${Date.now()}-${platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content,
      title
    },
    overrideHeight: height,
    overrideWidth: width
  });
  
  return outputPath;
}
```

### 4. Remotion Video Composition

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
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
  
  const contentOpacity = interpolate(
    frame,
    [30, 60],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={durationInFrames}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          padding: 60
        }}>
          <div style={{ 
            opacity: titleOpacity,
            fontSize: 72,
            fontWeight: 'bold',
            color: '#fff',
            textAlign: 'center',
            marginBottom: 40
          }}>
            {title}
          </div>
          
          <div style={{
            opacity: contentOpacity,
            fontSize: 36,
            color: '#e0e0e0',
            textAlign: 'center',
            lineHeight: 1.6
          }}>
            {content.substring(0, 200)}...
          </div>
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## API Routes

### Complete Content Pipeline

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlLatestNews } from '@/lib/research/crawler';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, tone, generateVideo, platform } = await req.json();
    
    // Step 1: Research
    const research = await crawlLatestNews(keyword);
    
    if (research.length === 0) {
      return NextResponse.json(
        { error: 'No research data found' },
        { status: 404 }
      );
    }
    
    // Step 2: Generate Content
    const generator = new ContentGenerator();
    const content = await generator.generateContent({
      format,
      language,
      tone,
      research
    });
    
    let videoUrl = null;
    
    // Step 3: Generate Video (optional)
    if (generateVideo) {
      const videoPath = await renderVideo({
        content,
        title: `${keyword} - ${format}`,
        platform: platform || 'reels'
      });
      
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      data: {
        keyword,
        research: research.length,
        content,
        videoUrl
      }
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline processing failed' },
      { status: 500 }
    );
  }
}
```

## Usage Patterns

### Basic Content Generation

```typescript
// Example: Generate a blog post
const response = await fetch('/api/pipeline', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI automation 2026',
    format: 'how-to',
    language: 'en',
    tone: 'expert',
    generateVideo: false
  })
});

const { data } = await response.json();
console.log('Generated content:', data.content);
```

### Full Pipeline with Video

```typescript
// Example: Create content + video for social media
const response = await fetch('/api/pipeline', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'marketing trends',
    format: 'toplist',
    language: 'vi',
    tone: 'friendly',
    generateVideo: true,
    platform: 'tiktok'
  })
});

const { data } = await response.json();
console.log('Content:', data.content);
console.log('Video URL:', data.videoUrl);
```

### Batch Processing

```typescript
// lib/batch/processor.ts
export async function processBatchKeywords(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      body: JSON.stringify({
        keyword,
        format: 'pov',
        language: 'en',
        tone: 'expert',
        generateVideo: true,
        platform: 'reels'
      })
    });
    
    const data = await response.json();
    results.push(data);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
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

# Preview Remotion video locally
npm run remotion:preview

# Render video directly
npm run remotion:render
```

## Common Troubleshooting

### API Rate Limiting

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>, delay: number = 1000): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          await new Promise(r => setTimeout(r, delay));
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.process();
    });
  }
  
  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
    }
    this.processing = false;
  }
}
```

### Video Rendering Memory Issues

```typescript
// Adjust Remotion memory settings
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setChromiumOpenGlRenderer('angle');
Config.setDelayRenderTimeoutInMilliseconds(90000);
Config.setVideoImageFormat('jpeg');
Config.setJpegQuality(80); // Reduce quality for large renders
```

### Error Handling Wrapper

```typescript
// lib/utils/error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      console.warn(`Attempt ${i + 1} failed:`, error);
      
      if (i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
      }
    }
  }
  
  throw lastError!;
}
```

## Performance Optimization

### Caching Research Results

```typescript
// lib/cache/research-cache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!
});

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached as ResearchResult[] | null;
}

export async function setCachedResearch(
  keyword: string, 
  data: ResearchResult[]
) {
  await redis.setex(`research:${keyword}`, 3600, JSON.stringify(data));
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Automation system, enabling AI agents to effectively assist developers in implementing, customizing, and troubleshooting the content automation workflow.
