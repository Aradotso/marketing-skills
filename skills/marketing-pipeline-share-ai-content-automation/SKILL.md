---
name: marketing-pipeline-share-ai-content-automation
description: Automated content creation pipeline with AI research, script generation, multilingual content, and video rendering using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - create multilingual marketing content automatically
  - generate videos from written content using Remotion
  - build an AI content pipeline with Claude and OpenAI
  - scrape news and generate content scripts automatically
  - set up automated social media content generation
  - create TikTok and Reels videos from AI-generated scripts
  - build a complete content automation workflow
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow: from news research and script generation to multilingual content production and video rendering.

## What It Does

The Marketing Pipeline Share automates:
- **Auto-Research**: Crawls and analyzes recent news from TechCrunch, a16z, X (Twitter), LinkedIn
- **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Multilingual Output**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Converts written content into videos using Remotion for Reels, TikTok, Shorts
- **Multiple Content Formats**: Infographics, video shorts, and social media posts

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research API (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # News crawling & research
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core Components

### 1. Research/News Crawler

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  selector: string;
}

interface ResearchResult {
  title: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}

export async function crawlNews(
  keyword: string,
  sources: NewsSource[]
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];

  for (const source of sources) {
    try {
      const response = await axios.get(source.url, {
        params: { q: keyword, from: 'last24h' },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'X-RapidAPI-Host': 'news-api.rapidapi.com'
        }
      });

      const articles = response.data.articles || [];
      
      for (const article of articles) {
        results.push({
          title: article.title,
          content: article.content,
          source: source.name,
          publishedAt: new Date(article.publishedAt),
          insights: extractInsights(article.content)
        });
      }
    } catch (error) {
      console.error(`Error crawling ${source.name}:`, error);
    }
  }

  return results;
}

function extractInsights(content: string): string[] {
  // Extract key data points, statistics, quotes
  const insights: string[] = [];
  const numberPattern = /\d+%|\$\d+[BM]?|\d+\s*(million|billion)/gi;
  const matches = content.match(numberPattern);
  
  if (matches) {
    insights.push(...matches);
  }
  
  return insights;
}
```

### 2. AI Content Generation with Claude/OpenAI

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: ResearchResult[];
}

interface GeneratedContent {
  title: string;
  body: string;
  summary: string;
  hashtags: string[];
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

  async generateWithClaude(request: ContentRequest): Promise<GeneratedContent> {
    const prompt = this.buildPrompt(request);

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

  async generateWithOpenAI(request: ContentRequest): Promise<GeneratedContent> {
    const prompt = this.buildPrompt(request);

    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and trending topics.'
      }, {
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
      max_tokens: 4096
    });

    const content = completion.choices[0].message.content || '';
    return this.parseContent(content);
  }

  private buildPrompt(request: ContentRequest): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with clear rankings',
      'pov': 'Write from a unique perspective with personal insights',
      'case-study': 'Analyze a real-world example with data and outcomes',
      'how-to': 'Provide step-by-step instructions with actionable advice'
    };

    const researchContext = request.researchData
      .map(r => `${r.title} (${r.source}): ${r.content.substring(0, 200)}...`)
      .join('\n\n');

    return `
Create a ${request.format} article about "${request.keyword}" in ${request.language === 'en' ? 'English' : 'Vietnamese'}.

Tone: ${request.tone}
Format: ${formatInstructions[request.format]}

Recent Research Data:
${researchContext}

Requirements:
1. Use data and insights from the research
2. Write in a ${request.tone} tone
3. Include statistics and quotes where relevant
4. Make it engaging and shareable
5. Add 5-8 relevant hashtags

Output in JSON format:
{
  "title": "Article title",
  "body": "Full article content with formatting",
  "summary": "Brief summary (2-3 sentences)",
  "hashtags": ["hashtag1", "hashtag2"]
}
`;
  }

  private parseContent(content: string): GeneratedContent {
    try {
      // Try to extract JSON from markdown code blocks
      const jsonMatch = content.match(/```json\s*([\s\S]*?)\s*```/);
      const jsonString = jsonMatch ? jsonMatch[1] : content;
      
      return JSON.parse(jsonString);
    } catch (error) {
      // Fallback parsing if JSON extraction fails
      return {
        title: this.extractTitle(content),
        body: content,
        summary: content.substring(0, 200),
        hashtags: this.extractHashtags(content)
      };
    }
  }

  private extractTitle(content: string): string {
    const titleMatch = content.match(/^#\s*(.+)$/m);
    return titleMatch ? titleMatch[1] : 'Untitled';
  }

  private extractHashtags(content: string): string[] {
    const hashtags = content.match(/#\w+/g) || [];
    return hashtags.slice(0, 8);
  }
}
```

