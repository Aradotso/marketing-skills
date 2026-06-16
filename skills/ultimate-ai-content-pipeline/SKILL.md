---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do i set up the ai content pipeline
  - generate automated content with ai research
  - create videos from content using remotion
  - automate content workflow with claude and openai
  - build ai powered content generation system
  - set up automated content research and publishing
  - use the marketing pipeline for content creation
  - configure ai content automation with video rendering
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based automated content creation system that handles research, scriptwriting, and video generation using Claude/OpenAI and Remotion.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case studies, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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
# Using npm
npm install

# Using yarn
yarn install

# Using pnpm
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research and scraping
│   │   ├── video/       # Remotion video generation
│   │   └── content/     # Content generation logic
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core API Usage

### 1. Content Research

```typescript
import { researchTopic } from '@/lib/research/crawler';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 10
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics
  };
}

// Usage
const data = await gatherResearch('AI automation');
console.log(data.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(research: any, format: string) {
  const prompt = `
Based on this research data:
${JSON.stringify(research, null, 2)}

Create a ${format} article that:
- Uses the latest insights and statistics
- Maintains a professional yet engaging tone
- Includes actionable takeaways
- Is optimized for social media sharing
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(topic: string, language: 'en' | 'vi') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator. Generate content in ${language === 'en' ? 'English' : 'Vietnamese'}.`
      },
      {
        role: 'user',
        content: `Create engaging content about: ${topic}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: string, outputPath: string) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      title: 'Generated Content',
      duration: 30
    }
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.props,
  });

  return outputPath;
}

// Usage for different platforms
await generateVideo(content, './output/reels.mp4'); // 9:16 aspect
await generateVideo(content, './output/youtube.mp4'); // 16:9 aspect
```

## Complete Workflow Example

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/renderer';
import { publishToSocial } from '@/lib/publish';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });

    // Step 2: Generate content with AI
    console.log('✍️ Generating content...');
    const content = await generateContent(research, 'toplist');
    
    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo(content, './output/video.mp4');

    // Step 4: Publish (optional)
    console.log('📤 Publishing...');
    await publishToSocial({
      content,
      videoPath,
      platforms: ['facebook', 'instagram', 'tiktok']
    });

    return {
      success: true,
      content,
      videoPath
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute
runContentPipeline('AI automation trends 2024');
```

## Content Format Templates

### Toplist Format

```typescript
interface ToplistConfig {
  title: string;
  items: number;
  includeStats: boolean;
  tone: 'professional' | 'casual' | 'expert';
}

async function generateToplist(research: any, config: ToplistConfig) {
  const prompt = `
Create a toplist article with ${config.items} items about ${config.title}.
Tone: ${config.tone}
Include statistics: ${config.includeStats}
Research data: ${JSON.stringify(research)}
`;

  return await generateContent(research, prompt);
}
```

### Case Study Format

```typescript
async function generateCaseStudy(research: any, company: string) {
  const prompt = `
Create a detailed case study about ${company}'s approach.
Include:
- Challenge/Problem
- Solution implemented
- Results with metrics
- Key takeaways
Use data: ${JSON.stringify(research)}
`;

  return await generateContent(research, prompt);
}
```

## Next.js API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: Request) {
  const { keyword, sources, timeframe } = await request.json();

  try {
    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h'
    });

    return NextResponse.json({ success: true, data: research });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(request: Request) {
  const { research, format, language } = await request.json();

  try {
    const content = await generateContent(research, format, language);
    
    return NextResponse.json({ 
      success: true, 
      content 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// app/api/render-video/route.ts
import { NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/renderer';

export async function POST(request: Request) {
  const { content, platform } = await request.json();

  try {
    const videoPath = await generateVideo(content, platform);
    
    return NextResponse.json({ 
      success: true, 
      videoPath,
      downloadUrl: `/api/download?path=${videoPath}`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
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

# Run type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video preview
npm run remotion:preview

# Render video directly
npm run remotion:render
```

## Configuration

### Remotion Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');

// Platform-specific configs
export const PLATFORM_CONFIGS = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30
  },
  youtube: {
    width: 1920,
    height: 1080,
    fps: 60
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30
  }
};
```

### AI Model Configuration

```typescript
// lib/config/ai.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 2000,
    temperature: 0.7
  }
};
```

## Common Patterns

### Rate Limiting for API Calls

```typescript
class RateLimiter {
  private lastCall: number = 0;
  private minInterval: number;

  constructor(callsPerMinute: number) {
    this.minInterval = 60000 / callsPerMinute;
  }

  async throttle() {
    const now = Date.now();
    const timeSinceLastCall = now - this.lastCall;
    
    if (timeSinceLastCall < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastCall)
      );
    }
    
    this.lastCall = Date.now();
  }
}

const aiLimiter = new RateLimiter(20); // 20 calls per minute

async function callAI(prompt: string) {
  await aiLimiter.throttle();
  return await generateContent(prompt);
}
```

### Error Handling and Retry Logic

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      console.log(`Attempt ${i + 1} failed, retrying...`);
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => generateContent(research, 'toplist'));
```

### Batch Processing

```typescript
async function processBatch(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline(keyword);
      results.push({ keyword, success: true, data: result });
    } catch (error) {
      results.push({ keyword, success: false, error: error.message });
    }
    
    // Add delay between batches
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}

validateEnv();
```

### Remotion Rendering Errors

```bash
# Clear Remotion cache
npx remotion clear-cache

# Check ffmpeg installation
ffmpeg -version

# Install ffmpeg if missing (Mac)
brew install ffmpeg

# Install ffmpeg (Ubuntu)
sudo apt-get install ffmpeg
```

### Memory Issues with Large Videos

```typescript
// Adjust Node.js memory limit
// package.json scripts:
{
  "scripts": {
    "render": "NODE_OPTIONS='--max-old-space-size=4096' npm run remotion:render"
  }
}
```

### Rate Limit Handling

```typescript
async function handleRateLimit(error: any) {
  if (error.status === 429) {
    const retryAfter = error.headers?.['retry-after'] || 60;
    console.log(`Rate limited. Waiting ${retryAfter}s...`);
    await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
    return true; // Retry
  }
  return false; // Don't retry
}
```

### Debug Mode

```typescript
// Enable debug logging
export const DEBUG = process.env.DEBUG === 'true';

function log(...args: any[]) {
  if (DEBUG) {
    console.log('[DEBUG]', new Date().toISOString(), ...args);
  }
}

// Usage
log('Research completed:', research);
```

## Best Practices

1. **Always validate environment variables before running the pipeline**
2. **Use rate limiting to avoid API quota exhaustion**
3. **Implement proper error handling and logging**
4. **Cache research results to reduce API calls**
5. **Test video rendering with short durations first**
6. **Use TypeScript for type safety across the pipeline**
7. **Monitor API usage and costs regularly**
