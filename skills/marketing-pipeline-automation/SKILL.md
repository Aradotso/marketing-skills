---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - research and generate marketing content automatically
  - create video content from text with Remotion
  - build automated content workflow with Claude
  - generate multilingual content with AI
  - crawl news sources for content research
  - set up AI content generation pipeline
  - automate social media content creation
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to work with an end-to-end content automation pipeline that handles research, scriptwriting, content generation, and video rendering. The system crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 and OpenAI to generate multi-format content in multiple languages, and finally renders videos using Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes data from major news sources (24h data)
2. **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
3. **Multilingual Output**: Generates content in English and Vietnamese simultaneously
4. **Video Generation**: Automatically renders videos and infographics using Remotion
5. **Multi-Platform Export**: Optimizes content for Reels, TikTok, Shorts

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
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion rendering
npm run remotion
```

## Key Architecture Components

### 1. Research Module (Data Crawling)

```typescript
// lib/research/crawler.ts
import { scrapeArticles } from './scrapers';

interface ResearchConfig {
  sources: string[];
  timeRange: number; // hours
  keywords: string[];
}

async function performResearch(config: ResearchConfig) {
  const articles = await scrapeArticles({
    sources: config.sources,
    since: Date.now() - (config.timeRange * 60 * 60 * 1000)
  });
  
  const filtered = articles.filter(article => 
    config.keywords.some(keyword => 
      article.title.toLowerCase().includes(keyword.toLowerCase()) ||
      article.content.toLowerCase().includes(keyword.toLowerCase())
    )
  );
  
  return filtered;
}

// Usage example
const research = await performResearch({
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: 24,
  keywords: ['AI', 'marketing', 'automation']
});
```

### 2. Content Generation with Claude/OpenAI

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any[];
}

async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);
  
  // Using Claude for content generation
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

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list format with detailed explanations',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Structure as a problem-solution case study with data',
    'how-to': 'Create a step-by-step tutorial with actionable tips'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include light humor while maintaining value'
  };
  
  return `
You are a content creator for ${request.language === 'en' ? 'English' : 'Vietnamese'} audience.

Topic: ${request.topic}
Format: ${formatInstructions[request.format]}
Tone: ${toneInstructions[request.tone]}

Research Data:
${JSON.stringify(request.researchData, null, 2)}

Create comprehensive content that:
1. Incorporates the latest research data
2. Provides actionable insights
3. Includes specific examples and statistics
4. Maintains the specified tone throughout
5. Is optimized for ${request.language === 'en' ? 'English' : 'Vietnamese'} readers
`;
}
```

### 3. API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { performResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, language } = body;
    
    // Step 1: Research
    const researchData = await performResearch({
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeRange: 24,
      keywords: [keyword]
    });
    
    // Step 2: Generate content
    const content = await generateContent({
      topic: keyword,
      format,
      tone,
      language,
      researchData
    });
    
    // Step 3: Return result
    return NextResponse.json({
      success: true,
      content,
      sources: researchData.length
    });
    
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoProps {
  title: string;
  content: string[];
  style: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<VideoProps> = ({ title, content, style }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{
        padding: '40px',
        color: 'white',
        opacity,
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center'
      }}>
        <h1 style={{ fontSize: '48px', marginBottom: '20px' }}>
          {title}
        </h1>
        {content.map((point, index) => {
          const pointFrame = frame - (index * fps * 2);
          const pointOpacity = Math.max(0, Math.min(1, pointFrame / fps));
          
          return (
            <p key={index} style={{
              fontSize: '24px',
              marginBottom: '15px',
              opacity: pointOpacity,
              transform: `translateY(${Math.max(0, 20 - pointFrame)}px)`
            }}>
              • {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderConfig {
  title: string;
  content: string[];
  style: 'reels' | 'tiktok' | 'shorts';
  outputPath: string;
}

async function renderVideo(config: RenderConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style
    },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: config.outputPath,
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style
    },
  });
  
  return config.outputPath;
}

export { renderVideo };
```

### 5. Complete Pipeline Integration

```typescript
// lib/pipeline/orchestrator.ts
import { performResearch } from '../research/crawler';
import { generateContent } from '../ai/content-generator';
import { renderVideo } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoStyle?: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for: ${config.keyword}`);
  
  // Phase 1: Research
  console.log('Phase 1: Researching...');
  const researchData = await performResearch({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: 24,
    keywords: [config.keyword]
  });
  
  // Phase 2: Generate content for each language
  console.log('Phase 2: Generating content...');
  const contents = await Promise.all(
    config.languages.map(async (language) => {
      const content = await generateContent({
        topic: config.keyword,
        format: config.format,
        tone: config.tone,
        language,
        researchData
      });
      
      return { language, content };
    })
  );
  
  // Phase 3: Generate videos if requested
  let videos = [];
  if (config.generateVideo) {
    console.log('Phase 3: Rendering videos...');
    videos = await Promise.all(
      contents.map(async ({ language, content }) => {
        const videoPath = `./output/video_${config.keyword}_${language}.mp4`;
        const extractedPoints = extractKeyPoints(content);
        
        await renderVideo({
          title: config.keyword,
          content: extractedPoints,
          style: config.videoStyle || 'reels',
          outputPath: videoPath
        });
        
        return { language, videoPath };
      })
    );
  }
  
  console.log('Pipeline complete!');
  
  return {
    research: researchData,
    contents,
    videos
  };
}

function extractKeyPoints(content: string): string[] {
  // Extract bullet points or key sentences
  const lines = content.split('\n');
  return lines
    .filter(line => line.trim().length > 0)
    .slice(0, 5)
    .map(line => line.replace(/^[•\-\d.]+\s*/, '').trim());
}

export { runContentPipeline };
```

## Common Usage Patterns

### Pattern 1: Single Content Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

// Generate a single piece of content
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  format: 'toplist',
  tone: 'expert',
  languages: ['en'],
  generateVideo: false
});

