---
name: ultimate-ai-content-pipeline
description: Automated content pipeline that researches, generates scripts, and creates videos using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I set up the AI content automation pipeline
  - generate automated content with research and video
  - create content pipeline with Claude and OpenAI
  - automate content from research to video generation
  - use the marketing content pipeline system
  - build automated content workflow with AI
  - set up content generation with Remotion video
  - configure AI content research and scripting
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a complete content automation system that handles the entire content creation workflow: from researching trending topics (crawling TechCrunch, a16z, Twitter, LinkedIn), generating scripts in multiple formats and languages, to automatically rendering videos using Remotion. Built with TypeScript, Next.js, and integrates with Claude 3, OpenAI, and RapidAPI.

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
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for content research/crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Content research & crawling
│   │   ├── generator/   # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Content Research & Crawling

The system automatically scans sources for trending topics:

```typescript
// src/lib/research/scanner.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchResult {
  title: string;
  url: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}

export async function scanTrendingTopics(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const articles = await client.searchArticles({
      query: keyword,
      source: source,
      timeRange: '24h'
    });
    
    results.push(...articles);
  }
  
  return results;
}

// Extract insights from research
export async function extractInsights(
  research: ResearchResult[]
): Promise<string[]> {
  const insights = research.flatMap(r => r.insights);
  return [...new Set(insights)]; // Remove duplicates
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/lib/generator/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  research: ResearchResult[];
  provider?: 'claude' | 'openai';
}

export async function generateContent(
  options: GenerateContentOptions
): Promise<string> {
  const {
    keyword,
    format,
    language,
    tone,
    research,
    provider = 'claude'
  } = options;
  
  const researchSummary = research
    .map(r => `- ${r.title}: ${r.content.substring(0, 200)}...`)
    .join('\n');
  
  const prompt = buildPrompt(keyword, format, language, tone, researchSummary);
  
  if (provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
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
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

async function generateWithOpenAI(prompt: string): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
    max_tokens: 4096,
  });
  
  return completion.choices[0]?.message?.content || '';
}

function buildPrompt(
  keyword: string,
  format: ContentFormat,
  language: Language,
  tone: Tone,
  research: string
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article',
    'pov': 'Write from a specific point of view or perspective',
    'case-study': 'Analyze as a detailed case study with data',
    'how-to': 'Create a step-by-step tutorial guide'
  };
  
  const languageInstructions = {
    'en': 'Write in English',
    'vi': 'Write in Vietnamese'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative tone',
    'friendly': 'Use conversational, approachable tone',
    'humorous': 'Use witty, engaging tone with humor'
  };
  
  return `
You are a professional content creator. ${formatInstructions[format]} about "${keyword}".

${languageInstructions[language]}.
${toneInstructions[tone]}.

Use this recent research as reference:
${research}

Requirements:
- Include data and statistics from the research
- Make it actionable and valuable
- Optimize for engagement
- Include a compelling headline
- Add relevant hashtags at the end
`;
}
```

### 3. Video Generation with Remotion

Convert content into video format:

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  aspectRatio: '9:16' | '16:9' | '1:1'; // TikTok/Reels, YouTube, Instagram
  duration: number; // in seconds
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const { content, title, aspectRatio, duration } = config;
  
  // Bundle Remotion composition
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
      title,
      aspectRatio,
    },
  });
  
  // Calculate dimensions based on aspect ratio
  const dimensions = getVideoDimensions(aspectRatio);
  
  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content,
      title,
      aspectRatio,
    },
    ...dimensions,
  });
  
  return outputPath;
}

function getVideoDimensions(aspectRatio: string) {
  switch (aspectRatio) {
    case '9:16': // Vertical (TikTok, Reels)
      return { width: 1080, height: 1920 };
    case '16:9': // Horizontal (YouTube)
      return { width: 1920, height: 1080 };
    case '1:1': // Square (Instagram)
      return { width: 1080, height: 1080 };
    default:
      return { width: 1920, height: 1080 };
  }
}
```

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  aspectRatio,
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
        padding: 60,
      }}
    >
      <div style={{ opacity }}>
        <h1
          style={{
            color: 'white',
            fontSize: aspectRatio === '9:16' ? 48 : 72,
            textAlign: 'center',
            marginBottom: 40,
            fontWeight: 'bold',
          }}
        >
          {title}
        </h1>
        <p
          style={{
            color: '#e0e0e0',
            fontSize: aspectRatio === '9:16' ? 24 : 32,
            textAlign: 'center',
            lineHeight: 1.6,
            maxWidth: '80%',
          }}
        >
          {content.substring(0, 300)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline Integration

Combine all features into a single workflow:

```typescript
// src/lib/pipeline/content-pipeline.ts
import { scanTrendingTopics, extractInsights } from '@/lib/research/scanner';
import { generateContent } from '@/lib/generator/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  generateVideo?: boolean;
  videoAspectRatio?: '9:16' | '16:9' | '1:1';
}

