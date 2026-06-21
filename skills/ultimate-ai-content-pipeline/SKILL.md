---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline that researches, generates scripts, and creates videos automatically using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline for automated content creation
  - generate automated video content from research to publication
  - create multi-format content with AI research and video rendering
  - automate content workflow from research to video generation
  - build an AI content pipeline with Claude and Remotion
  - set up automated content generation with research crawling
  - create AI-powered marketing content pipeline
  - integrate OpenAI and Claude for content automation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

Ultimate AI Content Pipeline is a comprehensive content automation system that handles the entire content creation workflow: from research (crawling TechCrunch, a16z, Twitter, LinkedIn), to script generation in multiple formats (Toplist, POV, Case Study, How-to), to automatic video rendering using Remotion. It supports bilingual content (English/Vietnamese) and integrates with Claude 3, OpenAI, and RapidAPI.

The pipeline automates up to 90% of the content creation workflow, making it ideal for content creators, marketers, and businesses looking to scale their content production.

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
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Social Media APIs (optional)
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if using persistent storage)
DATABASE_URL=your_database_url

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Video rendering (Remotion)
npm run render
```

## Core Modules & Architecture

### 1. Research Module (Auto-Scan)

Automatically crawls and analyzes recent content from multiple sources:

```typescript
// lib/research/crawler.ts
import { RapidAPI } from '@/lib/api/rapidapi';

interface ResearchConfig {
  sources: string[];
  timeframe: '24h' | '7d' | '30d';
  keywords: string[];
  language: 'en' | 'vi' | 'both';
}

export async function conductResearch(config: ResearchConfig) {
  const rapidAPI = new RapidAPI(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    config.sources.map(async (source) => {
      switch (source) {
        case 'techcrunch':
          return await rapidAPI.fetchTechCrunch(config.keywords);
        case 'twitter':
          return await rapidAPI.fetchTwitter(config.keywords, config.timeframe);
        case 'linkedin':
          return await rapidAPI.fetchLinkedIn(config.keywords);
        default:
          return [];
      }
    })
  );
  
  return aggregateAndAnalyze(results.flat());
}

function aggregateAndAnalyze(rawData: any[]) {
  // Extract insights, trends, and data points
  return {
    trends: extractTrends(rawData),
    insights: extractInsights(rawData),
    dataBacked: extractStatistics(rawData),
    sources: rawData.map(d => d.url)
  };
}
```

### 2. Content Generation Module

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type VoiceTone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  format: ContentFormat;
  tone: VoiceTone;
  language: 'en' | 'vi' | 'both';
  researchData: any;
  targetAudience: string;
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
  });

  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';

  if (config.language === 'both') {
    return {
      en: content,
      vi: await translateContent(content, 'vi')
    };
  }

  return content;
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list format highlighting the top items',
    'pov': 'Write from a specific point of view with strong opinions',
    'case-study': 'Structure as a detailed case study with problem-solution-results',
    'how-to': 'Create step-by-step instructional content'
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and humor while maintaining informativeness'
  };

  return `
You are a content creator for ${config.targetAudience}.

Format: ${formatInstructions[config.format]}
Tone: ${toneInstructions[config.tone]}
Language: ${config.language === 'vi' ? 'Vietnamese' : 'English'}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Create comprehensive, engaging content that:
1. Uses the latest research data provided
2. Includes specific statistics and sources
3. Follows the ${config.format} format exactly
4. Maintains a ${config.tone} tone throughout
5. Is optimized for social media sharing

Output the content ready for publication.
  `.trim();
}
```

### 3. Video Generation Module (Remotion)

Automatically render videos from generated content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
  style: 'minimal' | 'dynamic' | 'infographic';
}

export async function renderContentVideo(config: VideoConfig) {
  const dimensions = getVideoDimensions(config.format);
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      style: config.style,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'renders',
    `video-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: config.content,
      style: config.style,
    },
  });

  return {
    path: outputLocation,
    duration: composition.durationInFrames / composition.fps,
    dimensions,
  };
}

function getVideoDimensions(format: string) {
  const dimensions = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'shorts': { width: 1080, height: 1920 },
  };
  return dimensions[format] || dimensions.reels;
}
```

### 4. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, spring } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  style: 'minimal' | 'dynamic' | 'infographic';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, style }) => {
  const frame = useCurrentFrame();
  const { fps, width, height } = useVideoConfig();

  const sections = parseContentSections(content);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {sections.map((section, index) => {
        const startFrame = index * fps * 3; // 3 seconds per section
        const progress = spring({
          frame: frame - startFrame,
          fps,
          config: {
            damping: 200,
          },
        });

        if (frame < startFrame || frame > startFrame + fps * 3) {
          return null;
        }

        return (
          <AbsoluteFill
            key={index}
            style={{
              opacity: progress,
              transform: `scale(${progress})`,
              display: 'flex',
              flexDirection: 'column',
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
            }}
          >
            <h1 style={{ 
              color: '#fff', 
              fontSize: 72,
              textAlign: 'center',
              fontWeight: 'bold',
              marginBottom: 40,
            }}>
              {section.title}
            </h1>
            <p style={{
              color: '#ccc',
              fontSize: 36,
              textAlign: 'center',
              maxWidth: '80%',
            }}>
              {section.description}
            </p>
          </AbsoluteFill>
        );
      })}
    </AbsoluteFill>
  );
};

