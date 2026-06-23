---
name: marketing-pipeline-content-automation
description: AI-powered content pipeline that auto-researches, generates scripts, and creates videos from keywords using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline automation
  - generate content with automatic research and video rendering
  - create marketing content from keywords using AI
  - automate content creation with Claude and OpenAI
  - build AI-powered content generation workflow
  - use Remotion to render videos from generated content
  - set up auto-research content pipeline with multi-language support
  - configure AI content automation with TechCrunch and Twitter scraping
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**Marketing Pipeline Share** is an end-to-end AI content automation system that transforms a single keyword into complete content assets including researched articles, multi-language scripts, and rendered videos. The pipeline combines web scraping (TechCrunch, a16z, Twitter, LinkedIn), AI content generation (Claude 3, OpenAI), and video rendering (Remotion) into a unified workflow.

**Key capabilities:**
- Auto-research from live sources (24h news aggregation)
- Multi-format content generation (Top lists, POV, Case Studies, How-to)
- Dual-language output (English + Vietnamese)
- Automatic video/infographic rendering for social platforms
- Customizable tone and target audience

## Installation

### Prerequisites

```bash
# Node.js 18+ and npm/yarn required
node --version  # Should be 18.0.0 or higher
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create `.env` file with the following variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer

# Database (if applicable)
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Core Architecture

### Pipeline Flow

```typescript
// src/lib/pipeline/contentPipeline.ts
import { ResearchEngine } from './research/ResearchEngine';
import { ContentGenerator } from './generator/ContentGenerator';
import { VideoRenderer } from './video/VideoRenderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone: 'expert' | 'friendly' | 'humorous';
  videoEnabled: boolean;
}

class ContentPipeline {
  private research: ResearchEngine;
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.research = new ResearchEngine();
    this.generator = new ContentGenerator();
    this.renderer = new VideoRenderer();
  }

  async execute(config: PipelineConfig) {
    // Step 1: Research phase
    const researchData = await this.research.gather({
      keyword: config.keyword,
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeframe: '24h'
    });

    // Step 2: Content generation
    const content = await this.generator.create({
      research: researchData,
      format: config.format,
      languages: config.languages,
      tone: config.tone
    });

    // Step 3: Video rendering (if enabled)
    let videoUrls = null;
    if (config.videoEnabled) {
      videoUrls = await this.renderer.generate({
        content: content,
        platforms: ['reels', 'tiktok', 'shorts']
      });
    }

    return {
      content,
      videoUrls,
      metadata: {
        keyword: config.keyword,
        generatedAt: new Date(),
        sources: researchData.sources
      }
    };
  }
}

export default ContentPipeline;
```

## Research Engine

### Auto-Scraping Implementation

```typescript
// src/lib/research/ResearchEngine.ts
import axios from 'axios';

interface ResearchResult {
  sources: SourceData[];
  insights: string[];
  statistics: Record<string, any>;
}

export class ResearchEngine {
  private rapidApiKey: string;

  constructor() {
    this.rapidApiKey = process.env.RAPIDAPI_KEY!;
  }

  async gather(params: {
    keyword: string;
    sources: string[];
    timeframe: string;
  }): Promise<ResearchResult> {
    const results = await Promise.all([
      this.scrapeTechCrunch(params.keyword),
      this.scrapeTwitter(params.keyword),
      this.scrapeLinkedIn(params.keyword)
    ]);

    return this.aggregateResults(results);
  }

  private async scrapeTechCrunch(keyword: string) {
    const response = await axios.get('https://techcrunch-api.p.rapidapi.com/search', {
      params: { query: keyword, limit: 10 },
      headers: {
        'X-RapidAPI-Key': this.rapidApiKey,
        'X-RapidAPI-Host': 'techcrunch-api.p.rapidapi.com'
      }
    });

    return response.data.articles.map((article: any) => ({
      title: article.title,
      url: article.url,
      summary: article.summary,
      publishedAt: article.published_at,
      source: 'techcrunch'
    }));
  }

  private async scrapeTwitter(keyword: string) {
    const response = await axios.get('https://twitter-api45.p.rapidapi.com/search', {
      params: { query: keyword, count: 20 },
      headers: {
        'X-RapidAPI-Key': this.rapidApiKey,
        'Authorization': `Bearer ${process.env.TWITTER_BEARER_TOKEN}`
      }
    });

    return response.data.tweets;
  }

  private aggregateResults(results: any[]): ResearchResult {
    const allSources = results.flat();
    const insights = this.extractInsights(allSources);
    const statistics = this.extractStatistics(allSources);

    return { sources: allSources, insights, statistics };
  }