export async function runContentPipeline(
  config: PipelineConfig
) {
  const {
    keyword,
    format,
    language,
    tone,
    generateVideo = false,
    videoAspectRatio = '9:16',
  } = config;
  
  // Step 1: Research
  console.log('📡 Scanning trending topics...');
  const research = await scanTrendingTopics(keyword);
  const insights = await extractInsights(research);
  
  // Step 2: Generate Content
  console.log('🧠 Generating content...');
  const content = await generateContent({
    keyword,
    format,
    language,
    tone,
    research,
    provider: 'claude',
  });
  
  // Step 3: Generate Video (optional)
  let videoPath: string | null = null;
  if (generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo({
      content,
      title: keyword,
      aspectRatio: videoAspectRatio,
      duration: 30,
    });
  }
  
  return {
    content,
    insights,
    research: research.slice(0, 5), // Top 5 sources
    videoPath,
    metadata: {
      keyword,
      format,
      language,
      tone,
      generatedAt: new Date(),
    },
  };
}
```

## API Routes (Next.js)

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      language: body.language || 'en',
      tone: body.tone || 'friendly',
      generateVideo: body.generateVideo || false,
      videoAspectRatio: body.videoAspectRatio || '9:16',
    });
    
    return NextResponse.json({
      success: true,
      data: result,
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

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Render Remotion video standalone
npm run remotion:render
```

## Common Usage Patterns

### Pattern 1: Quick Content Generation

```typescript
// Generate content without video
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
});

console.log(result.content);
```

### Pattern 2: Full Pipeline with Video

```typescript
// Generate content + video for social media
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  format: 'how-to',
  language: 'vi',
  tone: 'friendly',
  generateVideo: true,
  videoAspectRatio: '9:16', // For TikTok/Reels
});

console.log('Content:', result.content);
console.log('Video:', result.videoPath);
```

### Pattern 3: Batch Processing

```typescript
// Generate multiple content pieces
const keywords = [
  'AI Marketing',
  'Content Automation',
  'Video Generation'
];

const results = await Promise.all(
  keywords.map(keyword =>
    runContentPipeline({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
    })
  )
);
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Add rate limiting and retries
import pRetry from 'p-retry';

async function generateContentWithRetry(options: GenerateContentOptions) {
  return pRetry(
    () => generateContent(options),
    {
      retries: 3,
      onFailedAttempt: (error) => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      },
    }
  );
}
```

### Issue: Video Rendering Memory Issues

```typescript
// Render video in smaller chunks or reduce quality
await renderMedia({
  // ... other options
  scale: 0.75, // Reduce to 75% quality
  concurrency: 1, // Render one frame at a time
});
```

### Issue: Research Data Quality

```typescript
// Filter and validate research results
function filterQualityResearch(results: ResearchResult[]): ResearchResult[] {
  return results.filter(r => 
    r.content.length > 100 && // Minimum content length
    r.insights.length > 0 && // Must have insights
    isRecent(r.publishedAt, 48) // Within 48 hours
  );
}

function isRecent(date: Date, hours: number): boolean {
  const now = new Date();
  const diff = now.getTime() - date.getTime();
  return diff <= hours * 60 * 60 * 1000;
}
```

## Advanced Configuration

### Custom AI Prompts

```typescript
// src/lib/generator/prompts.ts
export const CUSTOM_PROMPTS = {
  viral: `Create viral-worthy content that is highly shareable...`,
  seo: `Optimize for SEO with keywords, meta descriptions...`,
  educational: `Create comprehensive educational content...`,
};

// Use custom prompts
const content = await generateContent({
  ...options,
  customPrompt: CUSTOM_PROMPTS.viral,
});
```

### Multi-language Support

```typescript
// Generate content in both languages simultaneously
const [englishContent, vietnameseContent] = await Promise.all([
  generateContent({ ...options, language: 'en' }),
  generateContent({ ...options, language: 'vi' }),
]);
```
