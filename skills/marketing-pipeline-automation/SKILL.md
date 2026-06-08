---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline with Claude and OpenAI
  - generate videos from content automatically
  - crawl news and create content pipeline
  - use Remotion for automated video rendering
  - build AI content research and generation system
  - automate social media content with AI
  - create content pipeline from research to video
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates content creation from research through video generation. The pipeline crawls news sources, generates content in multiple formats using Claude/OpenAI, and renders videos automatically with Remotion.

## What This Project Does

The Marketing Pipeline automates:
- **Auto-scanning research**: Crawls TechCrunch, a16z, Twitter, LinkedIn for recent news
- **AI content generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language support**: Generates Vietnamese and English versions simultaneously
- **Video rendering**: Automatically creates infographics and short videos using Remotion
- **Social media optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research & Crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Content Settings
DEFAULT_LANGUAGE=en
SECONDARY_LANGUAGE=vi

# Video Rendering
REMOTION_BUNDLE_PATH=./out/bundle
VIDEO_OUTPUT_PATH=./output/videos
```

## Core Modules

### 1. Research & Crawling Module

```typescript
// src/research/crawler.ts
import { ResearchCrawler } from './research/crawler';

interface CrawlerConfig {
  sources: string[];
  timeRange: number; // hours
  maxResults: number;
}

const crawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: 24,
  maxResults: 50
});

// Crawl recent news
async function crawlNews(keyword: string) {
  const results = await crawler.scan({
    keyword,
    filters: {
      language: 'en',
      minEngagement: 100
    }
  });
  
  return results.map(item => ({
    title: item.title,
    url: item.url,
    excerpt: item.excerpt,
    publishedAt: item.publishedAt,
    source: item.source
  }));
}

// Usage
const aiNews = await crawlNews('artificial intelligence');
console.log(`Found ${aiNews.length} articles`);
```

### 2. AI Content Generation

```typescript
// src/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  targetAudience: string;
}

class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }

  async generateWithClaude(
    research: any[],
    config: ContentConfig
  ): Promise<string> {
    const prompt = this.buildPrompt(research, config);
    
    const message = await this.claude.messages.create({
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
    research: any[],
    config: ContentConfig
  ): Promise<string> {
    const prompt = this.buildPrompt(research, config);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'system',
        content: 'You are an expert content creator specializing in marketing content.'
      }, {
        role: 'user',
        content: prompt
      }],
      temperature: 0.7
    });

    return completion.choices[0].message.content || '';
  }

  private buildPrompt(research: any[], config: ContentConfig): string {
    const researchData = research.map(r => 
      `Title: ${r.title}\nSource: ${r.source}\nExcerpt: ${r.excerpt}`
    ).join('\n\n');

    return `
Based on the following research data, create a ${config.format} article in ${config.language}.

Tone: ${config.tone}
Target Audience: ${config.targetAudience}

Research Data:
${researchData}

Requirements:
- Create engaging, data-backed content
- Include specific examples and insights from the research
- Structure according to ${config.format} format
- Use ${config.tone} tone throughout
- Optimize for readability and engagement
    `.trim();
  }
}

// Usage example
const generator = new ContentGenerator();

const content = await generator.generateWithClaude(aiNews, {
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  targetAudience: 'Tech entrepreneurs and marketers'
});
```

### 3. Multi-Language Content Pipeline

```typescript
// src/pipeline/multilingual.ts
interface BilingualContent {
  english: string;
  vietnamese: string;
  metadata: {
    generatedAt: Date;
    format: string;
    wordCount: { en: number; vi: number };
  };
}

async function generateBilingualContent(
  research: any[],
  format: ContentConfig['format']
): Promise<BilingualContent> {
  const generator = new ContentGenerator();

  const [english, vietnamese] = await Promise.all([
    generator.generateWithClaude(research, {
      format,
      tone: 'expert',
      language: 'en',
      targetAudience: 'International tech audience'
    }),
    generator.generateWithClaude(research, {
      format,
      tone: 'friendly',
      language: 'vi',
      targetAudience: 'Vietnamese marketers and content creators'
    })
  ]);

  return {
    english,
    vietnamese,
    metadata: {
      generatedAt: new Date(),
      format,
      wordCount: {
        en: english.split(/\s+/).length,
        vi: vietnamese.split(/\s+/).length
      }
    }
  };
}
```

### 4. Video Rendering with Remotion

```typescript
// src/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  composition: string;
  inputProps: Record<string, any>;
  outputPath: string;
  format: 'mp4' | 'gif';
  dimensions: {
    width: number;
    height: number;
  };
}

