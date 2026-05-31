---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I auto-generate content with AI research
  - set up automated content pipeline with video generation
  - use Claude and OpenAI for content automation
  - create AI-powered marketing content workflow
  - generate videos from blog posts automatically
  - build content research and writing pipeline
  - automate content creation from keyword to video
  - integrate Remotion for AI content videos
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating multi-format content (blog posts, scripts), to rendering videos with Remotion. The pipeline integrates Claude 3, OpenAI, RapidAPI for news crawling, and supports bilingual output (English/Vietnamese).

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-scans research sources**: Crawls real-time data from TechCrunch, a16z, Twitter, LinkedIn within the last 24 hours
- **Generates diverse content formats**: Creates Toplists, POV articles, Case Studies, How-to guides using Claude or OpenAI
- **Supports bilingual output**: Generates content in both English and Vietnamese with customizable tone (expert, friendly, humorous)
- **Renders videos automatically**: Converts written content into infographics and short-form videos using Remotion
- **Optimizes for multi-platform**: Exports videos in ratios suitable for Reels, TikTok, Shorts

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
# Install all dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Model APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Key Architecture & Components

### 1. Research Module (Content Crawling)

```typescript
// lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

export interface ResearchSource {
  url: string;
  title: string;
  content: string;
  publishedAt: Date;
  source: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
}

export async function crawlRecentNews(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<ResearchSource[]> {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY);
  
  const sources = await Promise.all([
    rapidAPI.searchTechCrunch(keyword, timeframe),
    rapidAPI.searchTwitter(keyword, timeframe),
    rapidAPI.searchLinkedIn(keyword, timeframe),
  ]);
  
  return sources.flat().sort(
    (a, b) => b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}

// Usage example
const research = await crawlRecentNews('AI marketing automation', '24h');
console.log(`Found ${research.length} recent articles`);
```

### 2. Content Generation with Claude/OpenAI

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ContentTone = 'expert' | 'friendly' | 'humorous';
export type Language = 'en' | 'vi';

interface GenerateContentParams {
  keyword: string;
  research: ResearchSource[];
  format: ContentFormat;
  tone: ContentTone;
  language: Language;
  provider: 'claude' | 'openai';
}

export async function generateContent(params: GenerateContentParams) {
  const { keyword, research, format, tone, language, provider } = params;
  
  const prompt = buildPrompt(keyword, research, format, tone, language);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
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
        {
          role: 'system',
          content: 'You are an expert content writer specializing in marketing.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      temperature: 0.7,
    });
    
    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(
  keyword: string,
  research: ResearchSource[],
  format: ContentFormat,
  tone: ContentTone,
  language: Language
): string {
  const researchContext = research
    .map((r) => `- ${r.title} (${r.source}): ${r.content.slice(0, 200)}...`)
    .join('\n');
  
  const formatInstructions = {
    toplist: 'Create a numbered list article with rankings and explanations',
    pov: 'Write from a specific perspective with strong opinions',
    'case-study': 'Analyze real examples with data and outcomes',
    'how-to': 'Provide step-by-step actionable instructions',
  };
  
  const toneGuidelines = {
    expert: 'authoritative, data-driven, professional',
    friendly: 'conversational, accessible, warm',
    humorous: 'witty, entertaining, light-hearted',
  };
  
  return `
Write a comprehensive ${format} article about "${keyword}" in ${language === 'en' ? 'English' : 'Vietnamese'}.

Tone: ${toneGuidelines[tone]}
Format: ${formatInstructions[format]}

Recent Research Data:
${researchContext}

Requirements:
- Use data from the research sources provided
- Include specific examples and statistics
- Make it actionable and valuable
- ${language === 'vi' ? 'Write in Vietnamese with natural, engaging language' : 'Write in clear, engaging English'}
- Length: 1500-2000 words
`;
}
```

### 3. Bilingual Content Generation

```typescript
// lib/ai/bilingual-generator.ts
export async function generateBilingualContent(
  keyword: string,
  research: ResearchSource[],
  format: ContentFormat,
  tone: ContentTone
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      research,
      format,
      tone,
      language: 'en',
      provider: 'claude',
    }),
    generateContent({
      keyword,
      research,
      format,
      tone,
      language: 'vi',
      provider: 'claude',
    }),
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}

