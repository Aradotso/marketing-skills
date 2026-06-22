---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated posts
  - generate content from research to video automatically
  - create AI-driven content workflow with Claude and OpenAI
  - build automated marketing content pipeline
  - integrate Remotion for automatic video rendering
  - crawl news sources and generate social media content
  - automate research scriptwriting and posting workflow
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates multi-format content using Claude/OpenAI, and renders videos automatically using Remotion.

## What This Project Does

The marketing-pipeline-automation system provides:

- **Auto-Scan Research**: Crawls real-time data from major news sources within 24h
- **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
- **Multi-language Support**: Generates simultaneous Vietnamese and English versions
- **Video Rendering**: Automatically creates infographics and short videos from written content using Remotion
- **Social Media Integration**: Exports videos optimized for Reels, TikTok, and Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Model APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Content Settings
DEFAULT_LANGUAGE=en
ENABLE_MULTI_LANGUAGE=true
OUTPUT_VIDEO_FORMAT=mp4
VIDEO_QUALITY=high
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── crawlers/          # News source crawlers
│   ├── ai/                # AI content generation
│   ├── video/             # Remotion video rendering
│   ├── api/               # API routes
│   └── types/             # TypeScript definitions
├── pages/                 # Next.js pages
├── public/                # Static assets
└── remotion/              # Video templates
```

## Core Components

### 1. Research Crawler

```typescript
// src/crawlers/newsCrawler.ts
import { CrawlerConfig, NewsArticle } from '../types';

export class NewsCrawler {
  private sources: string[];
  private timeRange: number; // hours

  constructor(config: CrawlerConfig) {
    this.sources = config.sources || ['techcrunch', 'a16z', 'twitter'];
    this.timeRange = config.timeRange || 24;
  }

  async crawlAllSources(): Promise<NewsArticle[]> {
    const articles: NewsArticle[] = [];
    
    for (const source of this.sources) {
      const sourceArticles = await this.crawlSource(source);
      articles.push(...sourceArticles);
    }

    return this.filterByTimeRange(articles);
  }

  private async crawlSource(source: string): Promise<NewsArticle[]> {
    // Implementation using RapidAPI or custom scrapers
    const endpoint = this.getSourceEndpoint(source);
    
    const response = await fetch(endpoint, {
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        'X-RapidAPI-Host': this.getApiHost(source)
      }
    });

    return await response.json();
  }

  private filterByTimeRange(articles: NewsArticle[]): NewsArticle[] {
    const cutoffTime = Date.now() - (this.timeRange * 60 * 60 * 1000);
    return articles.filter(a => new Date(a.publishedAt).getTime() > cutoffTime);
  }

  private getSourceEndpoint(source: string): string {
    const endpoints = {
      techcrunch: 'https://techcrunch.com/wp-json/wp/v2/posts',
      a16z: 'https://a16z.com/wp-json/wp/v2/posts',
      twitter: 'https://twitter-api45.p.rapidapi.com/search'
    };
    return endpoints[source] || '';
  }

  private getApiHost(source: string): string {
    return `${source}-api.p.rapidapi.com`;
  }
}

// Usage
const crawler = new NewsCrawler({
  sources: ['techcrunch', 'a16z'],
  timeRange: 24
});

const articles = await crawler.crawlAllSources();
```

### 2. AI Content Generator

```typescript
// src/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';
import { ContentFormat, GeneratedContent } from '../types';

export class AIContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  private defaultModel: 'claude' | 'openai';

  constructor(defaultModel: 'claude' | 'openai' = 'claude') {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
    this.defaultModel = defaultModel;
  }

  async generateContent(
    research: string,
    format: ContentFormat,
    language: 'en' | 'vi',
    tone: 'expert' | 'friendly' | 'humorous' = 'expert'
  ): Promise<GeneratedContent> {
    const prompt = this.buildPrompt(research, format, language, tone);

    if (this.defaultModel === 'claude') {
      return await this.generateWithClaude(prompt);
    } else {
      return await this.generateWithOpenAI(prompt);
    }
  }

  private async generateWithClaude(prompt: string): Promise<GeneratedContent> {
    const message = await this.anthropic.messages.create({
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

    return this.parseContent(content);
  }

  private async generateWithOpenAI(prompt: string): Promise<GeneratedContent> {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
      max_tokens: 4096
    });

    const content = completion.choices[0].message.content || '';
    return this.parseContent(content);
  }

  private buildPrompt(
    research: string,
    format: ContentFormat,
    language: 'en' | 'vi',
    tone: string
  ): string {
    const formatInstructions = {
      toplist: 'Create a numbered list format with clear rankings',
      pov: 'Write from a personal perspective with strong opinions',
      casestudy: 'Structure as: Problem → Solution → Results with data',
      howto: 'Step-by-step tutorial format with actionable instructions'
    };

    const languageInstruction = language === 'vi' 
      ? 'Write in Vietnamese' 
      : 'Write in English';

    return `
You are a ${tone} content creator. Based on the following research data, create content in ${format} format.

${languageInstruction}.

Research Data:
${research}

Format Instructions:
${formatInstructions[format]}

Requirements:
- Include specific data points and statistics
- Add engaging headlines and subheadings
- Include 3-5 key takeaways
- Optimize for social media sharing
- Length: 800-1200 words

Output in JSON format:
{
  "title": "...",
  "subtitle": "...",
  "body": "...",
  "keyTakeaways": ["...", "..."],
  "hashtags": ["...", "..."],
  "videoScript": "..."
}
`;
  }

  private parseContent(rawContent: string): GeneratedContent {
    try {
      const jsonMatch = rawContent.match(/\{[\s\S]*\}/);
      if (jsonMatch) {
        return JSON.parse(jsonMatch[0]);
      }
    } catch (e) {
      // Fallback parsing
    }

    return {
      title: '',
      subtitle: '',
      body: rawContent,
      keyTakeaways: [],
      hashtags: [],
      videoScript: ''
    };
  }

  async generateMultiLanguage(
    research: string,
    format: ContentFormat
  ): Promise<{ en: GeneratedContent; vi: GeneratedContent }> {
    const [enContent, viContent] = await Promise.all([
      this.generateContent(research, format, 'en'),
      this.generateContent(research, format, 'vi')
    ]);

    return { en: enContent, vi: viContent };
  }
}

