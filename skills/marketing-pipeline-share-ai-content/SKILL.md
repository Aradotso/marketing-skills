---
name: marketing-pipeline-share-ai-content
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video
  - set up marketing pipeline for auto content generation
  - create AI content workflow from research to video
  - build automated content system with Claude and Remotion
  - generate videos and articles automatically from keywords
  - implement AI-powered marketing content pipeline
  - use marketing-pipeline-share for content automation
  - set up auto research and video generation workflow
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end AI content automation system that transforms a single keyword into fully-researched articles and videos. The pipeline automatically crawls latest news from TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats using Claude/OpenAI, and renders videos using Remotion.

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if applicable)
DATABASE_URL=your_database_connection_string

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Video rendering (Remotion)
npm run render
```

## Core Architecture

### 1. Research Module (Auto-Scan)

Crawls and analyzes content from multiple sources:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  name: string;
  url: string;
  parser: (data: any) => ResearchItem[];
}

interface ResearchItem {
  title: string;
  content: string;
  url: string;
  publishedAt: Date;
  source: string;
}

export async function crawlNews(keyword: string, timeframe: number = 24): Promise<ResearchItem[]> {
  const sources: ResearchSource[] = [
    {
      name: 'TechCrunch',
      url: `https://api.rapidapi.com/techcrunch/search?q=${encodeURIComponent(keyword)}`,
      parser: parseTechCrunch
    },
    {
      name: 'Twitter',
      url: `https://api.rapidapi.com/twitter/search?q=${encodeURIComponent(keyword)}`,
      parser: parseTwitter
    }
  ];

  const results = await Promise.all(
    sources.map(async (source) => {
      try {
        const response = await axios.get(source.url, {
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
            'X-RapidAPI-Host': 'auto-research.p.rapidapi.com'
          }
        });
        return source.parser(response.data);
      } catch (error) {
        console.error(`Error fetching from ${source.name}:`, error);
        return [];
      }
    })
  );

  return results.flat().filter(item => isRecent(item.publishedAt, timeframe));
}

function isRecent(date: Date, hoursAgo: number): boolean {
  const cutoff = new Date(Date.now() - hoursAgo * 60 * 60 * 1000);
  return date >= cutoff;
}
```

### 2. Content Generation with AI

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: ResearchItem[];
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(request);

  if (provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

async function generateWithOpenAI(prompt: string): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator and marketer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    max_tokens: 4096
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a unique perspective with strong opinions and insights',
    'case-study': 'Analyze real examples with data, outcomes, and lessons learned',
    'how-to': 'Provide step-by-step instructions with actionable advice'
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Write in a conversational, approachable style',
    'humorous': 'Include wit and humor while maintaining value'
  };

  const researchContext = request.researchData
    .map(item => `- ${item.title} (${item.source}): ${item.content.substring(0, 200)}...`)
    .join('\n');

  return `
Create a ${request.format} article about "${request.keyword}" in ${request.language === 'en' ? 'English' : 'Vietnamese'}.

TONE: ${toneInstructions[request.tone]}
FORMAT: ${formatInstructions[request.format]}

RESEARCH DATA (use this to make the content data-backed and current):
${researchContext}

Requirements:
- Include specific examples and data from the research
- Make it engaging and valuable for the target audience
- Optimize for SEO with natural keyword usage
- Length: 1500-2000 words
- Include a compelling headline and subheadings
`;
}
```

### 3. Video Generation with Remotion

Automatically render videos from generated content:

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { z } from 'zod';

export const VideoSchema = z.object({
  title: z.string(),
  points: z.array(z.object({
    heading: z.string(),
    content: z.string()
  })),
  brandColor: z.string().default('#FF6B6B')
});