class VideoRenderer {
  private bundleLocation: string | null = null;

  async initialize() {
    // Bundle the Remotion project
    this.bundleLocation = await bundle({
      entryPoint: path.resolve('./src/video/index.ts'),
      webpackOverride: (config) => config
    });
  }

  async renderVideo(config: VideoConfig): Promise<string> {
    if (!this.bundleLocation) {
      await this.initialize();
    }

    const composition = await selectComposition({
      serveUrl: this.bundleLocation!,
      id: config.composition,
      inputProps: config.inputProps
    });

    await renderMedia({
      composition,
      serveUrl: this.bundleLocation!,
      codec: 'h264',
      outputLocation: config.outputPath,
      inputProps: config.inputProps,
      chromiumOptions: {
        headless: true
      }
    });

    return config.outputPath;
  }
}

// Video composition component
// src/video/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points, duration }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={30}>
        <div style={{
          fontSize: 60,
          color: 'white',
          textAlign: 'center',
          padding: 40
        }}>
          {title}
        </div>
      </Sequence>

      {points.map((point, index) => (
        <Sequence
          key={index}
          from={30 + (index * 60)}
          durationInFrames={60}
        >
          <div style={{
            fontSize: 40,
            color: 'white',
            padding: 40
          }}>
            {index + 1}. {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// src/pipeline/orchestrator.ts
import { ResearchCrawler } from '../research/crawler';
import { ContentGenerator } from '../ai/content-generator';
import { VideoRenderer } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

class ContentPipeline {
  private crawler: ResearchCrawler;
  private generator: ContentGenerator;
  private videoRenderer: VideoRenderer;

  constructor() {
    this.crawler = new ResearchCrawler({
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeRange: 24,
      maxResults: 50
    });
    this.generator = new ContentGenerator();
    this.videoRenderer = new VideoRenderer();
  }

  async run(config: PipelineConfig) {
    console.log(`Starting pipeline for keyword: ${config.keyword}`);

    // Step 1: Research
    console.log('Step 1: Crawling research data...');
    const research = await this.crawler.scan({
      keyword: config.keyword,
      filters: { language: 'en', minEngagement: 100 }
    });
    console.log(`Found ${research.length} relevant articles`);

    // Step 2: Generate content
    console.log('Step 2: Generating content...');
    const content: Record<string, string> = {};
    
    for (const lang of config.languages) {
      content[lang] = await this.generator.generateWithClaude(research, {
        format: config.format,
        tone: 'expert',
        language: lang,
        targetAudience: lang === 'en' 
          ? 'International marketers' 
          : 'Vietnamese content creators'
      });
    }

    // Step 3: Generate video (if enabled)
    let videoPath: string | null = null;
    if (config.generateVideo) {
      console.log('Step 3: Rendering video...');
      
      const dimensions = this.getVideoDimensions(config.videoFormat);
      const contentPoints = this.extractKeyPoints(content[config.languages[0]]);

      videoPath = await this.videoRenderer.renderVideo({
        composition: 'ContentVideo',
        inputProps: {
          title: config.keyword,
          points: contentPoints,
          duration: 30
        },
        outputPath: `./output/${config.keyword.replace(/\s+/g, '-')}.mp4`,
        format: 'mp4',
        dimensions
      });
    }

    return {
      research: research.slice(0, 5), // Top 5 sources
      content,
      videoPath,
      metadata: {
        keyword: config.keyword,
        format: config.format,
        generatedAt: new Date(),
        wordCounts: Object.entries(content).reduce((acc, [lang, text]) => {
          acc[lang] = text.split(/\s+/).length;
          return acc;
        }, {} as Record<string, number>)
      }
    };
  }

  private getVideoDimensions(format?: string) {
    switch (format) {
      case 'reels':
      case 'tiktok':
      case 'shorts':
        return { width: 1080, height: 1920 }; // 9:16
      default:
        return { width: 1920, height: 1080 }; // 16:9
    }
  }

  private extractKeyPoints(content: string): string[] {
    // Simple extraction - could be enhanced with AI
    const lines = content.split('\n').filter(line => 
      line.trim().match(/^[\d\-\*]\.?\s+/)
    );
    return lines.slice(0, 5).map(line => 
      line.replace(/^[\d\-\*]\.?\s+/, '').trim()
    );
  }
}

// Usage
const pipeline = new ContentPipeline();

const result = await pipeline.run({
  keyword: 'AI content marketing trends 2024',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormat: 'reels'
});

console.log('Pipeline completed:', result.metadata);
```

## API Routes (Next.js)

```typescript
// pages/api/pipeline/run.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '../../../src/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, format, languages, generateVideo, videoFormat } = req.body;

    const pipeline = new ContentPipeline();
    const result = await pipeline.run({
      keyword,
      format,
      languages,
      generateVideo,
      videoFormat
    });

    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline execution failed',
      message: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

## Common Patterns

### Pattern 1: Quick Content Generation

```typescript
// Quick single-language content
import { ContentPipeline } from './src/pipeline/orchestrator';

const pipeline = new ContentPipeline();

const quickContent = await pipeline.run({
  keyword: 'marketing automation',
  format: 'how-to',
  languages: ['en'],
  generateVideo: false
});

console.log(quickContent.content.en);
```

### Pattern 2: Full Multi-Platform Export

```typescript
// Generate content + videos for all platforms
const formats = ['reels', 'tiktok', 'shorts'] as const;

for (const format of formats) {
  const result = await pipeline.run({
    keyword: 'social media strategy',
    format: 'toplist',
    languages: ['en', 'vi'],
    generateVideo: true,
    videoFormat: format
  });
  
  console.log(`${format} video generated at:`, result.videoPath);
}
```

### Pattern 3: Scheduled Content Generation

```typescript
// Use with cron or scheduler
import cron from 'node-cron';

const keywords = [
  'AI marketing tools',
  'content automation',
  'social media trends'
];

// Run daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  const today = new Date().toISOString().split('T')[0];
  const keyword = keywords[new Date().getDay() % keywords.length];
  
  const result = await pipeline.run({
    keyword,
    format: 'toplist',
    languages: ['en', 'vi'],
    generateVideo: true,
    videoFormat: 'reels'
  });
  
  // Save to database or file system
  await saveContent(`content-${today}.json`, result);
});
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function generateWithRetry(
  generator: ContentGenerator,
  research: any[],
  config: ContentConfig,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generator.generateWithClaude(research, config);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

### Issue: Video Rendering Memory Errors

```typescript
// Use concurrency limits
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent renders

const videos = await Promise.all(
  configs.map(config => 
    limit(() => videoRenderer.renderVideo(config))
  )
);
```

### Issue: Empty Research Results

```typescript
// Fallback to multiple sources
async function robustResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  
  for (const source of sources) {
    const results = await crawler.scan({
      keyword,
      filters: { source }
    });
    
    if (results.length > 0) {
      return results;
    }
  }
  
  throw new Error('No research data found for keyword');
}
```

### Issue: Content Quality Validation

```typescript
// Validate generated content
function validateContent(content: string): boolean {
  return (
    content.length > 500 && // Minimum length
    content.split('\n\n').length > 3 && // Has paragraphs
    !content.includes('[ERROR]') && // No error markers
    !content.includes('I cannot') // Not a refusal
  );
}

let content = await generator.generateWithClaude(research, config);
if (!validateContent(content)) {
  console.log('Content validation failed, regenerating...');
  content = await generator.generateWithOpenAI(research, config);
}
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const researchCache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function getCachedResearch(keyword: string) {
  const cached = researchCache.get<any[]>(keyword);
  if (cached) {
    return cached;
  }
  
  const results = await crawler.scan({ keyword });
  researchCache.set(keyword, results);
  return results;
}
```

This skill provides comprehensive coverage of the Marketing Pipeline automation system, enabling AI coding agents to effectively assist developers in implementing automated content creation, multi-language generation, and video rendering workflows.