// Usage
const generator = new AIContentGenerator('claude');

const content = await generator.generateContent(
  researchData,
  'toplist',
  'en',
  'expert'
);

const multiLang = await generator.generateMultiLanguage(
  researchData,
  'casestudy'
);
```

### 3. Video Renderer (Remotion)

```typescript
// src/video/videoRenderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { GeneratedContent } from '../types';
import path from 'path';

export class VideoRenderer {
  private compositionId: string;
  private outputDir: string;

  constructor(outputDir: string = './output/videos') {
    this.compositionId = 'ContentVideo';
    this.outputDir = outputDir;
  }

  async renderVideo(
    content: GeneratedContent,
    platform: 'reels' | 'tiktok' | 'shorts' = 'reels'
  ): Promise<string> {
    // Bundle the Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride: (config) => config,
    });

    // Get composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: this.compositionId,
      inputProps: {
        content,
        platform
      }
    });

    // Render video
    const outputLocation = path.join(
      this.outputDir,
      `${Date.now()}-${platform}.mp4`
    );

    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps: {
        content,
        platform,
        dimensions: this.getPlatformDimensions(platform)
      }
    });

    return outputLocation;
  }

  private getPlatformDimensions(platform: string) {
    const dimensions = {
      reels: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 },
      shorts: { width: 1080, height: 1920 },
      landscape: { width: 1920, height: 1080 }
    };

    return dimensions[platform] || dimensions.reels;
  }

  async renderAllPlatforms(content: GeneratedContent): Promise<string[]> {
    const platforms: Array<'reels' | 'tiktok' | 'shorts'> = [
      'reels',
      'tiktok',
      'shorts'
    ];

    const videos = await Promise.all(
      platforms.map(platform => this.renderVideo(content, platform))
    );

    return videos;
  }
}

// Usage
const renderer = new VideoRenderer('./public/videos');
const videoPath = await renderer.renderVideo(generatedContent, 'reels');
const allVideos = await renderer.renderAllPlatforms(generatedContent);
```

### 4. Complete Pipeline

```typescript
// src/pipeline/contentPipeline.ts
import { NewsCrawler } from '../crawlers/newsCrawler';
import { AIContentGenerator } from '../ai/contentGenerator';
import { VideoRenderer } from '../video/videoRenderer';
import { ContentFormat } from '../types';

export class ContentPipeline {
  private crawler: NewsCrawler;
  private generator: AIContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.crawler = new NewsCrawler({
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: 24
    });
    this.generator = new AIContentGenerator('claude');
    this.renderer = new VideoRenderer();
  }

  async runFullPipeline(
    keyword: string,
    format: ContentFormat,
    generateVideo: boolean = true
  ) {
    console.log('🔍 Step 1: Crawling research data...');
    const articles = await this.crawler.crawlAllSources();
    const relevantArticles = this.filterByKeyword(articles, keyword);
    const research = this.aggregateResearch(relevantArticles);

    console.log('🧠 Step 2: Generating content with AI...');
    const content = await this.generator.generateMultiLanguage(
      research,
      format
    );

    console.log('💾 Step 3: Saving content...');
    const contentId = await this.saveContent(content);

    let videos: string[] = [];
    if (generateVideo) {
      console.log('🎬 Step 4: Rendering videos...');
      videos = await this.renderer.renderAllPlatforms(content.en);
    }

    console.log('✅ Pipeline complete!');
    return {
      contentId,
      content,
      videos,
      articleCount: relevantArticles.length
    };
  }

  private filterByKeyword(articles: any[], keyword: string) {
    return articles.filter(article => 
      article.title.toLowerCase().includes(keyword.toLowerCase()) ||
      article.content.toLowerCase().includes(keyword.toLowerCase())
    );
  }

  private aggregateResearch(articles: any[]): string {
    return articles.map(a => `
Title: ${a.title}
Source: ${a.source}
Date: ${a.publishedAt}
Summary: ${a.summary || a.content.substring(0, 300)}
---
    `).join('\n');
  }

  private async saveContent(content: any): Promise<string> {
    // Save to database or file system
    const id = `content_${Date.now()}`;
    // Implementation depends on your storage solution
    return id;
  }
}

