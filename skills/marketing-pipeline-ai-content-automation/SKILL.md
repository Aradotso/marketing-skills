---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline
  - generate videos from articles automatically
  - crawl news sources for content research
  - create multilingual content with Claude
  - automate social media content workflow
  - build an AI marketing content system
  - use Remotion for automated video generation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers use the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating articles in multiple formats and languages, to rendering videos for social media.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

- **Auto-scans** news sources (TechCrunch, a16z, Twitter/X, LinkedIn) for trending topics
- **Generates content** in multiple formats (Top Lists, POV, Case Studies, How-tos) using Claude 3 or OpenAI
- **Supports bilingual output** (English and Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for TikTok, Reels, and YouTube Shorts
- **Provides a Next.js interface** for managing the entire pipeline

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

### Environment Variables

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Config
REMOTION_LICENSE_KEY=your_remotion_license_key

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run remotion
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video generation
│   ├── remotion/        # Remotion video templates
│   └── types/           # TypeScript types
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core Functionality

### 1. Research & Crawling

The system automatically crawls news sources for trending topics:

```typescript
// src/lib/crawler/news-scanner.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  summary: string;
}

export async function scanTrendingNews(
  keywords: string[],
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const apiKey = process.env.RAPIDAPI_KEY;
  if (!apiKey) throw new Error('RAPIDAPI_KEY not configured');

  const client = new RapidAPIClient(apiKey);
  const articles: NewsArticle[] = [];

  for (const source of sources) {
    const results = await client.searchNews({
      query: keywords.join(' OR '),
      sources: [source],
      timeframe: '24h'
    });
    
    articles.push(...results);
  }

  return articles;
}

// Usage in your pipeline
const trendingTopics = await scanTrendingNews(
  ['AI', 'marketing automation', 'content creation']
);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Tone = 'expert' | 'friendly' | 'humorous';
type Language = 'en' | 'vi';

interface GenerateContentParams {
  topic: string;
  format: ContentFormat;
  tone: Tone;
  language: Language;
  researchData: string;
}

export async function generateContent(
  params: GenerateContentParams,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const { topic, format, tone, language, researchData } = params;

  const systemPrompt = buildSystemPrompt(format, tone, language);
  const userPrompt = `
Topic: ${topic}

Research Data:
${researchData}

Please create a ${format} article based on this research. 
Make it engaging, data-driven, and optimized for social media sharing.
`;

  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });

    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      system: systemPrompt,
      messages: [
        { role: 'user', content: userPrompt }
      ],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });

    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt }
      ],
    });

    return completion.choices[0].message.content || '';
  }
}

function buildSystemPrompt(
  format: ContentFormat,
  tone: Tone,
  language: Language
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a specific point of view with strong opinions',
    'case-study': 'Analyze a real example with data and outcomes',
    'how-to': 'Provide step-by-step instructions',
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Write in a conversational, approachable style',
    'humorous': 'Include wit and humor while staying informative',
  };

  return `You are an expert content creator specializing in ${format} articles.
Tone: ${toneInstructions[tone]}
Language: ${language === 'en' ? 'English' : 'Vietnamese'}
Format: ${formatInstructions[format]}

Always include:
- Attention-grabbing headline
- Data-backed insights
- Actionable takeaways
- Social media-optimized formatting`;
}
```

### 3. Bilingual Content Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
// src/lib/content/bilingual-generator.ts
import { generateContent, GenerateContentParams } from '@/lib/ai/content-generator';

interface BilingualContent {
  english: string;
  vietnamese: string;
}

export async function generateBilingualContent(
  baseParams: Omit<GenerateContentParams, 'language'>
): Promise<BilingualContent> {
  const [english, vietnamese] = await Promise.all([
    generateContent({ ...baseParams, language: 'en' }),
    generateContent({ ...baseParams, language: 'vi' })
  ]);

  return { english, vietnamese };
}

// Usage
const content = await generateBilingualContent({
  topic: 'AI in Marketing Automation',
  format: 'how-to',
  tone: 'friendly',
  researchData: trendingArticles.map(a => a.summary).join('\n\n')
});
```

### 4. Video Generation with Remotion

Create videos from content automatically:

```typescript
// src/remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import React from 'react';

interface ArticleVideoProps {
  title: string;
  keyPoints: string[];
  bgColor: string;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  keyPoints,
  bgColor
}) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{
          justifyContent: 'center',
          alignItems: 'center',
          padding: 60
        }}>
          <h1 style={{
            fontSize: 60,
            fontWeight: 'bold',
            textAlign: 'center',
            opacity: Math.min(1, frame / 30)
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={90 + index * 120}
          durationInFrames={120}
        >
          <AbsoluteFill style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60
          }}>
            <div style={{
              fontSize: 40,
              textAlign: 'center',
              opacity: Math.min(1, (frame - (90 + index * 120)) / 20)
            }}>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderVideoParams {
  title: string;
  keyPoints: string[];
  outputPath: string;
  format: 'tiktok' | 'reels' | 'youtube-shorts';
}

