---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, scripting, video generation and multi-platform publishing
triggers:
  - automate content creation pipeline
  - generate AI content with research
  - create videos from text automatically
  - build content automation workflow
  - set up AI content generation system
  - automate research to video pipeline
  - create multi-format content with AI
  - build marketing content automation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that researches topics, generates multi-format content (articles, scripts), and renders videos automatically. Integrates Claude 3, OpenAI, web scraping, and Remotion for end-to-end content production.

## What It Does

- **Auto-Research**: Crawls news sources (TechCrunch, Twitter, LinkedIn) for trending topics
- **Multi-Format Content**: Generates articles in various formats (listicles, case studies, how-tos)
- **Bilingual Output**: Creates content in both English and Vietnamese
- **Video Generation**: Renders videos/infographics from content using Remotion
- **Platform Optimization**: Outputs video in formats for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Web Scraping (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using Supabase/Prisma)
DATABASE_URL=your_database_url_here

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Project

```bash
# Development mode
npm run dev
# or
yarn dev

# Build for production
npm run build

# Start production server
npm start

# Run video rendering server (Remotion)
npm run remotion
```

## Core Components

### 1. Research Module

Automatically scrapes and analyzes content from multiple sources:

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}

export async function researchTopic(
  keyword: string,
  sources: string[] = ['techcrunch', 'twitter', 'linkedin']
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const data = await scrapeSource(source, keyword);
    const analyzed = await analyzeWithAI(data);
    results.push(...analyzed);
  }
  
  return results;
}

async function scrapeSource(source: string, keyword: string) {
  const options = {
    method: 'GET',
    url: `https://api.rapidapi.com/${source}/search`,
    params: { q: keyword, timeframe: '24h' },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      'X-RapidAPI-Host': `${source}.p.rapidapi.com`
    }
  };
  
  const response = await axios.request(options);
  return response.data;
}
```

### 2. Content Generation with AI

Generate multi-format content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'professional' | 'friendly' | 'humorous';

interface ContentRequest {
  topic: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: ResearchResult[];
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(request);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
    
    const message = await anthropic.messages.create({
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
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear points',
    'pov': 'Write from a unique perspective or angle',
    'case-study': 'Present a detailed analysis with examples',
    'how-to': 'Create a step-by-step tutorial'
  };
  
  const researchContext = request.researchData
    .map(r => `- ${r.title}: ${r.content.substring(0, 200)}...`)
    .join('\n');
  
  return `
You are a ${request.tone} content writer. 
Create a ${request.format} article about "${request.topic}" in ${request.language}.

${formatInstructions[request.format]}

Use this recent research data:
${researchContext}

Requirements:
- Include data and statistics from the research
- Make it engaging and actionable
- Optimize for social media sharing
- Add relevant hashtags
  `.trim();
}
```

### 3. Video Rendering with Remotion

Transform content into videos:

```typescript
// remotion/compositions/ContentVideo.tsx
import { Composition } from 'remotion';
import { z } from 'zod';

export const ContentVideoSchema = z.object({
  title: z.string(),
  points: z.array(z.string()),
  duration: z.number(),
  platform: z.enum(['reels', 'tiktok', 'shorts'])
});

export type ContentVideoProps = z.infer<typeof ContentVideoSchema>;

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  platform
}) => {
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  return (
    <div style={{
      width: dimensions[platform].width,
      height: dimensions[platform].height,
      background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
      display: 'flex',
      flexDirection: 'column',
      justifyContent: 'center',
      padding: 60
    }}>
      <h1 style={{ 
        fontSize: 72, 
        color: 'white',
        marginBottom: 40,
        textAlign: 'center'
      }}>
        {title}
      </h1>
      
      {points.map((point, i) => (
        <div key={i} style={{
          fontSize: 36,
          color: 'white',
          marginBottom: 20,
          opacity: 0.9
        }}>
          {i + 1}. {point}
        </div>
      ))}
    </div>
  );
};
```

