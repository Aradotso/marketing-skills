---
name: marketing-pipeline-content-automation
description: Automated AI content pipeline for research, script generation, video creation, and social media posting using Claude, OpenAI, and Remotion
triggers:
  - "how do I automate content creation with AI"
  - "generate marketing videos automatically"
  - "crawl news and create content from it"
  - "set up content pipeline with Claude and OpenAI"
  - "render videos from text using Remotion"
  - "automate social media content generation"
  - "create multilingual content with AI"
  - "build automated marketing workflow"
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation. The pipeline crawls news sources, generates multilingual content in various formats, and produces social media videos using AI.

## What This Project Does

The Marketing Pipeline automates:
- **Auto-Research**: Crawls TechCrunch, a16z, X (Twitter), LinkedIn for latest news (24h)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual Output**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Converts text content to videos/infographics using Remotion
- **Social Media Optimization**: Exports videos in formats optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites
```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
```

### Install Dependencies
```bash
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration
Create a `.env.local` file in the root directory:

```env
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for News Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Optional: Social Media APIs
FACEBOOK_PAGE_ACCESS_TOKEN=your_token
TWITTER_API_KEY=your_key
```

## Key Components & Architecture

### 1. Research Module (News Crawling)

```typescript
// src/lib/research/crawler.ts
import { RapidAPI } from './api-clients';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}

export async function crawlLatestNews(
  keywords: string[],
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const rapidAPI = new RapidAPI(process.env.RAPIDAPI_KEY!);
  
  const articles = await rapidAPI.searchNews({
    q: keywords.join(' OR '),
    sources: sources.join(','),
    from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString(),
    sortBy: 'publishedAt',
    language: 'en'
  });
  
  return articles.map(article => ({
    title: article.title,
    url: article.url,
    publishedAt: article.publishedAt,
    source: article.source.name,
    content: article.content || article.description
  }));
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: NewsArticle[];
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
  
  async generateWithClaude(request: ContentRequest): Promise<string> {
    const systemPrompt = this.buildSystemPrompt(request);
    const userPrompt = this.buildUserPrompt(request);
    
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      system: systemPrompt,
      messages: [
        {
          role: 'user',
          content: userPrompt
        }
      ]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  async generateWithOpenAI(request: ContentRequest): Promise<string> {
    const systemPrompt = this.buildSystemPrompt(request);
    const userPrompt = this.buildUserPrompt(request);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt }
      ],
      max_tokens: 4096,
      temperature: 0.7
    });
    
    return completion.choices[0].message.content || '';
  }
  
  private buildSystemPrompt(request: ContentRequest): string {
    const toneMap = {
      expert: 'professional and authoritative',
      friendly: 'conversational and approachable',
      humorous: 'witty and entertaining'
    };
    
    const formatMap = {
      'toplist': 'numbered list format with clear headings',
      'pov': 'opinion piece with strong perspective',
      'case-study': 'detailed analysis with data and examples',
      'how-to': 'step-by-step tutorial format'
    };
    
    return `You are an expert content creator specializing in ${request.format} articles.
Write in a ${toneMap[request.tone]} tone.
Use the ${formatMap[request.format]} structure.
Language: ${request.language === 'vi' ? 'Vietnamese' : 'English'}
Always cite sources and include data-backed insights.`;
  }
  
  private buildUserPrompt(request: ContentRequest): string {
    const researchSummary = request.researchData
      .map(article => `- ${article.title} (${article.source}): ${article.content.slice(0, 200)}...`)
      .join('\n');
    
    return `Topic: ${request.topic}

Recent Research Data:
${researchSummary}

Create a comprehensive ${request.format} article about this topic using the research data provided.
Include specific examples, statistics, and actionable insights.`;
  }
}
```

### 3. Video Generation with Remotion

```typescript
// src/video/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

export interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor: string;
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  brandColor,
  duration
}) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{
          flex: 1,
          justifyContent: 'center',
          alignItems: 'center',
          display: 'flex',
          flexDirection: 'column',
          opacity: titleOpacity
        }}>
          <h1 style={{
            color: brandColor,
            fontSize: 80,
            textAlign: 'center',
            padding: '0 100px'
          }}>
            {title}
          </h1>
        </div>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <PointSlide point={point} index={index + 1} color={brandColor} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ point: string; index: number; color: string }> = ({
  point,
  index,
  color
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 20], [0, 1]);
  const scale = interpolate(frame, [0, 20], [0.8, 1]);
  
  return (
    <AbsoluteFill style={{
      justifyContent: 'center',
      alignItems: 'center',
      opacity,
      transform: `scale(${scale})`
    }}>
      <div style={{ padding: '0 100px', maxWidth: 1200 }}>
        <h2 style={{ color, fontSize: 60, marginBottom: 40 }}>
          {index}. {point.split(':')[0]}
        </h2>
        <p style={{ color: '#fff', fontSize: 40, lineHeight: 1.5 }}>
          {point.split(':').slice(1).join(':')}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// src/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  outputPath: string,
  props: ContentVideoProps
): Promise<string> {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/video/index.ts'),
    () => undefined,
    { webpackOverride: (config) => config }
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: props
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: props
  });
  
  return outputPath;
}
```

### 4. Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { crawlLatestNews } from '../research/crawler';
import { ContentGenerator } from '../ai/content-generator';
import { renderContentVideo } from '../../video/render';
import path from 'path';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
}

export class ContentPipeline {
  private generator: ContentGenerator;
  
  constructor() {
    this.generator = new ContentGenerator();
  }
  
  async run(config: PipelineConfig) {
    console.log(`Starting pipeline for keyword: ${config.keyword}`);
    
    // Step 1: Research
    console.log('Crawling latest news...');
    const articles = await crawlLatestNews([config.keyword]);
    console.log(`Found ${articles.length} articles`);
    
    // Step 2: Generate content for each language
    const contents: Record<string, string> = {};
    
    for (const lang of config.languages) {
      console.log(`Generating ${lang} content...`);
      const content = await this.generator.generateWithClaude({
        topic: config.keyword,
        format: config.format,
        tone: config.tone,
        language: lang,
        researchData: articles
      });
      contents[lang] = content;
    }
    
    // Step 3: Generate video if requested
    let videoPath: string | null = null;
    if (config.generateVideo) {
      console.log('Rendering video...');
      const points = this.extractKeyPoints(contents['en'] || contents['vi']);
      videoPath = await renderContentVideo(
        path.join(process.cwd(), 'output', `${config.keyword}-${Date.now()}.mp4`),
        {
          title: config.keyword,
          points,
          brandColor: '#00ff88',
          duration: 300
        }
      );
      console.log(`Video rendered: ${videoPath}`);
    }
    
    return {
      contents,
      videoPath,
      articles
    };
  }
  
  private extractKeyPoints(content: string): string[] {
    // Extract numbered points or major sections
    const matches = content.match(/(?:^|\n)(?:\d+\.|#{1,3})\s+(.+?)(?:\n|$)/g);
    return matches 
      ? matches.slice(0, 5).map(m => m.trim().replace(/^\d+\.\s*/, ''))
      : [];
  }
}
```

## CLI Usage

```typescript
// src/cli/index.ts
import { Command } from 'commander';
import { ContentPipeline } from '../lib/pipeline/orchestrator';

const program = new Command();

program
  .name('content-pipeline')
  .description('AI-powered content automation pipeline')
  .version('1.0.0');

program
  .command('generate')
  .description('Generate content from keyword')
  .requiredOption('-k, --keyword <keyword>', 'Content keyword')
  .option('-f, --format <format>', 'Content format', 'toplist')
  .option('-t, --tone <tone>', 'Writing tone', 'expert')
  .option('-l, --languages <languages>', 'Languages (comma-separated)', 'en,vi')
  .option('--video', 'Generate video', false)
  .action(async (options) => {
    const pipeline = new ContentPipeline();
    
    const result = await pipeline.run({
      keyword: options.keyword,
      format: options.format,
      tone: options.tone,
      languages: options.languages.split(','),
      generateVideo: options.video
    });
    
    console.log('Pipeline completed!');
    console.log('Contents generated:', Object.keys(result.contents));
    if (result.videoPath) {
      console.log('Video:', result.videoPath);
    }
  });

program.parse();
```

