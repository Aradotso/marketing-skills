---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline that researches, generates scripts, and creates videos automatically using Claude/OpenAI and Remotion
triggers:
  - automate my content pipeline with AI research and video generation
  - generate marketing content from research to video automatically
  - set up AI content automation with Claude and OpenAI
  - create automated content workflow with video rendering
  - build content pipeline that researches and generates videos
  - use marketing pipeline automation for content creation
  - configure AI-powered content generation and video export
  - automate content research writing and video production
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive AI-powered content automation system that handles the entire content creation workflow: from research and data crawling, to script generation, and finally video rendering. Built with TypeScript, Next.js, and integrates Claude 3, OpenAI, and Remotion for end-to-end content automation.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- Generate content in multiple formats (Toplist, POV, Case Study, How-to)
- Bilingual support (English & Vietnamese) with customizable tone
- Automatic video and infographic rendering via Remotion
- Export optimized for Reels, TikTok, Shorts

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
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research & Crawling
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
REMOTION_S3_BUCKET=your_bucket_name

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
DATABASE_URL=your_database_url_here
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Render video compositions (Remotion)
npm run remotion:render
```

## Core Architecture

### 1. Research Module (Auto-Scan)

Automatically crawls and analyzes fresh data from multiple sources:

```typescript
// lib/research/crawler.ts
import { RapidAPI } from '@/lib/api/rapid';

interface ResearchConfig {
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  keyword: string;
  timeframe: '24h' | '7d' | '30d';
  language: 'en' | 'vi';
}

export async function performResearch(config: ResearchConfig) {
  const rapidAPI = new RapidAPI(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    config.sources.map(source => 
      rapidAPI.fetchNews({
        source,
        keyword: config.keyword,
        timeframe: config.timeframe
      })
    )
  );
  
  // Aggregate and analyze data
  const insights = await analyzeResearchData(results.flat());
  
  return {
    rawData: results,
    insights,
    timestamp: new Date().toISOString()
  };
}

async function analyzeResearchData(data: any[]) {
  // Extract key insights, trends, statistics
  return {
    topTrends: extractTrends(data),
    keyStatistics: extractStats(data),
    relevantCaseStudies: extractCases(data)
  };
}
```

### 2. Content Generation (AI Module)

Generate content using Claude or OpenAI with various formats:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type VoiceTone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentParams {
  keyword: string;
  format: ContentFormat;
  tone: VoiceTone;
  language: 'en' | 'vi';
  researchData: any;
  provider: 'claude' | 'openai';
}

export async function generateContent(params: GenerateContentParams) {
  const prompt = buildPrompt(params);
  
  if (params.provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string) {
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
  
  return parseContentResponse(message.content[0].text);
}

async function generateWithOpenAI(prompt: string) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: 'You are an expert content creator and marketer.'
    }, {
      role: 'user',
      content: prompt
    }],
    temperature: 0.7
  });
  
  return parseContentResponse(completion.choices[0].message.content!);
}

function buildPrompt(params: GenerateContentParams): string {
  const formatInstructions = {
    'toplist': 'Create a top 10 list format',
    'pov': 'Write from a unique point of view',
    'case-study': 'Analyze as a detailed case study',
    'how-to': 'Structure as a step-by-step guide'
  };
  
  return `
    Create ${params.language === 'en' ? 'English' : 'Vietnamese'} content about: ${params.keyword}
    
    Format: ${formatInstructions[params.format]}
    Tone: ${params.tone}
    
    Use this research data:
    ${JSON.stringify(params.researchData.insights, null, 2)}
    
    Requirements:
    - Include data-backed insights
    - Add relevant statistics
    - Structure for social media engagement
    - Include actionable takeaways
  `;
}

interface ParsedContent {
  title: string;
  sections: { heading: string; content: string }[];
  keyPoints: string[];
  cta: string;
}

function parseContentResponse(text: string): ParsedContent {
  // Parse AI response into structured format
  const lines = text.split('\n');
  
  return {
    title: lines[0].replace(/^#\s*/, ''),
    sections: extractSections(lines),
    keyPoints: extractKeyPoints(lines),
    cta: extractCTA(lines)
  };
}
```

