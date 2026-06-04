---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to script generation to video rendering using AI and Remotion
triggers:
  - how do I generate automated content with AI pipeline
  - set up content research and video generation system
  - use remotion to create marketing videos automatically
  - automate content creation from keyword to video
  - integrate claude and openai for content generation
  - create multi-language content with AI automation
  - build automated marketing content pipeline
  - generate videos from text content automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from researching trending topics across multiple sources (TechCrunch, a16z, Twitter, LinkedIn), to generating multilingual content in various formats, to automatically rendering videos with Remotion.

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

## Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Core Features

### 1. Research & Content Crawling

The system automatically crawls and analyzes content from multiple sources:

```typescript
// Example research module structure
import { ResearchService } from './services/research';

interface ResearchConfig {
  sources: string[];
  keywords: string[];
  timeframe: '24h' | '7d' | '30d';
  language: 'en' | 'vi' | 'both';
}

async function performResearch(config: ResearchConfig) {
  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });
  
  const results = await researchService.crawl({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    keywords: config.keywords,
    timeframe: config.timeframe
  });
  
  return researchService.analyzeInsights(results);
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  keywords: string[];
}

async function generateContent(
  research: any,
  config: ContentConfig
) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!
  });
  
  const prompt = buildPrompt(research, config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content;
}

function buildPrompt(research: any, config: ContentConfig): string {
  return `
    Based on the following research insights:
    ${JSON.stringify(research, null, 2)}
    
    Create a ${config.format} article in ${config.language} with a ${config.tone} tone.
    Include data-backed insights and recent trends.
    Keywords to focus on: ${config.keywords.join(', ')}
  `;
}
```

### 3. Multilingual Content Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
interface MultilingualContent {
  en: string;
  vi: string;
}

async function generateMultilingualContent(
  research: any,
  baseConfig: Omit<ContentConfig, 'language'>
): Promise<MultilingualContent> {
  const [enContent, viContent] = await Promise.all([
    generateContent(research, { ...baseConfig, language: 'en' }),
    generateContent(research, { ...baseConfig, language: 'vi' })
  ]);
  
  return {
    en: enContent,
    vi: viContent
  };
}
```

### 4. Video Generation with Remotion

Automatically render videos from generated content:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  outputPath: string;
}

async function generateVideo(config: VideoConfig) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: webpackOverride
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      format: config.format
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: config.outputPath,
    inputProps: {
      content: config.content,
      format: config.format
    }
  });
  
  return config.outputPath;
}
```

### 5. Complete Pipeline Example

```typescript
// Full content pipeline from keyword to video
async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await performResearch({
      sources: ['techcrunch', 'a16z', 'twitter'],
      keywords: [keyword],
      timeframe: '24h',
      language: 'both'
    });
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateMultilingualContent(research, {
      format: 'toplist',
      tone: 'expert',
      keywords: [keyword]
    });
    
    // Step 3: Generate videos for both languages
    console.log('🎬 Rendering videos...');
    const [enVideo, viVideo] = await Promise.all([
      generateVideo({
        content: content.en,
        format: 'reels',
        outputPath: `./output/${keyword}-en.mp4`
      }),
      generateVideo({
        content: content.vi,
        format: 'reels',
        outputPath: `./output/${keyword}-vi.mp4`
      })
    ]);
    
    console.log('✅ Pipeline complete!');
    return {
      research,
      content,
      videos: { en: enVideo, vi: viVideo }
    };
  } catch (error) {
    console.error('❌ Pipeline error:', error);
    throw error;
  }
}
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, language } = req.body;
  
  try {
    const research = await performResearch({
      sources: ['techcrunch', 'twitter'],
      keywords: [keyword],
      timeframe: '24h',
      language: language || 'both'
    });
    
    const content = await generateContent(research, {
      format: format || 'toplist',
      tone: 'expert',
      language: language || 'en',
      keywords: [keyword]
    });
    
    res.status(200).json({ success: true, content });
  } catch (error) {
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

### Video Generation Endpoint

```typescript
// pages/api/generate-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { content, format } = req.body;
  
  try {
    const videoPath = await generateVideo({
      content,
      format: format || 'reels',
      outputPath: `./public/videos/${Date.now()}.mp4`
    });
    
    res.status(200).json({ 
      success: true, 
      videoUrl: videoPath.replace('./public', '')
    });
  } catch (error) {
    res.status(500).json({ error: 'Video generation failed' });
  }
}
```

## Remotion Video Components

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  content, 
  format 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
        opacity
      }}
    >
      <div style={{ 
        color: 'white', 
        fontSize: format === 'reels' ? 48 : 42,
        padding: 40,
        textAlign: 'center'
      }}>
        {content}
      </div>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = await fetchTrendingKeywords();
  await batchGenerateContent(trendingKeywords);
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries reached');
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos sequentially to avoid memory issues
async function renderVideosSequentially(configs: VideoConfig[]) {
  const results = [];
  for (const config of configs) {
    const result = await generateVideo(config);
    results.push(result);
    // Give system time to clean up
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  return results;
}
```

### Content Quality Validation

```typescript
function validateContent(content: string): boolean {
  const minLength = 500;
  const hasHeadings = /#{1,3}\s/.test(content);
  const hasData = /\d+%|\d+\s*(người|users|million)/.test(content);
  
  return content.length >= minLength && hasHeadings && hasData;
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render a single video (if CLI exists)
npm run render -- --id=ContentVideo --props='{"content":"test"}'

# Run tests
npm test
```