  private extractInsights(sources: any[]): string[] {
    // AI-powered insight extraction logic
    return sources
      .map(s => s.summary || s.text)
      .filter(Boolean)
      .slice(0, 5);
  }

  private extractStatistics(sources: any[]): Record<string, any> {
    return {
      totalSources: sources.length,
      sentiment: this.analyzeSentiment(sources),
      trendingTopics: this.extractTrends(sources)
    };
  }
}
```

## Content Generation

### Multi-Format Generator

```typescript
// src/lib/generator/ContentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface GeneratorConfig {
  research: ResearchResult;
  format: string;
  languages: string[];
  tone: string;
}

export class ContentGenerator {
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

  async create(config: GeneratorConfig) {
    const prompts = this.buildPrompts(config);
    
    const contents = await Promise.all(
      config.languages.map(lang => 
        this.generateForLanguage(prompts[lang], lang, config)
      )
    );

    return contents;
  }

  private buildPrompts(config: GeneratorConfig) {
    const { research, format, tone } = config;
    
    const baseContext = `
Based on the following research data:
${JSON.stringify(research.insights, null, 2)}

Statistics: ${JSON.stringify(research.statistics, null, 2)}
`;

    return {
      en: `${baseContext}\n\nCreate a ${format} article in English with a ${tone} tone.`,
      vi: `${baseContext}\n\nTạo bài viết dạng ${format} bằng Tiếng Việt với giọng văn ${tone}.`
    };
  }

  private async generateForLanguage(
    prompt: string, 
    language: string, 
    config: GeneratorConfig
  ) {
    // Use Claude for long-form content
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

    return {
      language,
      format: config.format,
      content,
      metadata: {
        model: 'claude-3-5-sonnet',
        tokensUsed: message.usage.output_tokens
      }
    };
  }

  async enhanceWithOpenAI(content: string, task: string) {
    // Use OpenAI for refinement tasks
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: 'You are a content enhancement expert.' },
        { role: 'user', content: `${task}\n\nContent:\n${content}` }
      ]
    });

    return completion.choices[0].message.content;
  }
}
```

## Video Rendering with Remotion

### Automatic Video Generation

```typescript
// src/lib/video/VideoRenderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderConfig {
  content: any;
  platforms: string[];
}

export class VideoRenderer {
  async generate(config: RenderConfig) {
    const compositions = config.platforms.map(platform => ({
      platform,
      aspectRatio: this.getAspectRatio(platform),
      duration: this.calculateDuration(config.content)
    }));

    const results = await Promise.all(
      compositions.map(comp => this.renderForPlatform(comp, config.content))
    );

    return results;
  }

  private async renderForPlatform(composition: any, content: any) {
    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
      webpackOverride: (config) => config
    });

    // Select composition
    const compositionId = `ContentVideo-${composition.platform}`;
    const comp = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: {
        content: content.content,
        title: content.title || 'Generated Content',
        aspectRatio: composition.aspectRatio
      }
    });

    // Render video
    const outputPath = path.join(
      process.cwd(), 
      'public', 
      'videos',
      `${Date.now()}-${composition.platform}.mp4`
    );

    await renderMedia({
      composition: comp,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: comp.props
    });

    return {
      platform: composition.platform,
      url: outputPath.replace(process.cwd() + '/public', ''),
      duration: comp.durationInFrames / comp.fps
    };
  }

  private getAspectRatio(platform: string): [number, number] {
    const ratios: Record<string, [number, number]> = {
      'reels': [9, 16],
      'tiktok': [9, 16],
      'shorts': [9, 16],
      'youtube': [16, 9],
      'linkedin': [1, 1]
    };
    return ratios[platform] || [16, 9];
  }

  private calculateDuration(content: any): number {
    const wordCount = content.content.split(' ').length;
    const wordsPerMinute = 150;
    return Math.ceil((wordCount / wordsPerMinute) * 60); // in seconds
  }
}
```

## API Usage Examples

### Next.js API Route

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import ContentPipeline from '@/lib/pipeline/contentPipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, languages, tone, videoEnabled } = req.body;

  try {
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute({
      keyword,
      format: format || 'toplist',
      languages: languages || ['en', 'vi'],
      tone: tone || 'expert',
      videoEnabled: videoEnabled ?? false
    });

    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error: any) {
    console.error('Pipeline error:', error);
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
}
```

### Frontend Component

