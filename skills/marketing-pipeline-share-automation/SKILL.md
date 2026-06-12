---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, auto-posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - set up AI marketing pipeline with Claude and OpenAI
  - generate videos automatically from articles
  - crawl news and create content automatically
  - build automated content workflow with Remotion
  - create multilingual marketing content pipeline
  - auto-post content to social media pages
  - research trending topics and generate scripts
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that handles the entire content lifecycle: from researching trending topics, generating scripts in multiple formats and languages, to rendering videos and auto-posting to social platforms. Built with TypeScript, Next.js, and integrates Claude 3, OpenAI, and Remotion for video generation.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Dual language support (English/Vietnamese) with customizable tone
- Automatic video/infographic rendering via Remotion
- Multi-platform optimization (Reels, TikTok, Shorts)

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

# Set up environment variables
cp .env.example .env.local
```

## Configuration

### Environment Variables

Create a `.env.local` file with the following:

```env
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Social Media Auto-posting
FACEBOOK_PAGE_ACCESS_TOKEN=your_facebook_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Next.js Setup

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion
```

## Core Workflows

### 1. Auto-Research Content

The research module crawls trending topics from multiple sources:

```typescript
// src/lib/research/crawler.ts
import { TechCrunchCrawler } from './sources/techcrunch';
import { TwitterCrawler } from './sources/twitter';
import { LinkedInCrawler } from './sources/linkedin';

interface ResearchResult {
  title: string;
  summary: string;
  source: string;
  url: string;
  publishedAt: Date;
  insights: string[];
}

export async function autoResearch(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<ResearchResult[]> {
  const crawlers = [
    new TechCrunchCrawler(process.env.RAPIDAPI_KEY!),
    new TwitterCrawler(process.env.TWITTER_BEARER_TOKEN!),
    new LinkedInCrawler(process.env.LINKEDIN_ACCESS_TOKEN!)
  ];

  const results = await Promise.all(
    crawlers.map(crawler => crawler.search(keyword, timeframe))
  );

  return results.flat().sort((a, b) => 
    b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}

// Usage
const trendingTopics = await autoResearch('AI marketing automation', '24h');
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type ContentTone = 'expert' | 'friendly' | 'humorous';
type Language = 'en' | 'vi';

interface ContentConfig {
  format: ContentFormat;
  tone: ContentTone;
  language: Language;
  researchData: ResearchResult[];
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

  async generateArticle(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);

    // Use Claude 3 for content generation
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    const content = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';

    return content;
  }

  private buildPrompt(config: ContentConfig): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list format highlighting top items/trends',
      'pov': 'Write from a unique perspective with strong opinions',
      'case-study': 'Present a detailed analysis with data and examples',
      'how-to': 'Provide step-by-step actionable instructions'
    };

    const toneInstructions = {
      'expert': 'Use professional language with industry terminology',
      'friendly': 'Use conversational, approachable language',
      'humorous': 'Include wit and light humor while staying informative'
    };

    const languageInstructions = {
      'en': 'Write in English',
      'vi': 'Write in Vietnamese'
    };

    return `
You are a professional content creator. Create an article based on the following:

FORMAT: ${formatInstructions[config.format]}
TONE: ${toneInstructions[config.tone]}
LANGUAGE: ${languageInstructions[config.language]}

RESEARCH DATA:
${config.researchData.map((r, i) => `
${i + 1}. ${r.title}
   Source: ${r.source}
   Summary: ${r.summary}
   Insights: ${r.insights.join(', ')}
`).join('\n')}

Requirements:
- Use data and examples from the research
- Include specific numbers and dates
- Make it engaging and actionable
- Optimize for social media sharing
- Include relevant hashtags

Generate the complete article now:
`;
  }

  async generateDualLanguage(config: Omit<ContentConfig, 'language'>): Promise<{
    en: string;
    vi: string;
  }> {
    const [enContent, viContent] = await Promise.all([
      this.generateArticle({ ...config, language: 'en' }),
      this.generateArticle({ ...config, language: 'vi' })
    ]);

    return { en: enContent, vi: viContent };
  }
}
```

### 3. Video Generation with Remotion

Automatically render videos from article content:

```typescript
// src/remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, Audio, Img, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ArticleVideoProps {
  title: string;
  points: string[];
  images: string[];
  audioUrl?: string;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  points,
  images,
  audioUrl
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const pointDuration = Math.floor(durationInFrames / points.length);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      {audioUrl && <Audio src={audioUrl} />}
      
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill style={{
          justifyContent: 'center',
          alignItems: 'center',
          padding: 40
        }}>
          <h1 style={{
            fontSize: 60,
            color: '#fff',
            textAlign: 'center',
            fontWeight: 'bold'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Points Sequences */}
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={(index + 1) * pointDuration}
          durationInFrames={pointDuration}
        >
          <AbsoluteFill style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60
          }}>
            {images[index] && (
              <Img
                src={images[index]}
                style={{
                  width: '100%',
                  height: '60%',
                  objectFit: 'cover',
                  borderRadius: 20,
                  marginBottom: 30
                }}
              />
            )}
            <p style={{
              fontSize: 36,
              color: '#fff',
              textAlign: 'center',
              maxWidth: '80%'
            }}>
              {point}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderArticleVideo(
  articleData: {
    title: string;
    points: string[];
    images: string[];
  },
  outputPath: string
): Promise<string> {
  const compositionId = 'ArticleVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: articleData,
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: articleData,
  });

  return outputPath;
}
```

### 4. Complete Pipeline Execution

Put it all together:

```typescript
// src/lib/pipeline/executor.ts
import { autoResearch } from '../research/crawler';
import { ContentGenerator } from '../ai/content-generator';
import { renderArticleVideo } from '../video/render';
import { autoPost } from '../social/publisher';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: ContentTone;
  platforms: ('facebook' | 'linkedin' | 'twitter')[];
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for: ${config.keyword}`);

  // Step 1: Research
  console.log('📡 Researching trending topics...');
  const researchData = await autoResearch(config.keyword, '24h');
  
  if (researchData.length === 0) {
    throw new Error('No research data found');
  }

  // Step 2: Generate Content
  console.log('🧠 Generating content with AI...');
  const generator = new ContentGenerator();
  const content = await generator.generateDualLanguage({
    format: config.format,
    tone: config.tone,
    researchData
  });

  // Step 3: Generate Video (if enabled)
  let videoPath: string | undefined;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    
    const points = content.en.split('\n')
      .filter(line => line.trim().startsWith('-'))
      .map(line => line.trim().substring(1).trim())
      .slice(0, 5);

    videoPath = await renderArticleVideo({
      title: config.keyword,
      points,
      images: researchData.slice(0, 5).map(r => r.url)
    }, `./output/${Date.now()}.mp4`);
  }

  // Step 4: Auto-post to platforms
  console.log('📤 Publishing to social media...');
  const results = await autoPost({
    content: content.en,
    videoPath,
    platforms: config.platforms
  });

  console.log('✅ Pipeline completed!');
  
  return {
    researchCount: researchData.length,
    content,
    videoPath,
    publishResults: results
  };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI marketing trends 2026',
  format: 'toplist',
  tone: 'expert',
  platforms: ['facebook', 'linkedin'],
  generateVideo: true
});
```