### 3. Video Generation (Remotion Integration)

Automatically render videos from generated content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/Composition';

interface VideoRenderParams {
  content: ParsedContent;
  platform: 'reels' | 'tiktok' | 'shorts';
  style: 'minimal' | 'dynamic' | 'professional';
}

export async function renderContentVideo(params: VideoRenderParams) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const { width, height } = dimensions[params.platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: params.content,
      style: params.style
    }
  });
  
  // Render video
  const outputPath = `./output/${Date.now()}-${params.platform}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: params.content,
      style: params.style
    }
  });
  
  return {
    path: outputPath,
    width,
    height,
    platform: params.platform
  };
}
```

```tsx
// remotion/Composition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import { ParsedContent } from '@/lib/ai/content-generator';

interface CompositionProps {
  content: ParsedContent;
  style: 'minimal' | 'dynamic' | 'professional';
}

export const VideoComposition: React.FC<CompositionProps> = ({ 
  content, 
  style 
}) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={90}>
        <TitleScene title={content.title} style={style} />
      </Sequence>
      
      {content.sections.map((section, index) => (
        <Sequence 
          key={index}
          from={90 + (index * 120)} 
          durationInFrames={120}
        >
          <ContentScene section={section} style={style} />
        </Sequence>
      ))}
      
      <Sequence from={90 + (content.sections.length * 120)}>
        <CTAScene cta={content.cta} style={style} />
      </Sequence>
    </AbsoluteFill>
  );
};
```

## API Routes

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { performResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, platform } = body;
    
    // Step 1: Research
    const researchData = await performResearch({
      sources: ['techcrunch', 'a16z', 'twitter'],
      keyword,
      timeframe: '24h',
      language
    });
    
    // Step 2: Generate content
    const content = await generateContent({
      keyword,
      format,
      tone: 'expert',
      language,
      researchData,
      provider: 'claude'
    });
    
    // Step 3: Render video
    const video = await renderContentVideo({
      content,
      platform,
      style: 'dynamic'
    });
    
    return NextResponse.json({
      success: true,
      content,
      video
    });
    
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Full Pipeline Execution

```typescript
// Example: Complete content automation workflow
import { performResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

async function runFullPipeline(keyword: string) {
  // 1. Auto-research
  console.log('🔍 Researching...');
  const research = await performResearch({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    keyword,
    timeframe: '24h',
    language: 'en'
  });
  
  // 2. Generate content (English)
  console.log('✍️ Generating English content...');
  const contentEN = await generateContent({
    keyword,
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    researchData: research,
    provider: 'claude'
  });
  
  // 3. Generate content (Vietnamese)
  console.log('✍️ Generating Vietnamese content...');
  const contentVI = await generateContent({
    keyword,
    format: 'toplist',
    tone: 'friendly',
    language: 'vi',
    researchData: research,
    provider: 'openai'
  });
  
  // 4. Render videos for multiple platforms
  console.log('🎬 Rendering videos...');
  const videos = await Promise.all([
    renderContentVideo({
      content: contentEN,
      platform: 'reels',
      style: 'dynamic'
    }),
    renderContentVideo({
      content: contentVI,
      platform: 'tiktok',
      style: 'professional'
    })
  ]);
  
  return {
    research,
    content: { en: contentEN, vi: contentVI },
    videos
  };
}
```

### Batch Content Generation

```typescript
// Generate multiple content pieces from different keywords
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runFullPipeline(keyword);
      results.push({ keyword, ...result });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
    } catch (error) {
      console.error(`Failed for ${keyword}:`, error);
    }
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Issues

- Ensure sufficient disk space for output videos
- Check Remotion AWS credentials for cloud rendering
- Verify FFmpeg installation for local rendering
- Use smaller compositions if running out of memory

### Research Data Quality

- Validate API keys are active and have sufficient quota
- Check source availability (some may be rate-limited)
- Use fallback providers if primary source fails
- Cache research results to avoid redundant API calls

## Configuration Tips

- Use Claude for more creative, narrative content
- Use OpenAI for structured, data-heavy content
- Adjust `temperature` parameter for creativity vs consistency
- Set longer `timeframe` for more comprehensive research
- Test different `style` options for video rendering performance
