---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from blog posts automatically
  - crawl news and create content with Claude
  - set up AI content pipeline with Remotion
  - build automated marketing content workflow
  - create multilingual content with AI
  - generate social media videos from articles
  - automate research to video content pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with Ultimate AI Content Pipeline, a comprehensive TypeScript-based automation system that handles the entire content creation workflow: from researching trending topics and generating articles to automatically rendering videos for social media platforms.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scans research sources** - Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh news within 24 hours
- **Generates multi-format content** - Creates articles in various formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports multilingual output** - Automatically produces English and Vietnamese versions
- **Renders videos automatically** - Uses Remotion to convert articles into Reels/TikTok/Shorts-ready videos
- **Manages posting workflows** - Schedules and publishes content across platforms

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install pnpm (recommended) or npm
npm install -g pnpm
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create `.env` file with required API keys:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_key
OPENAI_API_KEY=your_openai_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=postgresql://user:pass@localhost:5432/content_db

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social Media APIs
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

### Start Development Server

```bash
# Start Next.js development server
pnpm dev

# Server runs on http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── research/    # News crawling & research
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video generation
│   ├── remotion/        # Remotion video templates
│   └── utils/           # Helper functions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Research Module - Auto-Scan News

```typescript
// src/lib/research/scanner.ts
import { RapidAPIClient } from '@/lib/research/rapid-client';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: Date;
  source: string;
  summary: string;
}

export async function scanTrendingNews(
  topic: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<NewsArticle[]> {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const sources = [
    'techcrunch',
    'a16z',
    'twitter-trending',
    'linkedin-pulse'
  ];
  
  const articles = await Promise.all(
    sources.map(source => 
      client.fetchNews(source, topic, timeframe)
    )
  );
  
  return articles.flat().sort((a, b) => 
    b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}

// Usage example
const news = await scanTrendingNews('AI automation', '24h');
console.log(`Found ${news.length} articles`);
```

### 2. Content Generation with Claude

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

export class ContentGenerator {
  private anthropic: Anthropic;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }
  
  async generateArticle(
    topic: string,
    research: string,
    options: ContentOptions
  ): Promise<string> {
    const prompt = this.buildPrompt(topic, research, options);
    
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  private buildPrompt(
    topic: string,
    research: string,
    options: ContentOptions
  ): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with clear rankings',
      'pov': 'Write from a personal perspective with opinions',
      'case-study': 'Analyze with data, examples, and outcomes',
      'how-to': 'Provide step-by-step actionable instructions'
    };
    
    return `
You are an expert content writer creating a ${options.format} article.

Topic: ${topic}
Research Data: ${research}
Target Audience: ${options.targetAudience}
Tone: ${options.tone}
Language: ${options.language}

Instructions: ${formatInstructions[options.format]}

Create engaging, data-backed content with clear structure, headlines, and actionable insights.
`;
  }
}

// Usage
const generator = new ContentGenerator();
const article = await generator.generateArticle(
  'AI Content Automation in 2024',
  researchData,
  {
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    targetAudience: 'Digital marketers and content creators'
  }
);
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  brandColors: {
    primary: string;
    secondary: string;
  };
  duration: number; // in seconds
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export class VideoRenderer {
  async renderContentVideo(
    articleContent: string,
    config: VideoConfig
  ): Promise<string> {
    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
      webpackOverride: (config) => config,
    });
    
    // Extract key points from article
    const inputProps = {
      title: config.title,
      keyPoints: config.keyPoints,
      colors: config.brandColors,
    };
    
    // Get composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: 'ContentVideo',
      inputProps,
    });
    
    // Render video
    const outputPath = path.join(
      process.cwd(), 
      'public/videos',
      `${Date.now()}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps,
    });
    
    return outputPath;
  }
}