// Usage
const content = await generateBilingualContent(
  'AI content automation',
  researchData,
  'how-to',
  'expert'
);
```

### 4. Video Generation with Remotion

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number; // in seconds
}

export async function renderContentVideo(config: VideoConfig) {
  const { title, content, format, duration } = config;
  
  // Aspect ratios for different platforms
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  
  const { width, height } = dimensions[format];
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content: parseContentForVideo(content),
      duration,
    },
  });
  
  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${format}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      content: parseContentForVideo(content),
      duration,
    },
  });
  
  return outputLocation;
}

function parseContentForVideo(content: string): string[] {
  // Extract key points from content for video scenes
  const sentences = content.split(/[.!?]+/).filter((s) => s.trim().length > 0);
  return sentences.slice(0, 5).map((s) => s.trim()); // First 5 key points
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  duration,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  const currentScene = Math.floor(frame / (fps * 3)); // 3 seconds per scene
  const sceneContent = content[currentScene] || content[content.length - 1];
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      {frame < fps * 2 && (
        <h1
          style={{
            color: '#fff',
            fontSize: 72,
            fontWeight: 'bold',
            textAlign: 'center',
            opacity: titleOpacity,
          }}
        >
          {title}
        </h1>
      )}
      
      {frame >= fps * 2 && (
        <div
          style={{
            color: '#fff',
            fontSize: 48,
            textAlign: 'center',
            lineHeight: 1.6,
          }}
        >
          {sceneContent}
        </div>
      )}
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Workflow

```typescript
// lib/pipeline/content-pipeline.ts
export interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: ContentTone;
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(config: PipelineConfig) {
  const { keyword, format, tone, generateVideo, videoFormat } = config;
  
  console.log(`🚀 Starting pipeline for: ${keyword}`);
  
  // Step 1: Research
  console.log('📡 Crawling recent news...');
  const research = await crawlRecentNews(keyword, '24h');
  console.log(`✓ Found ${research.length} sources`);
  
  // Step 2: Generate bilingual content
  console.log('🧠 Generating content...');
  const content = await generateBilingualContent(keyword, research, format, tone);
  console.log('✓ Content generated');
  
  // Step 3: Generate video (if requested)
  let videoPath: string | null = null;
  if (generateVideo && videoFormat) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo({
      title: keyword,
      content: content.en,
      format: videoFormat,
      duration: 30,
    });
    console.log(`✓ Video saved: ${videoPath}`);
  }
  
  return {
    content,
    research,
    videoPath,
    metadata: {
      keyword,
      format,
      tone,
      generatedAt: new Date(),
      sourcesCount: research.length,
    },
  };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation 2024',
  format: 'toplist',
  tone: 'expert',
  generateVideo: true,
  videoFormat: 'reels',
});

console.log('Pipeline complete!', result.metadata);
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, generateVideo, videoFormat } = body;
    
    if (!keyword || !format || !tone) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      format,
      tone,
      generateVideo: generateVideo || false,
      videoFormat: videoFormat || 'reels',
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Quick Content Generation (No Video)

```typescript
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

const quickContent = await runContentPipeline({
  keyword: 'Social Media Marketing Trends',
  format: 'toplist',
  tone: 'friendly',
  generateVideo: false,
});
```

### Full Pipeline with Video

```typescript
const fullPipeline = await runContentPipeline({
  keyword: 'AI Content Creation Tools',
  format: 'how-to',
  tone: 'expert',
  generateVideo: true,
  videoFormat: 'tiktok',
});
```

### Custom Research Timeframe

```typescript
const weeklyResearch = await crawlRecentNews('marketing automation', '7d');
const content = await generateContent({
  keyword: 'marketing automation',
  research: weeklyResearch,
  format: 'case-study',
  tone: 'expert',
  language: 'en',
  provider: 'openai',
});
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  retries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < retries - 1) {
        await new Promise((resolve) => setTimeout(resolve, delay * (i + 1)));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRateLimit(() =>
  generateContent({
    keyword: 'test',
    research: [],
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    provider: 'claude',
  })
);
```

### Issue: Missing Environment Variables

```typescript
// lib/config/validate-env.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter((key) => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

// Call at app startup
validateEnv();
```

### Issue: Video Rendering Failures

```typescript
// Check Remotion configuration
import { getCompositions } from '@remotion/renderer';

export async function validateRemotionSetup() {
  try {
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
    });
    
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions.map((c) => c.id));
    return true;
  } catch (error) {
    console.error('Remotion setup error:', error);
    return false;
  }
}
```

## Performance Optimization

```typescript
// lib/cache/content-cache.ts
import { LRUCache } from 'lru-cache';

const contentCache = new LRUCache<string, any>({
  max: 100,
  ttl: 1000 * 60 * 60, // 1 hour
});

export async function getCachedContent(
  keyword: string,
  generator: () => Promise<any>
) {
  const cached = contentCache.get(keyword);
  if (cached) return cached;
  
  const result = await generator();
  contentCache.set(keyword, result);
  return result;
}
```

This skill provides comprehensive guidance for working with the Ultimate AI Content Pipeline, covering research automation, AI content generation, video rendering, and full pipeline orchestration.