### 5. Social Media Auto-posting

```typescript
// src/lib/social/publisher.ts
import { FacebookAPI } from './platforms/facebook';
import { LinkedInAPI } from './platforms/linkedin';

interface PostConfig {
  content: string;
  videoPath?: string;
  platforms: ('facebook' | 'linkedin' | 'twitter')[];
}

export async function autoPost(config: PostConfig) {
  const results: Record<string, { success: boolean; postId?: string; error?: string }> = {};

  for (const platform of config.platforms) {
    try {
      let postId: string;

      switch (platform) {
        case 'facebook':
          const fb = new FacebookAPI(process.env.FACEBOOK_PAGE_ACCESS_TOKEN!);
          postId = await fb.post({
            message: config.content,
            videoPath: config.videoPath
          });
          break;

        case 'linkedin':
          const li = new LinkedInAPI(process.env.LINKEDIN_ACCESS_TOKEN!);
          postId = await li.post({
            text: config.content,
            videoPath: config.videoPath
          });
          break;

        default:
          throw new Error(`Platform ${platform} not supported`);
      }

      results[platform] = { success: true, postId };
    } catch (error) {
      results[platform] = { 
        success: false, 
        error: error instanceof Error ? error.message : 'Unknown error' 
      };
    }
  }

  return results;
}
```

## API Routes (Next.js)

```typescript
// src/app/api/pipeline/run/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/executor';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      tone: body.tone || 'expert',
      platforms: body.platforms || ['facebook'],
      generateVideo: body.generateVideo ?? true
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error instanceof Error ? error.message : 'Pipeline failed'
    }, { status: 500 });
  }
}
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
async function batchProcess(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      format: 'toplist',
      tone: 'expert',
      platforms: ['facebook', 'linkedin'],
      generateVideo: true
    });
    
    results.push(result);
    
    // Avoid rate limits
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
  
  return results;
}
```

### Scheduled Content Generation

```typescript
// src/lib/scheduler/cron.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/executor';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'marketing automation', 'social media'];
  
  for (const keyword of keywords) {
    await runContentPipeline({
      keyword,
      format: 'how-to',
      tone: 'friendly',
      platforms: ['facebook'],
      generateVideo: true
    });
  }
});
```

## Troubleshooting

### API Rate Limits
```typescript
// Implement exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues
```typescript
// Reduce composition quality for large videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  inputProps: articleData,
  // Add memory optimization
  chromiumOptions: {
    headless: true,
    args: ['--max-old-space-size=4096']
  },
  scale: 0.8 // Reduce to 80% if memory constrained
});
```

### Missing Environment Variables
```typescript
// src/lib/utils/env-check.ts
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

## Performance Tips

- Use streaming responses for large AI generations
- Cache research results for 1 hour to reduce API calls
- Queue video rendering jobs to avoid memory spikes
- Implement webhook callbacks for long-running processes
- Use CDN for video hosting to reduce server load
