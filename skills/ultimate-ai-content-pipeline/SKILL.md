---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with AI research
  - create videos from blog posts automatically
  - use Claude for content generation workflow
  - build automated marketing content system
  - research and generate content with AI
  - set up Remotion video rendering pipeline
  - automate content creation from research to video
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates content creation from research and article generation to automatic video rendering. The pipeline leverages Claude 3, OpenAI, and Remotion to create a complete content production workflow.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates articles** in multiple formats (listicles, POV pieces, case studies, how-tos) using Claude or OpenAI
- **Supports multilingual content** (English and Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multi-platform** distribution (Reels, TikTok, Shorts)

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Package manager
npm --version
# or
yarn --version
```

### Clone and Install

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion (optional)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Key API Routes and Commands

### Starting the Application

```bash
# Development mode with hot reload
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion
```

### Main API Endpoints

The pipeline exposes several Next.js API routes:

**Research Endpoint**
```typescript
// POST /api/research
{
  "keyword": "artificial intelligence trends",
  "sources": ["techcrunch", "twitter", "linkedin"],
  "timeframe": "24h"
}
```

**Content Generation Endpoint**
```typescript
// POST /api/generate-content
{
  "researchData": {...},
  "format": "toplist", // or "pov", "case-study", "how-to"
  "language": "en", // or "vi"
  "tone": "professional", // or "friendly", "humorous"
  "aiProvider": "claude" // or "openai"
}
```

**Video Rendering Endpoint**
```typescript
// POST /api/render-video
{
  "content": "article content",
  "style": "infographic",
  "platform": "reels" // or "tiktok", "shorts"
}
```

## Core Usage Patterns

### 1. Research Data Crawling

```typescript
// lib/research/crawler.ts
import { crawlSources } from '@/lib/research/crawler';

async function fetchLatestNews(keyword: string) {
  const research = await crawlSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    limit: 10,
    timeframe: '24h'
  });
  
  return research;
}

// Returns structured data
interface ResearchData {
  articles: Array<{
    title: string;
    url: string;
    snippet: string;
    publishedAt: string;
    source: string;
  }>;
  insights: string[];
  keywords: string[];
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(
  researchData: ResearchData,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(researchData, format, language);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].text;
}

function buildPrompt(
  data: ResearchData, 
  format: string, 
  language: string
): string {
  const formatInstructions = {
    toplist: 'Create a numbered list article with engaging subheadings',
    pov: 'Write from a first-person perspective with strong opinions',
    'case-study': 'Analyze with data points, examples, and outcomes',
    'how-to': 'Provide step-by-step actionable instructions'
  };
  
  return `
You are a professional content writer. Using the following research data, create a ${format} article in ${language}.

Research Insights:
${data.insights.join('\n')}

Recent Articles:
${data.articles.map(a => `- ${a.title} (${a.source})`).join('\n')}

Format: ${formatInstructions[format]}

Write an engaging, data-backed article that provides unique value.
  `.trim();
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  researchData: ResearchData,
  format: string,
  language: string
) {
  const prompt = buildPrompt(researchData, format, language);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketing writer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  content: string,
  outputPath: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const platformSpecs = {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 }
  };
  
  const specs = platformSpecs[platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      platform
    },
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content,
      platform
    },
    ...specs
  });
  
  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  platform: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  content, 
  platform 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  // Parse content into sections
  const sections = content.split('\n\n');
  const currentSection = Math.floor(frame / (fps * 3)) % sections.length;
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        fontFamily: 'Inter, sans-serif',
        padding: 60,
        justifyContent: 'center',
        alignItems: 'center'
      }}
    >
      <div
        style={{
          fontSize: 48,
          color: 'white',
          textAlign: 'center',
          maxWidth: '80%',
          opacity: Math.min(1, frame / 30)
        }}
      >
        {sections[currentSection]}
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Workflow

```typescript
// lib/pipeline/workflow.ts
import { crawlSources } from '@/lib/research/crawler';
import { generateWithClaude } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/renderer';

async function runContentPipeline(
  keyword: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    language: 'en' | 'vi';
    generateVideo: boolean;
    platform?: 'reels' | 'tiktok' | 'shorts';
  }
) {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const researchData = await crawlSources({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    limit: 10,
    timeframe: '24h'
  });
  
  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const article = await generateWithClaude(
    researchData,
    options.format,
    options.language
  );
  
  // Step 3: Render Video (optional)
  let videoPath = null;
  if (options.generateVideo && options.platform) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo(
      article,
      `./output/${keyword}-${Date.now()}.mp4`,
      options.platform
    );
  }
  
  return {
    article,
    videoPath,
    researchData,
    metadata: {
      keyword,
      format: options.format,
      language: options.language,
      generatedAt: new Date().toISOString()
    }
  };
}

// Usage
const result = await runContentPipeline('AI marketing automation', {
  format: 'toplist',
  language: 'en',
  generateVideo: true,
  platform: 'reels'
});

console.log('Article:', result.article);
console.log('Video:', result.videoPath);
```

### 7. Next.js API Route Implementation

```typescript
// pages/api/pipeline.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/workflow';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, language, generateVideo, platform } = req.body;
    
    if (!keyword) {
      return res.status(400).json({ error: 'Keyword is required' });
    }
    
    const result = await runContentPipeline(keyword, {
      format: format || 'toplist',
      language: language || 'en',
      generateVideo: generateVideo || false,
      platform: platform || 'reels'
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline execution failed',
      details: error.message 
    });
  }
}
```

## Configuration Files

### TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "preserve",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowJs": true,
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

### Package.json Scripts

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "remotion": "remotion studio",
    "render": "remotion render",
    "pipeline": "ts-node scripts/run-pipeline.ts"
  }
}
```

## Troubleshooting

### API Key Issues

```typescript
// lib/utils/validate-env.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

### Rate Limiting

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchGenerate(prompts: string[]) {
  const tasks = prompts.map(prompt => 
    limit(() => generateWithClaude(prompt))
  );
  
  return Promise.all(tasks);
}
```

### Video Rendering Memory Issues

```typescript
// Increase Node memory for large video renders
// package.json
{
  "scripts": {
    "render:large": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### Error Handling Pattern

```typescript
// lib/utils/error-handler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

async function safePipelineRun(keyword: string) {
  try {
    return await runContentPipeline(keyword, {
      format: 'toplist',
      language: 'en',
      generateVideo: false
    });
  } catch (error) {
    if (error.message.includes('API key')) {
      throw new PipelineError(
        'Invalid API credentials',
        'generation',
        error
      );
    }
    
    if (error.message.includes('rate limit')) {
      throw new PipelineError(
        'Rate limit exceeded, retry after 60s',
        'research',
        error
      );
    }
    
    throw error;
  }
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLingual(keyword: string) {
  const languages = ['en', 'vi'] as const;
  
  const articles = await Promise.all(
    languages.map(async (lang) => {
      const content = await generateWithClaude(
        researchData,
        'toplist',
        lang
      );
      
      return { language: lang, content };
    })
  );
  
  return articles;
}
```

### Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI trends', 'marketing automation', 'content creation'];
  
  for (const topic of topics) {
    await runContentPipeline(topic, {
      format: 'toplist',
      language: 'en',
      generateVideo: true,
      platform: 'reels'
    });
  }
});
```

This skill provides comprehensive guidance for using the Ultimate AI Content Pipeline to automate content creation workflows from research through video generation.