function parseContentSections(content: string) {
  // Parse content into sections for video
  const lines = content.split('\n').filter(l => l.trim());
  const sections = [];
  
  for (let i = 0; i < lines.length; i += 2) {
    sections.push({
      title: lines[i],
      description: lines[i + 1] || '',
    });
  }
  
  return sections;
}
```

## Complete Pipeline Workflow

### End-to-End Content Creation

```typescript
// lib/pipeline/orchestrator.ts
import { conductResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/content/generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: VoiceTone;
  language: 'en' | 'vi' | 'both';
  includeVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('🔍 Step 1: Conducting research...');
  const research = await conductResearch({
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
    keywords: [config.keyword],
    language: config.language,
  });

  console.log('✍️ Step 2: Generating content...');
  const content = await generateContent({
    format: config.format,
    tone: config.tone,
    language: config.language,
    researchData: research,
    targetAudience: 'marketers and content creators',
  });

  let video = null;
  if (config.includeVideo && config.videoFormat) {
    console.log('🎬 Step 3: Rendering video...');
    video = await renderContentVideo({
      content: typeof content === 'string' ? content : content.en,
      format: config.videoFormat,
      duration: 30,
      style: 'dynamic',
    });
  }

  console.log('✅ Pipeline completed!');
  return {
    research,
    content,
    video,
    metadata: {
      keyword: config.keyword,
      format: config.format,
      createdAt: new Date().toISOString(),
    },
  };
}
```

### API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      tone: body.tone || 'friendly',
      language: body.language || 'en',
      includeVideo: body.includeVideo || false,
      videoFormat: body.videoFormat || 'reels',
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### 1. Quick Content Generation

```typescript
// Generate a quick article
const result = await runContentPipeline({
  keyword: 'AI marketing trends 2024',
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  includeVideo: false,
});

console.log(result.content);
```

### 2. Full Pipeline with Video

```typescript
// Complete workflow with video
const result = await runContentPipeline({
  keyword: 'ChatGPT marketing strategies',
  format: 'how-to',
  tone: 'friendly',
  language: 'both',
  includeVideo: true,
  videoFormat: 'reels',
});

// Content available in both languages
console.log(result.content.en);
console.log(result.content.vi);
console.log('Video path:', result.video.path);
```

### 3. Research-Only Mode

```typescript
// Just conduct research
import { conductResearch } from '@/lib/research/crawler';

const insights = await conductResearch({
  sources: ['techcrunch', 'twitter'],
  timeframe: '7d',
  keywords: ['AI', 'marketing automation'],
  language: 'en',
});

console.log('Trends:', insights.trends);
console.log('Sources:', insights.sources);
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are properly loaded
function validateEnvironment() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

// Call before running pipeline
validateEnvironment();
```

### Rate Limiting

```typescript
// Implement rate limiting for API calls
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      limit(() => runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'friendly',
        language: 'en',
        includeVideo: false,
      }))
    )
  );
  
  return results;
}
```

### Video Rendering Errors

```bash
# Ensure ffmpeg is installed
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows
# Download from https://ffmpeg.org/download.html
```

```typescript
// Verify ffmpeg availability
import { getExecutableBinary } from '@remotion/renderer';

try {
  const ffmpegPath = getExecutableBinary('ffmpeg');
  console.log('FFmpeg found at:', ffmpegPath);
} catch (error) {
  console.error('FFmpeg not found. Please install it.');
}
```

### Memory Issues with Large Content

```typescript
// Stream large content generation
async function generateLargeContent(config: ContentConfig) {
  const chunks = [];
  
  // Generate in smaller chunks
  for (const section of config.researchData.sections) {
    const chunk = await generateContent({
      ...config,
      researchData: { ...config.researchData, sections: [section] }
    });
    chunks.push(chunk);
  }
  
  return chunks.join('\n\n');
}
```

## Best Practices

1. **Always validate environment variables** before running the pipeline
2. **Cache research results** to avoid redundant API calls
3. **Use appropriate rate limiting** when making multiple requests
4. **Monitor API usage** to stay within quota limits
5. **Test video rendering** with short durations first
6. **Store generated content** with proper metadata for future reference
7. **Implement error recovery** for failed pipeline steps

## Additional Resources

- Check `HUONG_DAN_CAI_DAT.md` for detailed Vietnamese installation guide
- Remotion documentation: https://remotion.dev
- Claude API: https://docs.anthropic.com
- OpenAI API: https://platform.openai.com/docs