console.log(result.contents[0].content);
```

### Pattern 2: Multi-Language Content with Video

```typescript
// Generate content in both languages with video
const result = await runContentPipeline({
  keyword: 'Content Automation Best Practices',
  format: 'how-to',
  tone: 'friendly',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoStyle: 'reels'
});

console.log(`Generated ${result.contents.length} articles`);
console.log(`Generated ${result.videos.length} videos`);
```

### Pattern 3: Batch Processing

```typescript
const topics = [
  'AI Marketing Automation',
  'Social Media Trends',
  'Content Strategy Tips'
];

const results = await Promise.all(
  topics.map(topic => runContentPipeline({
    keyword: topic,
    format: 'pov',
    tone: 'expert',
    languages: ['en', 'vi'],
    generateVideo: true,
    videoStyle: 'shorts'
  }))
);

console.log(`Processed ${results.length} topics successfully`);
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delayMs: number;
  
  constructor(delayMs: number = 1000) {
    this.delayMs = delayMs;
  }
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.processQueue();
    });
  }
  
  private async processQueue() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift()!;
    
    await fn();
    await new Promise(resolve => setTimeout(resolve, this.delayMs));
    
    this.processing = false;
    this.processQueue();
  }
}

export const aiRateLimiter = new RateLimiter(2000); // 2s delay between AI calls
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delayMs: number = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      console.warn(`Attempt ${i + 1} failed, retrying in ${delayMs}ms...`);
      await new Promise(resolve => setTimeout(resolve, delayMs));
      delayMs *= 2; // Exponential backoff
    }
  }
  
  throw new Error('Max retries reached');
}
```

### Memory Management for Large Research Data

```typescript
// lib/utils/data-processor.ts
export function chunkArray<T>(array: T[], chunkSize: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < array.length; i += chunkSize) {
    chunks.push(array.slice(i, i + chunkSize));
  }
  return chunks;
}

// Process research data in chunks
async function processResearchInChunks(data: any[], chunkSize: number = 10) {
  const chunks = chunkArray(data, chunkSize);
  const results = [];
  
  for (const chunk of chunks) {
    const processed = await Promise.all(
      chunk.map(item => processResearchItem(item))
    );
    results.push(...processed);
  }
  
  return results;
}
```

## Configuration Best Practices

### Recommended API Settings

```typescript
// config/ai-settings.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7,
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7,
  },
  rateLimits: {
    claude: 50, // requests per minute
    openai: 60,
  }
};

export const RESEARCH_CONFIG = {
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  maxArticlesPerSource: 20,
  timeRangeHours: 24,
};

export const VIDEO_CONFIG = {
  fps: 30,
  dimensions: {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  },
  codec: 'h264',
  quality: 'high',
};
```

This skill provides comprehensive coverage of the marketing automation pipeline, enabling AI agents to effectively assist developers in implementing automated content generation workflows with research, AI writing, and video rendering capabilities.
