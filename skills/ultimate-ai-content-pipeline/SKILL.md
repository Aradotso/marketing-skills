---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline from research to script generation and video rendering using Claude, OpenAI, and Remotion
triggers:
  - how do I use the Ultimate AI Content Pipeline
  - set up automated content generation with research
  - create videos from content using Remotion
  - configure Claude and OpenAI for content automation
  - crawl news sources and generate articles automatically
  - render videos from AI-generated scripts
  - build an automated content marketing pipeline
  - generate multilingual content with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from crawling trending news, to generating multilingual articles with AI (Claude/OpenAI), to rendering videos with Remotion. It's designed for content creators, marketers, and businesses to optimize their content production by up to 90%.

## What It Does

- **Auto-Research**: Crawls real-time data from TechCrunch, a16z, X (Twitter), LinkedIn for trending topics
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual Support**: Generates content in both English and Vietnamese with customizable tones
- **Video Rendering**: Automatically converts written content into videos and infographics using Remotion
- **Multi-Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

```env
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application Settings
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app directory
│   ├── components/          # React components
│   ├── lib/                 # Core utilities
│   │   ├── ai/             # AI service integrations
│   │   ├── crawler/        # News crawling logic
│   │   ├── video/          # Remotion video generation
│   │   └── content/        # Content processing
│   ├── types/              # TypeScript types
│   └── utils/              # Helper functions
├── remotion/               # Remotion video templates
├── public/                 # Static assets
└── package.json
```

## Key Components & Usage

### 1. Content Research & Crawling

```typescript
// src/lib/crawler/news-scraper.ts
import { NewsSource } from '@/types/research';

interface CrawlOptions {
  sources: NewsSource[];
  timeRange: '24h' | '7d' | '30d';
  keywords?: string[];
  maxResults?: number;
}

export async function crawlNews(options: CrawlOptions) {
  const { sources, timeRange, keywords, maxResults = 50 } = options;
  
  const results = await Promise.all(
    sources.map(source => fetchFromSource(source, timeRange, keywords))
  );
  
  return results.flat().slice(0, maxResults);
}

// Usage example
const trendingNews = await crawlNews({
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: '24h',
  keywords: ['AI', 'startup', 'marketing'],
  maxResults: 20
});
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any[];
}

export async function generateContentWithClaude(request: ContentRequest) {
  const { topic, format, tone, language, researchData } = request;
  
  const prompt = buildPrompt(topic, format, tone, language, researchData);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return parseContentResponse(message.content);
}

function buildPrompt(
  topic: string,
  format: string,
  tone: string,
  language: string,
  data: any[]
): string {
  const dataContext = data.map(item => 
    `- ${item.title} (${item.source}): ${item.summary}`
  ).join('\n');
  
  return `You are an expert content writer. Create a ${format} article about "${topic}" in ${language}.
  
Tone: ${tone}
Recent research data:
${dataContext}

Requirements:
- Use data-backed insights from the research
- Include statistics and real examples
- Structure content for readability
- Add engaging headlines and subheadings
${language === 'vi' ? '- Write in Vietnamese with natural, native-sounding language' : ''}

Generate the complete article:`;
}
```