### 3. Multilingual Content Generation

```typescript
// lib/content/multilingual.ts
import { ContentGenerator } from '../ai/content-generator';

export async function generateMultilingualContent(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  researchData: ResearchResult[]
): Promise<{ en: GeneratedContent; vi: GeneratedContent }> {
  const generator = new ContentGenerator();

  const [enContent, viContent] = await Promise.all([
    generator.generateWithClaude({
      keyword,
      format,
      tone: 'expert',
      language: 'en',
      researchData
    }),
    generator.generateWithClaude({
      keyword,
      format,
      tone: 'expert',
      language: 'vi',
      researchData
    })
  ]);

  return { en: enContent, vi: viContent };
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: GeneratedContent;
  style: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition details
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.content.title,
      body: config.content.body,
      hashtags: config.content.hashtags,
      style: config.style
    },
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${config.style}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.content.title,
      body: config.content.body,
      hashtags: config.content.hashtags,
      style: config.style
    },
  });

  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import React from 'react';
import {
  AbsoluteFill,
  interpolate,
  useCurrentFrame,
  useVideoConfig,
  spring
} from 'remotion';

interface ContentVideoProps {
  title: string;
  body: string;
  hashtags: string[];
  style: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  body,
  hashtags,
  style
}) => {
  const frame = useCurrentFrame();
  const { fps, width, height } = useVideoConfig();

  const titleOpacity = spring({
    frame: frame - 10,
    fps,
    config: {
      damping: 200,
    },
  });

  const contentOpacity = interpolate(
    frame,
    [30, 50],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  // Aspect ratio based on platform
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        fontFamily: 'Arial, sans-serif',
        padding: 60
      }}
    >
      {/* Title */}
      <div
        style={{
          opacity: titleOpacity,
          fontSize: 72,
          fontWeight: 'bold',
          color: '#fff',
          marginBottom: 40,
          lineHeight: 1.2
        }}
      >
        {title}
      </div>

      {/* Body Content */}
      <div
        style={{
          opacity: contentOpacity,
          fontSize: 48,
          color: '#ddd',
          lineHeight: 1.6,
          maxHeight: height - 400,
          overflow: 'hidden'
        }}
      >
        {body.substring(0, 300)}...
      </div>

      {/* Hashtags */}
      <div
        style={{
          position: 'absolute',
          bottom: 60,
          left: 60,
          fontSize: 36,
          color: '#58a6ff',
          opacity: contentOpacity
        }}
      >
        {hashtags.join(' ')}
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { crawlNews } from '../research/crawler';
import { ContentGenerator } from '../ai/content-generator';
import { generateMultilingualContent } from '../content/multilingual';
import { renderContentVideo } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  generateVideo: boolean;
  videoStyle?: 'reels' | 'tiktok' | 'shorts';
}

interface PipelineResult {
  research: ResearchResult[];
  content: {
    en: GeneratedContent;
    vi: GeneratedContent;
  };
  videos?: {
    en?: string;
    vi?: string;
  };
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);

  // Step 1: Research
  console.log('Step 1: Crawling news sources...');
  const sources = [
    { name: 'TechCrunch', url: 'https://techcrunch.com', selector: '.article' },
    { name: 'a16z', url: 'https://a16z.com', selector: '.post' }
  ];
  
  const research = await crawlNews(config.keyword, sources);
  console.log(`Found ${research.length} articles`);

  // Step 2: Generate multilingual content
  console.log('Step 2: Generating content with AI...');
  const content = await generateMultilingualContent(
    config.keyword,
    config.format,
    research
  );
  console.log('Content generated in English and Vietnamese');

  // Step 3: Render videos (optional)
  let videos: { en?: string; vi?: string } | undefined;
  
  if (config.generateVideo && config.videoStyle) {
    console.log('Step 3: Rendering videos...');
    
    const [enVideo, viVideo] = await Promise.all([
      renderContentVideo({
        content: content.en,
        style: config.videoStyle,
        duration: 30
      }),
      renderContentVideo({
        content: content.vi,
        style: config.videoStyle,
        duration: 30
      })
    ]);

    videos = { en: enVideo, vi: viVideo };
    console.log('Videos rendered successfully');
  }

  return {
    research,
    content,
    videos
  };
}
```