Run via command line:
```bash
# Generate content with video
npm run cli generate -k "AI automation" --format toplist --video

# Generate multilingual content
npm run cli generate -k "Marketing trends 2024" --languages en,vi --tone friendly

# Generate case study without video
npm run cli generate -k "SaaS growth" --format case-study
```

## API Routes (Next.js)

```typescript
// pages/api/pipeline/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '../../../src/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, tone, languages, generateVideo } = req.body;
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.run({
      keyword,
      format: format || 'toplist',
      tone: tone || 'expert',
      languages: languages || ['en'],
      generateVideo: generateVideo || false
    });
    
    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

## Common Patterns

### Pattern 1: Batch Content Generation
```typescript
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];
  
  for (const keyword of keywords) {
    const result = await pipeline.run({
      keyword,
      format: 'toplist',
      tone: 'expert',
      languages: ['en', 'vi'],
      generateVideo: true
    });
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
  
  return results;
}
```

### Pattern 2: Custom AI Model Selection
```typescript
const generator = new ContentGenerator();

// Use Claude for creative content
const creativeContent = await generator.generateWithClaude({
  topic: 'Future of AI',
  format: 'pov',
  tone: 'humorous',
  language: 'en',
  researchData: articles
});

// Use OpenAI for structured content
const structuredContent = await generator.generateWithOpenAI({
  topic: 'SEO Best Practices',
  format: 'how-to',
  tone: 'expert',
  language: 'en',
  researchData: articles
});
```

### Pattern 3: Social Media Publishing
```typescript
import { FacebookAPI, TwitterAPI } from './social-apis';

async function publishContent(content: string, videoPath: string) {
  const fb = new FacebookAPI(process.env.FACEBOOK_PAGE_ACCESS_TOKEN!);
  const twitter = new TwitterAPI(process.env.TWITTER_API_KEY!);
  
  // Post to Facebook
  await fb.postVideo({
    pageId: 'your-page-id',
    videoPath,
    description: content.slice(0, 500)
  });
  
  // Post to Twitter
  await twitter.postTweet({
    text: content.slice(0, 280),
    media: videoPath
  });
}
```

## Troubleshooting

### Issue: API Rate Limits
```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, waiting ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Issue: Video Rendering Memory Issues
```typescript
// Use composition chunks for long videos
export async function renderLongVideo(props: ContentVideoProps) {
  const chunkSize = 10; // 10 seconds per chunk
  const chunks = Math.ceil(props.duration / chunkSize);
  
  for (let i = 0; i < chunks; i++) {
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: `chunk-${i}.mp4`,
      frameRange: [i * chunkSize * 30, (i + 1) * chunkSize * 30]
    });
  }
  
  // Merge chunks using ffmpeg
  // ...
}
```

### Issue: News Crawling Failures
```typescript
// Fallback to multiple sources
async function crawlWithFallback(keywords: string[]) {
  const sources = [
    ['techcrunch', 'theverge'],
    ['a16z', 'ycombinator'],
    ['producthunt', 'indiehackers']
  ];
  
  for (const sourceGroup of sources) {
    try {
      const articles = await crawlLatestNews(keywords, sourceGroup);
      if (articles.length > 0) return articles;
    } catch (error) {
      console.error(`Failed to crawl ${sourceGroup}:`, error);
    }
  }
  
  throw new Error('All news sources failed');
}
```

## Development Setup

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Run tests
npm test

# Render video locally
npm run remotion:render
```

This skill enables comprehensive automation of content marketing workflows using cutting-edge AI and video rendering technologies.