### 3. AI Content Generation with OpenAI

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentWithOpenAI(request: ContentRequest) {
  const prompt = buildPrompt(
    request.topic,
    request.format,
    request.tone,
    request.language,
    request.researchData
  );
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketing writer specializing in creating engaging, data-driven articles.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  contentTitle: string;
  contentBody: string;
  keyPoints: string[];
  platform: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

const PLATFORM_DIMENSIONS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

export async function generateVideo(config: VideoConfig) {
  const { platform, contentTitle, contentBody, keyPoints, duration } = config;
  const { width, height } = PLATFORM_DIMENSIONS[platform];
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: contentTitle,
      content: contentBody,
      keyPoints: keyPoints,
      duration: duration
    }
  });
  
  // Render video
  const outputLocation = path.resolve(`./output/${Date.now()}-${platform}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: contentTitle,
      content: contentBody,
      keyPoints: keyPoints
    }
  });
  
  return outputLocation;
}
```

### 5. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import React from 'react';
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  keyPoints: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  content, 
  keyPoints 
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  // Title animation
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
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        padding: '60px',
        color: 'white',
        fontFamily: 'Arial, sans-serif'
      }}>
        <h1 style={{
          opacity: titleOpacity,
          transform: `scale(${titleScale})`,
          fontSize: '64px',
          textAlign: 'center',
          marginBottom: '40px',
          fontWeight: 'bold'
        }}>
          {title}
        </h1>
        
        <div style={{
          fontSize: '32px',
          lineHeight: '1.6',
          maxWidth: '900px',
          textAlign: 'center'
        }}>
          {keyPoints.map((point, index) => {
            const pointOpacity = interpolate(
              frame,
              [60 + index * 90, 90 + index * 90],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <div key={index} style={{ 
                opacity: pointOpacity,
                marginBottom: '30px',
                padding: '20px',
                backgroundColor: 'rgba(255, 255, 255, 0.1)',
                borderRadius: '10px'
              }}>
                ✓ {point}
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Full Pipeline Integration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlNews } from '@/lib/crawler/news-scraper';
import { generateContentWithClaude } from '@/lib/ai/claude-generator';
import { generateVideo } from '@/lib/video/video-renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  platforms: ('reels' | 'tiktok' | 'shorts')[];
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('🚀 Starting content pipeline...');
  
  // Step 1: Research
  console.log('📡 Crawling news sources...');
  const researchData = await crawlNews({
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    keywords: [config.keyword],
    maxResults: 20
  });
  
  // Step 2: Generate content for each language
  console.log('🧠 Generating content with AI...');
  const articles = await Promise.all(
    config.languages.map(async (lang) => {
      const content = await generateContentWithClaude({
        topic: config.keyword,
        format: config.contentFormat,
        tone: config.tone,
        language: lang,
        researchData
      });
      
      return { language: lang, content };
    })
  );
  
  // Step 3: Generate videos (if enabled)
  let videos = [];
  if (config.generateVideo) {
    console.log('🎬 Rendering videos...');
    
    for (const article of articles) {
      for (const platform of config.platforms) {
        const videoPath = await generateVideo({
          contentTitle: article.content.title,
          contentBody: article.content.body,
          keyPoints: article.content.keyPoints,
          platform,
          duration: 30
        });
        
        videos.push({
          language: article.language,
          platform,
          path: videoPath
        });
      }
    }
  }
  
  console.log('✅ Pipeline complete!');
  
  return {
    researchData,
    articles,
    videos
  };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI marketing automation',
  contentFormat: 'how-to',
  tone: 'expert',
  languages: ['en', 'vi'],
  platforms: ['reels', 'tiktok'],
  generateVideo: true
});
```

## API Routes (Next.js)

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'how-to',
      tone: body.tone || 'expert',
      languages: body.languages || ['en'],
      platforms: body.platforms || ['reels'],
      generateVideo: body.generateVideo ?? true
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
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

# Build for production
npm run build

# Start production server
npm run start

# Run Remotion studio (for video template editing)
npm run remotion:studio
```

## Common Patterns

### Pattern 1: Single Article Generation

```typescript
import { crawlNews } from '@/lib/crawler/news-scraper';
import { generateContentWithClaude } from '@/lib/ai/claude-generator';

async function createSingleArticle(keyword: string) {
  const research = await crawlNews({
    sources: ['techcrunch'],
    timeRange: '24h',
    keywords: [keyword],
    maxResults: 10
  });
  
  const article = await generateContentWithClaude({
    topic: keyword,
    format: 'toplist',
    tone: 'friendly',
    language: 'en',
    researchData: research
  });
  
  return article;
}
```

### Pattern 2: Batch Video Generation

```typescript
async function generateMultiplePlatformVideos(content: any) {
  const platforms = ['reels', 'tiktok', 'shorts'] as const;
  
  const videos = await Promise.all(
    platforms.map(platform => 
      generateVideo({
        contentTitle: content.title,
        contentBody: content.body,
        keyPoints: content.keyPoints,
        platform,
        duration: 30
      })
    )
  );
  
  return videos;
}
```

### Pattern 3: Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Running scheduled content pipeline...');
  
  await runContentPipeline({
    keyword: 'daily tech trends',
    contentFormat: 'toplist',
    tone: 'expert',
    languages: ['en', 'vi'],
    platforms: ['reels'],
    generateVideo: true
  });
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff for API calls
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Remotion Memory Issues

If video rendering fails with memory errors:

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Claude/OpenAI Timeout

```typescript
// Add timeout handling
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 60000); // 60s timeout

try {
  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [...],
    signal: controller.signal
  });
} finally {
  clearTimeout(timeoutId);
}
```

### Missing Environment Variables

```typescript
// src/lib/config/validate-env.ts
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
```

## Best Practices

1. **Always validate environment variables on startup**
2. **Implement rate limiting for external API calls**
3. **Cache research data to avoid redundant crawling**
4. **Use TypeScript strict mode for type safety**
5. **Handle video rendering asynchronously for better UX**
6. **Store generated content in a database for reuse**
7. **Implement proper error logging and monitoring**

## Additional Resources

- [Anthropic Claude API Documentation](https://docs.anthropic.com/)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Remotion Documentation](https://www.remotion.dev/docs)
- [Next.js App Router](https://nextjs.org/docs/app)
