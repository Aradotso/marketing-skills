---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up the AI content pipeline"
  - "automate content research and generation"
  - "create video content from text automatically"
  - "build a content automation workflow"
  - "generate multilingual content with AI"
  - "scrape news and create content from it"
  - "render videos from blog posts"
  - "set up marketing content pipeline"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content creation pipeline: from researching trending topics, generating multi-format articles in multiple languages, to automatically rendering videos and graphics using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

This pipeline automates content creation through four main stages:

1. **Auto-Research**: Crawls fresh data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically converts content to infographics and short-form videos using Remotion

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping & data collection
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Components

### 1. Research Module

Auto-scrapes trending content from multiple sources:

```typescript
// src/lib/research/scraper.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  source: string;
  publishedAt: Date;
}

export async function researchTopic(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const data = await fetchFromSource(source, keyword);
    results.push(...data);
  }
  
  return results;
}

async function fetchFromSource(
  source: string,
  keyword: string
): Promise<ResearchResult[]> {
  const options = {
    method: 'GET',
    url: `https://api.rapidapi.com/${source}/search`,
    params: { q: keyword, limit: 10 },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': `${source}.rapidapi.com`
    }
  };
  
  const response = await axios.request(options);
  return response.data.articles || [];
}
```

### 2. AI Content Generation

Generate multi-format content using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  researchData: any[];
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
  
  async generateWithClaude(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);
    
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
  
  async generateWithOpenAI(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: 'You are an expert content creator.' },
        { role: 'user', content: prompt }
      ],
      temperature: 0.7
    });
    
    return completion.choices[0].message.content || '';
  }
  
  private buildPrompt(request: ContentRequest): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with clear benefits',
      'pov': 'Write from a unique perspective or opinion',
      'case-study': 'Analyze real examples with data and outcomes',
      'how-to': 'Provide step-by-step instructions'
    };
    
    const toneInstructions = {
      'professional': 'Use formal language and expert terminology',
      'friendly': 'Use conversational and approachable language',
      'humorous': 'Include wit and engaging storytelling'
    };
    
    return `
Create a ${request.format} article in ${request.language} about "${request.keyword}".

Format: ${formatInstructions[request.format]}
Tone: ${toneInstructions[request.tone]}

Research Data:
${JSON.stringify(request.researchData, null, 2)}

Requirements:
- Use recent data and statistics from the research
- Include actionable insights
- Optimize for readability and engagement
- Add relevant headings and subheadings
`;
  }
}
```

### 3. Video Generation with Remotion

Convert content to video format:

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format
    }
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format
    }
  });
  
  return outputLocation;
}
```

### 4. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  format
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{ 
          opacity,
          justifyContent: 'center',
          alignItems: 'center',
          padding: 40
        }}>
          <h1 style={{ 
            color: 'white',
            fontSize: 60,
            textAlign: 'center',
            fontWeight: 'bold'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      <Sequence from={90}>
        <AbsoluteFill style={{ 
          padding: 40,
          color: 'white',
          fontSize: 32,
          lineHeight: 1.6
        }}>
          {content}
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
// src/lib/pipeline/executor.ts
import { researchTopic } from '../research/scraper';
import { ContentGenerator } from '../ai/content-generator';
import { renderContentVideo } from '../video/render';

export async function executeContentPipeline(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to' = 'toplist',
  language: 'en' | 'vi' = 'en'
) {
  console.log(`🔍 Starting research for: ${keyword}`);
  
  // Step 1: Research
  const researchData = await researchTopic(keyword);
  console.log(`✅ Found ${researchData.length} research items`);
  
  // Step 2: Generate Content
  const generator = new ContentGenerator();
  const content = await generator.generateWithClaude({
    keyword,
    format,
    language,
    tone: 'professional',
    researchData
  });
  console.log(`✅ Content generated (${content.length} chars)`);
  
  // Step 3: Render Video
  const videoPath = await renderContentVideo({
    content: content.substring(0, 500), // First 500 chars for video
    title: `${keyword} - ${format}`,
    format: 'reels'
  });
  console.log(`✅ Video rendered: ${videoPath}`);
  
  return {
    content,
    videoPath,
    researchData
  };
}
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { executeContentPipeline } from '@/lib/pipeline/executor';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await executeContentPipeline(keyword, format, language);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Running the Application

### Development Mode

```bash
npm run dev
```

### Build for Production

```bash
npm run build
npm start
```

### Render Video Standalone

```bash
# Render a specific composition
npx remotion render src/index.ts ContentVideo output.mp4 \
  --props='{"title":"My Title","content":"Content here","format":"reels"}'
```

## Common Usage Patterns

### Pattern 1: Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await executeContentPipeline(keyword, 'toplist', 'en');
    results.push(result);
    
    // Wait to avoid rate limits
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Pattern 2: Multi-language Output

```typescript
async function generateMultiLanguage(keyword: string) {
  const [english, vietnamese] = await Promise.all([
    executeContentPipeline(keyword, 'how-to', 'en'),
    executeContentPipeline(keyword, 'how-to', 'vi')
  ]);
  
  return { english, vietnamese };
}
```

### Pattern 3: Custom Research Sources

```typescript
// Add custom scrapers
async function customResearch(keyword: string) {
  const custom = await fetch(`https://your-api.com/search?q=${keyword}`);
  const customData = await custom.json();
  
  const generator = new ContentGenerator();
  return generator.generateWithClaude({
    keyword,
    format: 'case-study',
    language: 'en',
    tone: 'professional',
    researchData: customData
  });
}
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
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(r => setTimeout(r, 2 ** i * 1000));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

- Ensure FFmpeg is installed: `brew install ffmpeg` (macOS) or `apt-get install ffmpeg` (Linux)
- Check Remotion version compatibility
- Verify output directory exists and has write permissions

### Memory Issues

```typescript
// For large batches, process in chunks
async function processInChunks<T>(
  items: T[],
  processor: (item: T) => Promise<any>,
  chunkSize = 5
) {
  const chunks = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    chunks.push(items.slice(i, i + chunkSize));
  }
  
  for (const chunk of chunks) {
    await Promise.all(chunk.map(processor));
  }
}
```

### Environment Variable Issues

```typescript
// Validate environment on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
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

## Best Practices

1. **Always validate research data** before generating content
2. **Cache research results** to avoid redundant API calls
3. **Use streaming** for long-form content generation
4. **Implement proper error handling** for all external API calls
5. **Monitor API usage** to stay within quota limits
6. **Store generated content** in a database for reuse
7. **Use TypeScript strict mode** for type safety