## API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, generateVideo, videoStyle } = body;

    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields: keyword, format' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format,
      generateVideo: generateVideo || false,
      videoStyle: videoStyle || 'reels'
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

// Generate a toplist article without video
const result = await runContentPipeline({
  keyword: 'AI trends 2026',
  format: 'toplist',
  generateVideo: false
});

console.log('English Title:', result.content.en.title);
console.log('Vietnamese Title:', result.content.vi.title);
```

### Complete Pipeline with Video

```typescript
// Generate content with video for TikTok
const result = await runContentPipeline({
  keyword: 'Web3 gaming',
  format: 'case-study',
  generateVideo: true,
  videoStyle: 'tiktok'
});

console.log('Research articles:', result.research.length);
console.log('English video:', result.videos?.en);
console.log('Vietnamese video:', result.videos?.vi);
```

### Custom Research Sources

```typescript
import { crawlNews } from '@/lib/research/crawler';

const customSources = [
  { name: 'ProductHunt', url: 'https://producthunt.com', selector: '.post' },
  { name: 'HackerNews', url: 'https://news.ycombinator.com', selector: '.athing' }
];

const research = await crawlNews('startup funding', customSources);
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the application
# http://localhost:3000
```

## Building for Production

```bash
# Build the Next.js application
npm run build

# Start production server
npm start
```

## Common Patterns

### Batch Content Generation

```typescript
const keywords = ['AI automation', 'Marketing tools', 'Content creation'];
const formats = ['toplist', 'how-to', 'pov'] as const;

const results = await Promise.all(
  keywords.map(async (keyword, index) => {
    return runContentPipeline({
      keyword,
      format: formats[index % formats.length],
      generateVideo: true,
      videoStyle: 'reels'
    });
  })
);
```

### Content Scheduling

```typescript
interface ScheduledContent {
  content: GeneratedContent;
  scheduledAt: Date;
  platform: string;
}

async function scheduleContent(
  keyword: string,
  scheduledAt: Date
): Promise<ScheduledContent> {
  const result = await runContentPipeline({
    keyword,
    format: 'pov',
    generateVideo: false
  });

  return {
    content: result.content.en,
    scheduledAt,
    platform: 'facebook'
  };
}
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY is not set');
}

if (!process.env.OPENAI_API_KEY) {
  throw new Error('OPENAI_API_KEY is not set');
}
```

### Rate Limiting

```typescript
// Implement rate limiting for API calls
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  keywords.map(keyword => 
    limit(() => runContentPipeline({
      keyword,
      format: 'toplist',
      generateVideo: false
    }))
  )
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for video rendering
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  inputProps: videoProps
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
function chunkArray<T>(array: T[], size: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}

const chunks = chunkArray(largeResearchArray, 10);
for (const chunk of chunks) {
  await processResearchChunk(chunk);
}
```

## Performance Optimization

### Caching Research Results

```typescript
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour cache

async function getCachedResearch(keyword: string): Promise<ResearchResult[]> {
  const cached = cache.get<ResearchResult[]>(keyword);
  if (cached) return cached;

  const research = await crawlNews(keyword, sources);
  cache.set(keyword, research);
  return research;
}
```

### Parallel Processing

```typescript
// Generate content for multiple languages simultaneously
const [enContent, viContent, esContent] = await Promise.all([
  generator.generateWithClaude({ ...config, language: 'en' }),
  generator.generateWithClaude({ ...config, language: 'vi' }),
  generator.generateWithClaude({ ...config, language: 'es' })
]);
```

This skill provides comprehensive coverage of the Marketing Pipeline Share project, enabling AI coding agents to effectively assist developers in implementing automated content creation workflows with research, multilingual generation, and video rendering capabilities.