Render video programmatically:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: string,
  platform: 'reels' | 'tiktok' | 'shorts'
): Promise<string> {
  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: extractTitle(content),
      points: extractPoints(content),
      duration: 30,
      platform
    }
  });
  
  const outputPath = path.join(
    process.cwd(), 
    'public', 
    'videos', 
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps
  });
  
  return outputPath;
}

function extractTitle(content: string): string {
  const lines = content.split('\n');
  return lines[0].replace(/^#+ /, '');
}

function extractPoints(content: string): string[] {
  const lines = content.split('\n');
  return lines
    .filter(line => /^\d+\./.test(line))
    .map(line => line.replace(/^\d+\.\s*/, ''))
    .slice(0, 5);
}
```

### 4. Complete Pipeline

Orchestrate the full workflow:

```typescript
// lib/pipeline/orchestrator.ts
import { researchTopic } from '../research/scraper';
import { generateContent } from '../ai/content-generator';
import { renderContentVideo } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(
  config: PipelineConfig
) {
  console.log('🔍 Starting research...');
  const researchData = await researchTopic(config.keyword);
  
  console.log('✍️ Generating content...');
  const content = await generateContent({
    topic: config.keyword,
    format: config.format,
    language: config.language,
    tone: config.tone,
    researchData
  });
  
  let videoPath: string | null = null;
  if (config.generateVideo && config.platform) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo(content, config.platform);
  }
  
  return {
    content,
    videoPath,
    research: researchData,
    metadata: {
      createdAt: new Date(),
      format: config.format,
      language: config.language
    }
  };
}
```

### 5. API Route Example

Create a Next.js API endpoint:

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, language, tone, generateVideo, platform } = req.body;
    
    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      tone: tone || 'professional',
      generateVideo: generateVideo || false,
      platform: platform || 'reels'
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Failed to generate content',
      details: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

## Common Usage Patterns

### Quick Content Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

// Generate a toplist article in English
const result = await runContentPipeline({
  keyword: 'AI marketing trends 2024',
  format: 'toplist',
  language: 'en',
  tone: 'professional',
  generateVideo: false
});

console.log(result.content);
```

### Generate with Video

```typescript
// Generate content + video for TikTok
const result = await runContentPipeline({
  keyword: '5 AI tools for marketers',
  format: 'toplist',
  language: 'en',
  tone: 'friendly',
  generateVideo: true,
  platform: 'tiktok'
});

console.log('Content:', result.content);
console.log('Video:', result.videoPath);
```

### Bilingual Content

```typescript
// Generate same content in both languages
const englishContent = await generateContent({
  topic: 'Content Marketing Automation',
  format: 'how-to',
  language: 'en',
  tone: 'professional',
  researchData: []
});

const vietnameseContent = await generateContent({
  topic: 'Content Marketing Automation',
  format: 'how-to',
  language: 'vi',
  tone: 'professional',
  researchData: []
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(
  request: ContentRequest,
  maxRetries: number = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request);
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Use smaller compositions or split rendering
const composition = await selectComposition({
  serveUrl: bundled,
  id: 'ContentVideo',
  inputProps: {
    // Limit points to avoid memory issues
    points: extractPoints(content).slice(0, 3),
    duration: 15 // Shorter duration
  }
});
```

### Missing Environment Variables

```typescript
// Add validation at startup
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

// Call in your app entry point
validateEnv();
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearchData(data: ResearchResult[]): ResearchResult[] {
  return data.filter(result => 
    result.content.length > 100 && // Minimum content length
    result.insights.length > 0 && // Has insights
    result.publishedAt > new Date(Date.now() - 48 * 60 * 60 * 1000) // Within 48h
  );
}
```

## Best Practices

1. **Cache Research Data**: Store results to avoid repeated API calls
2. **Queue Video Rendering**: Use a job queue for heavy rendering tasks
3. **Content Moderation**: Review AI-generated content before publishing
4. **Error Logging**: Implement comprehensive error tracking
5. **Rate Limiting**: Respect API limits with proper throttling