```typescript
// components/ContentGenerator.tsx
import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const generateContent = async (formData: any) => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });

      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    
    generateContent({
      keyword: formData.get('keyword'),
      format: formData.get('format'),
      languages: ['en', 'vi'],
      tone: formData.get('tone'),
      videoEnabled: formData.get('video') === 'on'
    });
  };

  return (
    <div className="content-generator">
      <form onSubmit={handleSubmit}>
        <input name="keyword" placeholder="Enter keyword..." required />
        <select name="format">
          <option value="toplist">Top List</option>
          <option value="pov">POV Article</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        <select name="tone">
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>
        <label>
          <input type="checkbox" name="video" />
          Generate Video
        </label>
        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="results">
          {result.content.map((item: any, idx: number) => (
            <div key={idx} className="content-item">
              <h3>{item.language.toUpperCase()}</h3>
              <div dangerouslySetInnerHTML={{ __html: item.content }} />
            </div>
          ))}
          
          {result.videoUrls && (
            <div className="videos">
              <h3>Generated Videos</h3>
              {result.videoUrls.map((video: any, idx: number) => (
                <video key={idx} src={video.url} controls />
              ))}
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// src/scripts/batchGenerate.ts
import ContentPipeline from '@/lib/pipeline/contentPipeline';

async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];

  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    const result = await pipeline.execute({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'expert',
      videoEnabled: true
    });

    results.push({ keyword, ...result });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }

  return results;
}

// Usage
const keywords = ['AI automation', 'Content marketing', 'Video generation'];
batchGenerate(keywords).then(results => {
  console.log(`Generated ${results.length} content pieces`);
});
```

### Custom Research Sources

```typescript
// Extend ResearchEngine with custom sources
class CustomResearchEngine extends ResearchEngine {
  async scrapeCustomSource(keyword: string) {
    // Your custom scraping logic
    const response = await fetch(`https://your-api.com/search?q=${keyword}`);
    const data = await response.json();
    
    return data.results.map((item: any) => ({
      title: item.title,
      url: item.link,
      summary: item.snippet,
      source: 'custom'
    }));
  }

  async gather(params: any): Promise<ResearchResult> {
    const baseResults = await super.gather(params);
    const customResults = await this.scrapeCustomSource(params.keyword);
    
    return {
      ...baseResults,
      sources: [...baseResults.sources, ...customResults]
    };
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rateLimiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;

  constructor(requestsPerMinute: number) {
    this.delay = 60000 / requestsPerMinute;
  }

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

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;

    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) {
        await fn();
        await new Promise(resolve => setTimeout(resolve, this.delay));
      }
    }

    this.processing = false;
  }
}

// Usage in ResearchEngine
const limiter = new RateLimiter(10); // 10 requests per minute

await limiter.add(() => this.scrapeTechCrunch(keyword));
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering for large content
const renderConfig = {
  composition: comp,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  inputProps: comp.props,
  chromiumOptions: {
    args: ['--disable-dev-shm-usage', '--no-sandbox']
  },
  envVariables: {
    NODE_OPTIONS: '--max-old-space-size=4096'
  }
};
```

### OpenAI/Claude Token Limits

```typescript
// Split large content for processing
function splitContent(content: string, maxTokens: number = 3000): string[] {
  const words = content.split(' ');
  const chunks: string[] = [];
  let currentChunk: string[] = [];

  for (const word of words) {
    currentChunk.push(word);
    if (currentChunk.length >= maxTokens / 4) { // Rough token estimate
      chunks.push(currentChunk.join(' '));
      currentChunk = [];
    }
  }

  if (currentChunk.length > 0) {
    chunks.push(currentChunk.join(' '));
  }

  return chunks;
}
```

## Configuration Best Practices

### Environment Variables Structure

```bash
# .env.production
OPENAI_API_KEY=sk-proj-...
ANTHROPIC_API_KEY=sk-ant-...
RAPIDAPI_KEY=...

# Rate limits
MAX_REQUESTS_PER_MINUTE=10
BATCH_SIZE=5

# Rendering
REMOTION_CONCURRENCY=2
VIDEO_QUALITY=high

# Caching
REDIS_URL=redis://localhost:6379
CACHE_TTL=3600
```

### TypeScript Configuration

```json
// tsconfig.json additions
{
  "compilerOptions": {
    "paths": {
      "@/lib/*": ["src/lib/*"],
      "@/components/*": ["components/*"],
      "@/remotion/*": ["src/remotion/*"]
    }
  }
}
```

This skill enables AI agents to effectively use the Marketing Pipeline Share project for automated content generation, from research through video rendering.