// Usage
const renderer = new VideoRenderer();
const videoPath = await renderer.renderContentVideo(
  article,
  {
    title: '5 AI Tools for 2024',
    keyPoints: ['Tool 1: AutoGPT', 'Tool 2: Claude', 'Tool 3: Midjourney'],
    brandColors: { primary: '#6366f1', secondary: '#ec4899' },
    duration: 60,
    aspectRatio: '9:16'
  }
);
```

### 4. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  colors: { primary: string; secondary: string };
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  colors
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const fadeIn = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor: colors.primary }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill 
          style={{ 
            justifyContent: 'center', 
            alignItems: 'center',
            opacity: fadeIn 
          }}
        >
          <h1 style={{ 
            fontSize: 72, 
            color: 'white',
            textAlign: 'center',
            padding: '0 40px'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Key Points Sequences */}
      {keyPoints.map((point, index) => (
        <Sequence 
          key={index}
          from={fps * (3 + index * 5)} 
          durationInFrames={fps * 5}
        >
          <AbsoluteFill style={{
            justifyContent: 'center',
            alignItems: 'center',
            backgroundColor: colors.secondary
          }}>
            <div style={{
              fontSize: 48,
              color: 'white',
              padding: '0 60px',
              textAlign: 'center'
            }}>
              <div style={{ fontSize: 96, marginBottom: 20 }}>
                {index + 1}
              </div>
              <p>{point}</p>
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { scanTrendingNews } from '@/lib/research/scanner';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/video/video-renderer';

interface PipelineResult {
  article: {
    title: string;
    content: string;
    language: string;
  };
  video: {
    path: string;
    duration: number;
  };
  metadata: {
    sources: string[];
    generatedAt: Date;
  };
}

export class ContentPipeline {
  private contentGenerator: ContentGenerator;
  private videoRenderer: VideoRenderer;
  
  constructor() {
    this.contentGenerator = new ContentGenerator();
    this.videoRenderer = new VideoRenderer();
  }
  
  async executeFullPipeline(
    keyword: string,
    options: {
      format: 'toplist' | 'pov' | 'case-study' | 'how-to';
      language: 'en' | 'vi';
      includeVideo: boolean;
    }
  ): Promise<PipelineResult> {
    // Step 1: Research
    console.log('🔍 Scanning trending news...');
    const news = await scanTrendingNews(keyword, '24h');
    const researchData = news.map(n => 
      `${n.title}: ${n.summary} (${n.source})`
    ).join('\n\n');
    
    // Step 2: Generate Content
    console.log('✍️ Generating article...');
    const article = await this.contentGenerator.generateArticle(
      keyword,
      researchData,
      {
        format: options.format,
        language: options.language,
        tone: 'expert',
        targetAudience: 'marketers and content creators'
      }
    );
    
    // Step 3: Extract title and key points
    const title = this.extractTitle(article);
    const keyPoints = this.extractKeyPoints(article, 5);
    
    let videoPath = '';
    let videoDuration = 0;
    
    // Step 4: Generate Video (if requested)
    if (options.includeVideo) {
      console.log('🎬 Rendering video...');
      videoPath = await this.videoRenderer.renderContentVideo(
        article,
        {
          title,
          keyPoints,
          brandColors: { primary: '#6366f1', secondary: '#ec4899' },
          duration: 60,
          aspectRatio: '9:16'
        }
      );
      videoDuration = 60;
    }
    
    console.log('✅ Pipeline complete!');
    
    return {
      article: {
        title,
        content: article,
        language: options.language
      },
      video: {
        path: videoPath,
        duration: videoDuration
      },
      metadata: {
        sources: news.map(n => n.url),
        generatedAt: new Date()
      }
    };
  }
  
  private extractTitle(article: string): string {
    const titleMatch = article.match(/^#\s+(.+)$/m);
    return titleMatch ? titleMatch[1] : 'Untitled Article';
  }
  
  private extractKeyPoints(article: string, count: number): string[] {
    const points: string[] = [];
    const lines = article.split('\n');
    
    for (const line of lines) {
      if (line.match(/^#+\s+\d+\.|^\d+\.|^-\s+/)) {
        points.push(line.replace(/^#+\s+\d+\.|^\d+\.|^-\s+/, '').trim());
        if (points.length >= count) break;
      }
    }
    
    return points;
  }
}

// Usage
const pipeline = new ContentPipeline();
const result = await pipeline.executeFullPipeline(
  'AI Marketing Automation',
  {
    format: 'toplist',
    language: 'en',
    includeVideo: true
  }
);

console.log('Article:', result.article.title);
console.log('Video:', result.video.path);
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, includeVideo } = body;
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.executeFullPipeline(keyword, {
      format: format || 'toplist',
      language: language || 'en',
      includeVideo: includeVideo ?? true
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## CLI Commands

### Development

```bash
# Start development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Type checking
pnpm type-check
```

### Remotion Commands

```bash
# Preview Remotion compositions
pnpm remotion preview