// Usage
const pipeline = new ContentPipeline();

const result = await pipeline.runFullPipeline(
  'artificial intelligence',
  'toplist',
  true
);

console.log(`Created content: ${result.contentId}`);
console.log(`Generated ${result.videos.length} videos`);
```

## API Routes (Next.js)

```typescript
// pages/api/generate-content.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '../../src/pipeline/contentPipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, generateVideo } = req.body;

  if (!keyword || !format) {
    return res.status(400).json({ 
      error: 'Missing required fields: keyword, format' 
    });
  }

  try {
    const pipeline = new ContentPipeline();
    const result = await pipeline.runFullPipeline(
      keyword,
      format,
      generateVideo
    );

    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Failed to generate content',
      details: error.message 
    });
  }
}
```

## TypeScript Types

```typescript
// src/types/index.ts
export type ContentFormat = 'toplist' | 'pov' | 'casestudy' | 'howto';

export interface NewsArticle {
  title: string;
  source: string;
  url: string;
  publishedAt: string;
  content: string;
  summary?: string;
  author?: string;
}

export interface CrawlerConfig {
  sources: string[];
  timeRange: number; // hours
}

export interface GeneratedContent {
  title: string;
  subtitle: string;
  body: string;
  keyTakeaways: string[];
  hashtags: string[];
  videoScript: string;
}

export interface PipelineResult {
  contentId: string;
  content: {
    en: GeneratedContent;
    vi: GeneratedContent;
  };
  videos: string[];
  articleCount: number;
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run specific pipeline
npm run pipeline -- --keyword "AI" --format toplist
```

## Common Patterns

### Scheduling Content Generation

```typescript
// src/scheduler/contentScheduler.ts
import cron from 'node-cron';
import { ContentPipeline } from '../pipeline/contentPipeline';

export class ContentScheduler {
  private pipeline: ContentPipeline;
  private schedules: Map<string, cron.ScheduledTask>;

  constructor() {
    this.pipeline = new ContentPipeline();
    this.schedules = new Map();
  }

  scheduleDaily(keyword: string, format: ContentFormat, time: string) {
    const task = cron.schedule(time, async () => {
      console.log(`Running scheduled content generation for: ${keyword}`);
      await this.pipeline.runFullPipeline(keyword, format, true);
    });

    this.schedules.set(`${keyword}-${format}`, task);
  }

  stopSchedule(keyword: string, format: ContentFormat) {
    const key = `${keyword}-${format}`;
    const task = this.schedules.get(key);
    if (task) {
      task.stop();
      this.schedules.delete(key);
    }
  }
}

// Usage - run daily at 9 AM
const scheduler = new ContentScheduler();
scheduler.scheduleDaily('marketing trends', 'toplist', '0 9 * * *');
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];

  for (const keyword of keywords) {
    try {
      const result = await pipeline.runFullPipeline(
        keyword,
        'toplist',
        false // Skip video for batch processing
      );
      results.push({ keyword, success: true, result });
    } catch (error) {
      results.push({ keyword, success: false, error: error.message });
    }
  }

  return results;
}

// Usage
const keywords = ['AI trends', 'Marketing automation', 'Social media'];
const batchResults = await batchGenerateContent(keywords);
```

## Troubleshooting

### API Rate Limits

```typescript
// Add rate limiting
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10 // limit each IP to 10 requests per windowMs
});

// Apply to API routes
app.use('/api/', limiter);
```

### Memory Issues with Video Rendering

```typescript
// Render videos sequentially instead of parallel
async renderVideosSequentially(content: GeneratedContent) {
  const platforms = ['reels', 'tiktok', 'shorts'];
  const videos = [];

  for (const platform of platforms) {
    const video = await this.renderer.renderVideo(content, platform);
    videos.push(video);
    
    // Clean up memory between renders
    if (global.gc) {
      global.gc();
    }
  }

  return videos;
}
```

### Claude/OpenAI Timeout Errors

```typescript
// Add retry logic
async function generateWithRetry(
  generator: AIContentGenerator,
  research: string,
  format: ContentFormat,
  maxRetries: number = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generator.generateContent(research, format, 'en');
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

### Crawler Blocking

```typescript
// Add user agent rotation and delays
private async crawlWithDelay(source: string): Promise<NewsArticle[]> {
  const userAgents = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)...'
  ];

  await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay

  return await fetch(endpoint, {
    headers: {
      'User-Agent': userAgents[Math.floor(Math.random() * userAgents.length)]
    }
  });
}
```

This skill enables complete automation of marketing content creation from research through video generation, suitable for content creators, marketers, and businesses looking to scale their content production.
