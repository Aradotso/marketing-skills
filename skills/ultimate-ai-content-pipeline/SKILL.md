---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do i generate automated content with ai pipeline
  - set up content automation with remotion video
  - create ai powered content research and scriptwriting
  - automate content creation from research to video
  - use claude and openai for content generation pipeline
  - build automated marketing content with ai agents
  - generate videos from articles using remotion
  - set up multilingual content automation system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a TypeScript-based content automation system that transforms keywords into complete content pieces including research, articles, and videos. It crawls real-time data from sources like TechCrunch and Twitter, generates multi-format content using Claude/OpenAI, and renders videos automatically using Remotion.

**Key capabilities:**
- Auto-research from live news sources (24h fresh data)
- Multi-format content generation (toplist, POV, case study, how-to)
- Bilingual output (English/Vietnamese) with customizable tone
- Automatic video/infographic rendering via Remotion
- Next.js frontend for content management

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

## Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run render
```

## Core Architecture

### 1. Research Module

The pipeline starts by crawling fresh content from multiple sources:

```typescript
// lib/research/crawler.ts
interface ResearchSource {
  name: string;
  url: string;
  category: string;
}

async function crawlSources(keyword: string): Promise<ResearchData[]> {
  const sources = [
    { name: 'TechCrunch', url: 'https://techcrunch.com/search', category: 'tech' },
    { name: 'a16z', url: 'https://a16z.com/posts', category: 'vc' }
  ];

  const results = await Promise.all(
    sources.map(source => fetchArticles(source, keyword))
  );

  return results.flat();
}

async function fetchArticles(source: ResearchSource, keyword: string) {
  const response = await fetch(source.url, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!
    }
  });
  
  const data = await response.json();
  return parseArticles(data, source.category);
}
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI based on research:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchData[];
}

async function generateContent(config: ContentConfig): Promise<string> {
  const systemPrompt = buildSystemPrompt(config);
  const userPrompt = buildUserPrompt(config);

  // Using Claude for better structured output
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: userPrompt
    }],
    system: systemPrompt
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a unique perspective with strong opinions backed by data',
    'case-study': 'Analyze a real example with problem, solution, and results',
    'how-to': 'Provide step-by-step instructions with actionable tips'
  };

  const toneGuide = {
    'expert': 'Professional, data-driven, authoritative',
    'friendly': 'Conversational, approachable, helpful',
    'humorous': 'Witty, entertaining, engaging'
  };

  return `You are an expert content writer. 
Format: ${formatInstructions[config.format]}
Tone: ${toneGuide[config.tone]}
Language: ${config.language === 'vi' ? 'Vietnamese' : 'English'}
Use the provided research data to create accurate, up-to-date content.`;
}

function buildUserPrompt(config: ContentConfig): string {
  const researchSummary = config.research
    .map(r => `- ${r.title}: ${r.summary}`)
    .join('\n');

  return `Based on this recent research:\n${researchSummary}\n\nCreate a ${config.format} article.`;
}
```

### 3. Dual-Language Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
// lib/ai/bilingual-generator.ts
async function generateBilingualContent(
  research: ResearchData[],
  format: ContentConfig['format']
): Promise<{ en: string; vi: string }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      format,
      language: 'en',
      tone: 'expert',
      research
    }),
    generateContent({
      format,
      language: 'vi',
      tone: 'friendly',
      research
    })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Rendering with Remotion

Convert generated content into videos:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  bgColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  bgColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        padding: 60,
        opacity,
        transform: `translateY(${Math.max(0, 30 - frame)}px)`
      }}>
        <h1 style={{ fontSize: 72, marginBottom: 40 }}>
          {title}
        </h1>
        {points.map((point, i) => {
          const pointFrame = frame - (i * fps * 2);
          const pointOpacity = Math.min(1, Math.max(0, pointFrame / fps));
          
          return (
            <p key={i} style={{ 
              fontSize: 48, 
              marginBottom: 30,
              opacity: pointOpacity 
            }}>
              {i + 1}. {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

Render the video:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: string, outputPath: string) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: extractTitle(content),
      points: extractPoints(content),
      bgColor: '#1a1a2e'
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
  });

  console.log(`Video rendered: ${outputPath}`);
}

function extractTitle(content: string): string {
  const match = content.match(/^#\s+(.+)$/m);
  return match ? match[1] : 'Content Title';
}

function extractPoints(content: string): string[] {
  const matches = content.match(/^\d+\.\s+(.+)$/gm) || [];
  return matches.map(m => m.replace(/^\d+\.\s+/, ''));
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { crawlSources } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/ai/bilingual-generator';
import { renderContentVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, generateVideo } = req.body;

  try {
    // Step 1: Research
    const research = await crawlSources(keyword);
    
    // Step 2: Generate content
    const content = await generateBilingualContent(research, format);
    
    // Step 3: Optional video generation
    let videoUrl = null;
    if (generateVideo) {
      const videoPath = `./output/${Date.now()}.mp4`;
      await renderContentVideo(content.en, videoPath);
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }

    res.status(200).json({
      success: true,
      content,
      videoUrl,
      research: research.length
    });
  } catch (error) {
    console.error('Generation error:', error);
    res.status(500).json({ 
      error: 'Content generation failed',
      details: error.message 
    });
  }
}
```