# Render specific composition
pnpm remotion render ContentVideo output.mp4

# Upgrade Remotion
pnpm remotion upgrade
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
// Generate multiple articles for different keywords
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      pipeline.executeFullPipeline(keyword, {
        format: 'toplist',
        language: 'en',
        includeVideo: false
      })
    )
  );
  
  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
  
  return successful;
}
```

### Pattern 2: Multilingual Content

```typescript
// Generate same content in multiple languages
async function generateMultilingual(keyword: string) {
  const pipeline = new ContentPipeline();
  
  const [enVersion, viVersion] = await Promise.all([
    pipeline.executeFullPipeline(keyword, {
      format: 'how-to',
      language: 'en',
      includeVideo: false
    }),
    pipeline.executeFullPipeline(keyword, {
      format: 'how-to',
      language: 'vi',
      includeVideo: false
    })
  ]);
  
  return { enVersion, viVersion };
}
```

### Pattern 3: Custom Video Templates

```typescript
// Create custom Remotion template
// src/remotion/CustomTemplate.tsx
import { AbsoluteFill, Img, staticFile } from 'remotion';

export const CustomTemplate: React.FC<{
  logo: string;
  content: string;
}> = ({ logo, content }) => {
  return (
    <AbsoluteFill>
      <Img src={staticFile(logo)} style={{ width: 100 }} />
      <p>{content}</p>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### Issue: API Rate Limits

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
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Issue: Video Rendering Memory Errors

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" pnpm dev

# Or in package.json scripts:
"scripts": {
  "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev"
}
```

### Issue: Missing Environment Variables

```typescript
// src/lib/utils/config.ts
export function validateEnv() {
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

// Call at app startup
validateEnv();
```

### Issue: Remotion Composition Not Found

```typescript
// Ensure composition is registered in src/remotion/index.ts
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={1800} // 60 seconds at 30fps
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Sample Title',
          keyPoints: [],
          colors: { primary: '#000', secondary: '#fff' }
        }}
      />
    </>
  );
};
```

## Best Practices

1. **Always validate inputs** before passing to AI or rendering
2. **Cache research results** to avoid redundant API calls
3. **Use TypeScript strictly** for type safety across pipeline
4. **Monitor API costs** - Claude and OpenAI can be expensive at scale
5. **Test video renders locally** before deploying to production
6. **Implement queue systems** for batch processing (use Bull or BullMQ)
7. **Store generated content** in database for reuse and analytics

## Advanced Configuration

### Custom Research Sources

```typescript
// src/lib/research/custom-sources.ts
export async function addCustomSource(
  sourceName: string,
  fetchFn: (topic: string) => Promise<NewsArticle[]>
) {
  // Register custom crawler
  sourceRegistry.set(sourceName, fetchFn);
}
```

### AI Model Selection

```typescript
// Switch between Claude models
const generator = new ContentGenerator();
generator.setModel('claude-3-opus-20240229'); // More powerful
// or
generator.setModel('claude-3-haiku-20240307'); // Faster, cheaper
```

This skill provides comprehensive guidance for using the Ultimate AI Content Pipeline to automate content creation from research to video generation.