const ASPECT_RATIOS = {
  'tiktok': { width: 1080, height: 1920 },
  'reels': { width: 1080, height: 1920 },
  'youtube-shorts': { width: 1080, height: 1920 }
};

export async function renderContentVideo(
  params: RenderVideoParams
): Promise<string> {
  const { title, keyPoints, outputPath, format } = params;
  const { width, height } = ASPECT_RATIOS[format];

  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: {
      title,
      keyPoints,
      bgColor: '#1a1a1a'
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      keyPoints,
      bgColor: '#1a1a1a'
    },
    imageFormat: 'jpeg',
  });

  return outputPath;
}
```

### 5. Complete Pipeline Orchestration

Tie everything together in a full automation pipeline:

```typescript
// src/lib/pipeline/content-pipeline.ts
import { scanTrendingNews } from '@/lib/crawler/news-scanner';
import { generateBilingualContent } from '@/lib/content/bilingual-generator';
import { renderContentVideo } from '@/lib/video/render-video';
import { extractKeyPoints } from '@/lib/content/parser';

interface PipelineConfig {
  keywords: string[];
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  videoFormats: ('tiktok' | 'reels' | 'youtube-shorts')[];
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('🔍 Step 1: Scanning trending news...');
  const articles = await scanTrendingNews(config.keywords);
  
  const researchData = articles
    .map(a => `${a.title}\n${a.summary}`)
    .join('\n\n---\n\n');

  console.log('✍️ Step 2: Generating bilingual content...');
  const content = await generateBilingualContent({
    topic: config.keywords.join(', '),
    format: config.format,
    tone: config.tone,
    researchData
  });

  console.log('🎬 Step 3: Rendering videos...');
  const keyPoints = extractKeyPoints(content.english);
  
  const videoPromises = config.videoFormats.map(format =>
    renderContentVideo({
      title: content.english.split('\n')[0], // First line as title
      keyPoints,
      outputPath: `./output/video-${format}-${Date.now()}.mp4`,
      format
    })
  );

  const videoPaths = await Promise.all(videoPromises);

  return {
    content,
    videos: videoPaths,
    research: articles
  };
}

// Usage
const result = await runContentPipeline({
  keywords: ['AI marketing', 'automation'],
  format: 'how-to',
  tone: 'friendly',
  videoFormats: ['tiktok', 'reels']
});

console.log('✅ Pipeline complete!');
console.log('English content:', result.content.english);
console.log('Vietnamese content:', result.content.vietnamese);
console.log('Videos:', result.videos);
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keywords, format, tone, videoFormats } = body;

    const result = await runContentPipeline({
      keywords,
      format,
      tone,
      videoFormats
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Schedule Content Generation

```typescript
// src/lib/scheduler/cron-pipeline.ts
import cron from 'node-cron';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export function scheduleContentGeneration() {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('⏰ Running scheduled content generation...');
    
    await runContentPipeline({
      keywords: ['marketing trends', 'AI tools'],
      format: 'toplist',
      tone: 'expert',
      videoFormats: ['tiktok', 'reels', 'youtube-shorts']
    });
  });
}
```

### Content Quality Validation

```typescript
// src/lib/content/validator.ts
export function validateContent(content: string): boolean {
  const minLength = 500;
  const hasHeading = /^#\s+.+/m.test(content);
  const hasData = /\d+%|\d+\s+(users|people|companies)/i.test(content);
  
  return (
    content.length >= minLength &&
    hasHeading &&
    hasData
  );
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff for API calls
async function callWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
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
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

Reduce concurrent renders and increase Node memory:

```bash
# In package.json scripts
"remotion": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
```

### Missing Environment Variables

```typescript
// src/lib/config/validate-env.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(v => !process.env[v]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

This skill enables AI coding agents to help developers set up and use the Marketing Pipeline for automated content creation, from research to video generation.
