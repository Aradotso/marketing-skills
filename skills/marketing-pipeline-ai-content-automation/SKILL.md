---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing pipeline for content
  - generate blog posts and videos from keywords automatically
  - use Claude and OpenAI for content automation
  - create AI-powered content research and scripting pipeline
  - automate video generation from written content
  - build automated content workflow with Remotion
  - research and write content using AI crawlers
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - an automated content creation system that handles research, scriptwriting, and video generation using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

- **Auto-crawls** recent news and insights from TechCrunch, a16z, Twitter, LinkedIn
- **Generates content** in multiple formats (listicles, POV, case studies, how-to) using Claude/OpenAI
- **Supports multiple languages** (English & Vietnamese) with customizable tone
- **Renders videos automatically** from written content using Remotion
- **Optimizes for platforms** (Reels, TikTok, Shorts)

Built with TypeScript, Next.js, and integrates with OpenAI, Anthropic Claude, and RapidAPI.

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

Create a `.env.local` file in the root directory with the required API keys:

```bash
# OpenAI Configuration
OPENAI_API_KEY=your_openai_key_here

# Anthropic Claude Configuration
ANTHROPIC_API_KEY=your_anthropic_key_here

# RapidAPI for News Crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom API endpoints
NEXT_PUBLIC_API_BASE_URL=http://localhost:3000/api

# Remotion Configuration (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (OpenAI, Claude)
│   │   ├── crawler/     # Content research crawlers
│   │   ├── generator/   # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── api/             # API routes
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key API Routes

### Content Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources } = await request.json();
  
  const results = await crawlNews({
    keyword,
    sources: sources || ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });
  
  return NextResponse.json({ data: results });
}
```

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function POST(request: NextRequest) {
  const { research, format, language, tone, provider } = await request.json();
  
  const prompt = buildPrompt(research, format, language, tone);
  
  let content;
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{ role: 'user', content: prompt }],
    });
    content = message.content[0].text;
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{ role: 'user', content: prompt }],
    });
    content = completion.choices[0].message.content;
  }
  
  return NextResponse.json({ content });
}

function buildPrompt(research: string, format: string, language: string, tone: string): string {
  return `Based on this research data: ${research}

Create a ${format} article in ${language} with a ${tone} tone.

Format requirements:
- ${format === 'toplist' ? 'Numbered list with clear points' : ''}
- ${format === 'pov' ? 'Personal perspective with insights' : ''}
- ${format === 'case-study' ? 'Problem-solution-results structure' : ''}
- Include data and statistics from the research
- Make it engaging and actionable`;
}
```

### Video Generation Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  const { content, templateId, platform } = await request.json();
  
  // Bundle Remotion composition
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: templateId,
    inputProps: {
      content,
      platform, // 'reels', 'tiktok', 'shorts'
    },
  });
  
  // Render video
  const outputLocation = path.join(process.cwd(), 'public/videos', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      platform,
    },
  });
  
  return NextResponse.json({ 
    videoUrl: `/videos/${path.basename(outputLocation)}` 
  });
}
```

## Core Library Usage

### Content Crawler

```typescript
// src/lib/crawler/index.ts
import axios from 'axios';

interface CrawlOptions {
  keyword: string;
  sources: string[];
  timeframe: string;
}

export async function crawlNews(options: CrawlOptions) {
  const { keyword, sources, timeframe } = options;
  
  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await axios.get('https://newsapi.p.rapidapi.com/search', {
        params: {
          q: keyword,
          sources: source,
          timeframe,
        },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          'X-RapidAPI-Host': 'newsapi.p.rapidapi.com',
        },
      });
      
      return response.data.articles;
    })
  );
  
  return results.flat();
}
```

### AI Content Generator