export const VideoComposition: React.FC<z.infer<typeof VideoSchema>> = ({
  title,
  points,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: 'white' }}>
      <Sequence from={0} durationInFrames={fps * 3}>
        <TitleScene title={title} brandColor={brandColor} />
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 4)}
          durationInFrames={fps * 4}
        >
          <PointScene point={point} brandColor={brandColor} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const TitleScene: React.FC<{ title: string; brandColor: string }> = ({
  title,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        opacity
      }}
    >
      <h1 style={{ color: brandColor, fontSize: 64, textAlign: 'center', padding: 40 }}>
        {title}
      </h1>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderVideo(
  title: string,
  points: Array<{ heading: string; content: string }>,
  outputPath: string
): Promise<string> {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const compositionId = 'VideoComposition';
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: { title, points, brandColor: '#FF6B6B' }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: { title, points, brandColor: '#FF6B6B' }
  });

  return outputPath;
}
```

### 4. Full Pipeline Orchestration

Combine all modules into a complete workflow:

```typescript
// lib/pipeline/orchestrator.ts
import { crawlNews } from '../research/crawler';
import { generateContent } from '../ai/content-generator';
import { renderVideo } from '../video/render';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  generateVideo: boolean;
  aiProvider: 'claude' | 'openai';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);

  // Step 1: Research
  console.log('Step 1: Crawling research data...');
  const researchData = await crawlNews(config.keyword, 24);
  console.log(`Found ${researchData.length} relevant articles`);

  // Step 2: Generate Content
  console.log('Step 2: Generating content with AI...');
  const content = await generateContent(
    {
      keyword: config.keyword,
      format: config.format,
      language: config.language,
      tone: config.tone,
      researchData
    },
    config.aiProvider
  );

  // Step 3: Extract key points for video
  const points = extractKeyPoints(content);

  // Step 4: Generate Video (optional)
  let videoPath: string | null = null;
  if (config.generateVideo) {
    console.log('Step 3: Rendering video...');
    videoPath = await renderVideo(
      config.keyword,
      points,
      `./output/video-${Date.now()}.mp4`
    );
    console.log(`Video rendered: ${videoPath}`);
  }

  return {
    content,
    researchData,
    videoPath,
    metadata: {
      keyword: config.keyword,
      format: config.format,
      language: config.language,
      generatedAt: new Date()
    }
  };
}

function extractKeyPoints(content: string): Array<{ heading: string; content: string }> {
  // Simple extraction - parse markdown headings and their content
  const sections = content.split(/^##\s+/gm).filter(Boolean);
  
  return sections.slice(0, 5).map(section => {
    const [heading, ...contentLines] = section.split('\n');
    return {
      heading: heading.trim(),
      content: contentLines.join('\n').trim().substring(0, 200)
    };
  });
}
```

## API Routes (Next.js)

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
      language: body.language || 'en',
      tone: body.tone || 'friendly',
      generateVideo: body.generateVideo || false,
      aiProvider: body.aiProvider || 'claude'
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

### Pattern 1: Quick Content Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

// Generate a toplist article in English
const result = await runContentPipeline({
  keyword: 'AI trends 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  generateVideo: false,
  aiProvider: 'claude'
});

console.log(result.content);
```

### Pattern 2: Bilingual Content with Video

```typescript
// Generate both English and Vietnamese versions with video
const [enResult, viResult] = await Promise.all([
  runContentPipeline({
    keyword: 'Marketing automation',
    format: 'how-to',
    language: 'en',
    tone: 'friendly',
    generateVideo: true,
    aiProvider: 'openai'
  }),
  runContentPipeline({
    keyword: 'Marketing automation',
    format: 'how-to',
    language: 'vi',
    tone: 'friendly',
    generateVideo: false,
    aiProvider: 'claude'
  })
]);
```

### Pattern 3: Custom Research Timeframe

```typescript
import { crawlNews } from '@/lib/research/crawler';

// Get research from the last 48 hours
const recentNews = await crawlNews('startup funding', 48);

// Use with custom content generation
const content = await generateContent({
  keyword: 'startup funding trends',
  format: 'case-study',
  language: 'en',
  tone: 'expert',
  researchData: recentNews
}, 'claude');
```

## Troubleshooting

### API Rate Limits

If you hit rate limits with Claude or OpenAI:

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateWithClaude(prompt);
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
}
```

### Video Rendering Issues

If Remotion fails to render:

```bash
# Ensure all dependencies are installed
npm install @remotion/bundler @remotion/renderer @remotion/cli

# Check FFmpeg installation
npx remotion versions

# Render with verbose logging
npx remotion render src/index.tsx VideoComposition output.mp4 --log=verbose
```

### Research Data Quality

Filter and validate research results:

```typescript
function validateResearchItem(item: ResearchItem): boolean {
  return (
    item.content.length > 100 &&
    item.title.length > 10 &&
    !item.content.includes('404') &&
    !item.content.includes('error')
  );
}

const validResearch = researchData.filter(validateResearchItem);
```

## Performance Optimization

```typescript
// Cache research results
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!
});

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) {
    return JSON.parse(cached as string);
  }
  
  const fresh = await crawlNews(keyword);
  await redis.setex(cacheKey, 3600, JSON.stringify(fresh)); // Cache for 1 hour
  return fresh;
}
```