## Common Patterns

### Pattern 1: Complete Content Pipeline

```typescript
// lib/pipeline/complete-flow.ts
import { crawlSources } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

async function runCompletePipeline(keyword: string) {
  console.log('🔍 Starting research...');
  const research = await crawlSources(keyword);
  
  console.log('✍️ Generating content...');
  const content = await generateContent({
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    research
  });
  
  console.log('🎬 Rendering video...');
  await renderContentVideo(content, `./output/${keyword}-video.mp4`);
  
  console.log('✅ Pipeline complete!');
  return { content, research };
}
```

### Pattern 2: Batch Content Generation

```typescript
// scripts/batch-generate.ts
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    const research = await crawlSources(keyword);
    const content = await generateBilingualContent(research, 'how-to');
    
    results.push({
      keyword,
      content,
      timestamp: new Date()
    });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}

// Usage
const keywords = ['AI marketing', 'content automation', 'video creation'];
const output = await batchGenerate(keywords);
```

### Pattern 3: Custom Video Templates

```typescript
// remotion/templates/ToplistVideo.tsx
export const ToplistVideo: React.FC<{ items: string[] }> = ({ items }) => {
  const frame = useCurrentFrame();
  const { durationInFrames, fps } = useVideoConfig();
  
  const itemDuration = durationInFrames / items.length;
  const currentIndex = Math.floor(frame / itemDuration);
  
  return (
    <AbsoluteFill>
      {items.map((item, i) => (
        <div
          key={i}
          style={{
            opacity: i === currentIndex ? 1 : 0,
            transition: 'opacity 0.5s',
          }}
        >
          <h2>#{i + 1}</h2>
          <p>{item}</p>
        </div>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### Issue: API Rate Limiting

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
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const task = this.queue.shift()!;
    await task();
    await new Promise(resolve => setTimeout(resolve, 1000));
    this.processing = false;
    this.process();
  }
}

export const apiLimiter = new RateLimiter();
```

### Issue: Video Rendering Fails

Check Remotion configuration and ensure all dependencies are installed:

```bash
# Install Remotion CLI
npm install -g @remotion/cli

# Test render locally
npx remotion preview remotion/index.ts
```

### Issue: Out of Memory During Generation

Implement streaming for large content:

```typescript
async function streamGeneration(config: ContentConfig) {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{ role: 'user', content: buildUserPrompt(config) }],
    system: buildSystemPrompt(config)
  });

  let fullContent = '';
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta' && 
        chunk.delta.type === 'text_delta') {
      fullContent += chunk.delta.text;
      process.stdout.write(chunk.delta.text);
    }
  }
  
  return fullContent;
}
```

## Advanced Usage

### Multi-Platform Video Export

```typescript
// lib/video/multi-platform.ts
const platformSpecs = {
  tiktok: { width: 1080, height: 1920, fps: 30 },
  youtube: { width: 1920, height: 1080, fps: 60 },
  instagram: { width: 1080, height: 1080, fps: 30 }
};

async function renderForAllPlatforms(content: string) {
  for (const [platform, specs] of Object.entries(platformSpecs)) {
    await renderMedia({
      composition: { ...baseComposition, ...specs },
      serveUrl: bundleLocation,
      outputLocation: `./output/${platform}-${Date.now()}.mp4`
    });
  }
}
```

This skill enables AI agents to effectively use the Ultimate AI Content Pipeline for automated content creation, from research through video generation.