```typescript
// src/lib/generator/ai-writer.ts
import Anthropic from '@anthropic-ai/sdk';

export class AIContentWriter {
  private client: Anthropic;
  
  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }
  
  async generateContent(params: {
    research: string;
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    language: 'en' | 'vi';
    tone: 'expert' | 'friendly' | 'humorous';
  }) {
    const systemPrompt = this.buildSystemPrompt(params.format, params.language, params.tone);
    
    const message = await this.client.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      system: systemPrompt,
      messages: [
        {
          role: 'user',
          content: `Research data:\n${params.research}\n\nGenerate the article.`
        }
      ],
    });
    
    return message.content[0].text;
  }
  
  private buildSystemPrompt(format: string, language: string, tone: string): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list format with clear, actionable points',
      'pov': 'Write from a personal perspective with unique insights',
      'case-study': 'Structure as problem, solution, and measurable results',
      'how-to': 'Provide step-by-step instructions with clear outcomes',
    };
    
    return `You are an expert content writer. 
Format: ${formatInstructions[format]}
Language: ${language === 'en' ? 'English' : 'Vietnamese'}
Tone: ${tone}
Always include data, statistics, and real examples from the research provided.`;
  }
}
```

### Remotion Video Component

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, platform }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  // Parse content into scenes
  const scenes = content.split('\n\n').filter(Boolean);
  const currentSceneIndex = Math.floor(frame / (fps * 3)); // 3 seconds per scene
  const currentScene = scenes[currentSceneIndex] || scenes[scenes.length - 1];
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 40,
      }}
    >
      <h1 style={{ 
        color: '#fff', 
        fontSize: platform === 'reels' ? 48 : 42,
        textAlign: 'center',
        fontWeight: 'bold',
      }}>
        {currentScene}
      </h1>
    </AbsoluteFill>
  );
};
```

## Common Usage Patterns

### Full Pipeline Example

```typescript
// Example: Complete content creation workflow
import { AIContentWriter } from '@/lib/generator/ai-writer';
import { crawlNews } from '@/lib/crawler';

async function createContentPipeline(keyword: string) {
  // Step 1: Research
  const research = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeframe: '24h',
  });
  
  // Step 2: Generate content
  const writer = new AIContentWriter();
  const article = await writer.generateContent({
    research: JSON.stringify(research),
    format: 'toplist',
    language: 'en',
    tone: 'expert',
  });
  
  // Step 3: Render video
  const videoResponse = await fetch('/api/render-video', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      content: article,
      templateId: 'content-video',
      platform: 'reels',
    }),
  });
  
  const { videoUrl } = await videoResponse.json();
  
  return { article, videoUrl };
}
```

### Multi-language Content Generation

```typescript
// Generate both English and Vietnamese versions
async function generateBilingualContent(research: string) {
  const writer = new AIContentWriter();
  
  const [english, vietnamese] = await Promise.all([
    writer.generateContent({
      research,
      format: 'how-to',
      language: 'en',
      tone: 'friendly',
    }),
    writer.generateContent({
      research,
      format: 'how-to',
      language: 'vi',
      tone: 'friendly',
    }),
  ]);
  
  return { english, vietnamese };
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video locally (if CLI configured)
npm run render

# Type checking
npm run type-check

# Lint code
npm run lint
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits with OpenAI or Claude:

```typescript
// Implement retry logic with exponential backoff
async function generateWithRetry(params: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await writer.generateContent(params);
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Timeouts

For long videos, increase timeout in Remotion config:

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setTimeoutInMilliseconds(300000); // 5 minutes
Config.setConcurrency(2); // Reduce CPU usage
```

### Memory Issues with Large Content

```typescript
// Stream large content instead of loading all at once
import { Readable } from 'stream';

async function streamContent(content: string) {
  const stream = Readable.from(content.split('\n'));
  // Process chunks instead of full content
  for await (const chunk of stream) {
    // Process each line
  }
}
```

### Environment Variable Not Found

Ensure all required environment variables are set:

```typescript
// src/lib/config.ts
export function validateConfig() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}
```

## Best Practices

1. **Always validate research data** before generating content to ensure quality
2. **Cache API responses** to reduce costs and improve performance
3. **Use streaming** for real-time content generation feedback
4. **Implement queue systems** for video rendering to handle multiple requests
5. **Monitor API usage** to stay within budget limits
6. **Test different AI models** (Claude vs OpenAI) for different content types
