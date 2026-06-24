---
name: marketing-pipeline-ai-content-automation
description: AI-powered content pipeline that automates research, scriptwriting, and video generation for marketing teams
triggers:
  - automate content creation with AI research
  - generate marketing videos from articles
  - crawl news sources and create content
  - build AI content pipeline with Claude
  - automate social media content generation
  - create video content from text automatically
  - set up marketing automation pipeline
  - research and generate content with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos automatically using Remotion.

## What This Project Does

The marketing-pipeline-share project provides:

- **Automated Research**: Crawls TechCrunch, a16z, Twitter, LinkedIn for recent news
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to)
- **Multi-language Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Converts written content into videos/infographics using Remotion
- **Format Optimization**: Exports videos optimized for Reels, TikTok, YouTube Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_api_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key Components and Usage

### 1. Research Module - Auto-Crawling

The research module automatically scans news sources and extracts insights:

```typescript
// lib/research/crawler.ts
import { ResearchCrawler } from '@/lib/research/crawler';

async function performResearch(keyword: string) {
  const crawler = new ResearchCrawler({
    apiKey: process.env.RAPIDAPI_KEY,
  });

  const results = await crawler.crawl({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 50,
  });

  return results;
}

// Usage
const insights = await performResearch('AI marketing automation');
console.log(insights);
```

### 2. Content Generation with AI

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentGenerationOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

async function generateContent(options: ContentGenerationOptions) {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(options);

  const response = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return response.content[0].text;
}

function buildPrompt(options: ContentGenerationOptions): string {
  return `Create a ${options.format} article about "${options.keyword}" in ${options.language}.
Tone: ${options.tone}

Research data:
${JSON.stringify(options.researchData, null, 2)}

Requirements:
- Use data-backed insights from the research
- Include statistics and quotes where relevant
- Structure according to ${options.format} format
- Make it engaging and actionable`;
}

// Usage
const article = await generateContent({
  keyword: 'Marketing Automation',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: insights,
});
```

### 3. Video Generation with Remotion

Convert content to video format:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  content: string;
  title: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
  outputPath: string;
}

async function renderVideo(options: VideoRenderOptions) {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: options.content,
      title: options.title,
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: options.outputPath,
    inputProps: {
      content: options.content,
      title: options.title,
      aspectRatio: options.aspectRatio,
    },
  });

  return options.outputPath;
}

// Usage
const videoPath = await renderVideo({
  content: article,
  title: 'Top 5 Marketing Automation Tools',
  aspectRatio: '9:16',
  outputPath: './output/video.mp4',
});
```

### 4. Complete Pipeline Workflow

Full end-to-end automation:

```typescript
// lib/pipeline/workflow.ts
import { performResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/content/generator';
import { renderVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  formats: ('toplist' | 'pov' | 'case-study' | 'how-to')[];
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoAspectRatio?: '9:16' | '16:9' | '1:1';
}

async function runPipeline(config: PipelineConfig) {
  console.log(`🔍 Starting research for: ${config.keyword}`);
  
  // Step 1: Research
  const researchData = await performResearch(config.keyword);
  
  const results = [];
  
  // Step 2: Generate content for each format and language
  for (const format of config.formats) {
    for (const language of config.languages) {
      console.log(`✍️ Generating ${format} in ${language}...`);
      
      const content = await generateContent({
        keyword: config.keyword,
        format,
        language,
        tone: 'expert',
        researchData,
      });
      
      results.push({
        format,
        language,
        content,
      });
      
      // Step 3: Generate video if enabled
      if (config.generateVideo) {
        console.log(`🎬 Rendering video for ${format} (${language})...`);
        
        const videoPath = await renderVideo({
          content,
          title: config.keyword,
          aspectRatio: config.videoAspectRatio || '9:16',
          outputPath: `./output/${format}-${language}.mp4`,
        });
        
        results[results.length - 1].videoPath = videoPath;
      }
    }
  }
  
  console.log('✅ Pipeline complete!');
  return results;
}

// Usage
const pipelineResults = await runPipeline({
  keyword: 'AI Content Marketing 2026',
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  videoAspectRatio: '9:16',
});
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { performResearch } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeRange } = await request.json();
    
    const results = await performResearch(keyword);
    
    return NextResponse.json({
      success: true,
      data: results,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  try {
    const options = await request.json();
    
    const content = await generateContent(options);
    
    return NextResponse.json({
      success: true,
      content,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Pipeline Endpoint

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runPipeline } from '@/lib/pipeline/workflow';

export async function POST(request: NextRequest) {
  try {
    const config = await request.json();
    
    const results = await runPipeline(config);
    
    return NextResponse.json({
      success: true,
      results,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Run specific pipeline script
npm run pipeline -- --keyword "AI Marketing" --format toplist
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const pipelineResult = await runPipeline({
      keyword,
      formats: ['toplist'],
      languages: ['en', 'vi'],
      generateVideo: false,
    });
    
    results.push(...pipelineResult);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Research Sources

```typescript
// lib/research/custom-source.ts
interface CustomSource {
  name: string;
  url: string;
  selector: string;
}

async function addCustomSource(source: CustomSource) {
  const crawler = new ResearchCrawler({
    apiKey: process.env.RAPIDAPI_KEY,
  });
  
  crawler.addSource({
    name: source.name,
    fetchFn: async () => {
      // Custom crawling logic
      const response = await fetch(source.url);
      const html = await response.text();
      // Parse with cheerio or similar
      return parsedArticles;
    },
  });
}
```

### Video Template Customization

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  content: string;
  title: string;
  aspectRatio: string;
}> = ({ content, title, aspectRatio }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{
        padding: 60,
        color: 'white',
        fontSize: 48,
        fontWeight: 'bold',
      }}>
        {title}
      </div>
      <div style={{
        padding: 60,
        paddingTop: 140,
        color: '#e0e0e0',
        fontSize: 32,
        lineHeight: 1.5,
      }}>
        {content.slice(0, 200)}...
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

If encountering rate limits:

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.process();
    });
  }
  
  private async process() {
    if (this.processing) return;
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      await fn();
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
    
    this.processing = false;
  }
}

const limiter = new RateLimiter();

// Usage
const content = await limiter.add(() => generateContent(options));
```

### Video Rendering Errors

Check Remotion configuration:

```bash
# Install required dependencies
npm install @remotion/bundler @remotion/renderer @remotion/cli

# Test rendering
npx remotion preview remotion/index.ts
```

### Research Crawler Issues

Handle crawler failures gracefully:

```typescript
async function safeResearch(keyword: string) {
  try {
    return await performResearch(keyword);
  } catch (error) {
    console.error('Research failed:', error);
    // Fallback to cached data or alternative source
    return getCachedResearch(keyword);
  }
}
```

### Memory Issues with Large Content

Stream processing for large batches:

```typescript
async function* streamPipeline(keywords: string[]) {
  for (const keyword of keywords) {
    const result = await runPipeline({
      keyword,
      formats: ['toplist'],
      languages: ['en'],
      generateVideo: false,
    });
    
    yield result;
  }
}

// Usage
for await (const result of streamPipeline(keywords)) {
  console.log('Generated:', result);
  // Process immediately or save to database
}
```

## Best Practices

1. **Always validate API keys** before running pipeline
2. **Use environment variables** for all credentials
3. **Implement retry logic** for API calls
4. **Cache research data** to avoid redundant crawls
5. **Monitor API usage** to stay within rate limits
6. **Test video rendering** with small content first
7. **Validate content quality** before video generation
