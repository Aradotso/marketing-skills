---
name: marketing-pipeline-automation
description: Automate content creation from research to video generation using AI and Remotion for marketing workflows
triggers:
  - how do i automate content creation with AI
  - set up marketing pipeline for automated posts
  - generate videos from content automatically
  - crawl news and create social media content
  - use remotion to render marketing videos
  - automate research and scriptwriting workflow
  - build AI content pipeline with Claude
  - create automated content from keywords
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline**, a TypeScript-based system that automates the entire content creation workflow: from research and scriptwriting to video generation and publishing. The pipeline integrates Claude/OpenAI for content generation, web scraping for real-time research, and Remotion for video rendering.

## What This Project Does

The Marketing Pipeline automates:

1. **Auto-Scan Research**: Crawls latest news from TechCrunch, a16z, Twitter, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Output**: Generates Vietnamese and English versions simultaneously
4. **Video Rendering**: Converts text content into videos/infographics using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

# Set up environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# .env file
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run render
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Core utilities
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── scraper/           # Web scraping modules
│   └── video/             # Remotion video generation
├── remotion/              # Remotion video templates
├── public/                # Static assets
└── types/                 # TypeScript definitions
```

## API Usage Patterns

### 1. Content Research API

```typescript
import { researchTopic } from '@/lib/scraper/research';

async function gatherResearch(keyword: string) {
  const sources = {
    techcrunch: true,
    a16z: true,
    twitter: true,
    linkedin: true
  };
  
  const researchData = await researchTopic(keyword, {
    sources,
    timeRange: '24h',
    maxResults: 20
  });
  
  return {
    insights: researchData.insights,
    dataPoints: researchData.statistics,
    trendingTopics: researchData.trends
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language } from '@/types';

async function createContent(topic: string, research: any) {
  const contentConfig = {
    format: 'toplist' as ContentFormat, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'] as Language[],
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    includeDataPoints: true,
    targetPlatform: 'linkedin'
  };
  
  const content = await generateContent({
    topic,
    research,
    config: contentConfig,
    aiProvider: 'claude' // or 'openai'
  });
  
  return content;
}
```

### 3. Using Claude API

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, researchData: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Based on this research data: ${JSON.stringify(researchData)}
        
Create a professional marketing article about: ${prompt}

Format: Toplist
Language: English and Vietnamese
Include: Statistics, trends, and actionable insights`
      }
    ],
  });
  
  return message.content;
}
```

### 4. Using OpenAI API

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content creator specializing in data-driven articles.'
      },
      {
        role: 'user',
        content: `Context: ${JSON.stringify(context)}
        
Task: ${prompt}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}
```

## Video Generation with Remotion

### 1. Basic Video Composition

```typescript
// remotion/compositions/MarketingVideo.tsx
import { Composition } from 'remotion';
import { VideoTemplate } from './templates/VideoTemplate';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="MarketingVideo"
        component={VideoTemplate}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920} // Vertical for Reels/TikTok
        defaultProps={{
          title: 'Your Title',
          content: [],
          branding: {
            logo: '/logo.png',
            color: '#000000'
          }
        }}
      />
    </>
  );
};
```

### 2. Video Template Component

```typescript
// remotion/templates/VideoTemplate.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface VideoProps {
  title: string;
  content: Array<{ text: string; highlight?: boolean }>;
  branding: { logo: string; color: string };
}

export const VideoTemplate: React.FC<VideoProps> = ({ title, content, branding }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#fff' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ fontSize: 64, color: branding.color, marginBottom: 40 }}>
          {title}
        </h1>
        {content.map((item, index) => (
          <p key={index} style={{ fontSize: 32, marginBottom: 20 }}>
            {item.text}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

### 3. Rendering Videos Programmatically

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: any) {
  const compositionId = 'MarketingVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      content: content.sections,
      branding: content.branding
    }
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${content.slug}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps,
  });
  
  return outputLocation;
}
```

## Complete Workflow Example

```typescript
// lib/workflows/content-pipeline.ts
import { researchTopic } from '@/lib/scraper/research';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { publishToSocial } from '@/lib/social/publisher';

export async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await researchTopic(keyword, {
      sources: { techcrunch: true, twitter: true },
      timeRange: '24h'
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      topic: keyword,
      research,
      config: {
        format: 'toplist',
        languages: ['en', 'vi'],
        tone: 'professional'
      },
      aiProvider: 'claude'
    });
    
    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: content.title,
      sections: content.sections,
      branding: { logo: '/logo.png', color: '#000' }
    });
    
    // Step 4: Publish (optional)
    console.log('📤 Publishing...');
    await publishToSocial({
      platforms: ['linkedin', 'twitter'],
      content: content.text,
      media: videoPath
    });
    
    return {
      success: true,
      content,
      video: videoPath
    };
    
  } catch (error) {
    console.error('Pipeline failed:', error);
    throw error;
  }
}
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/workflows/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
    
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/scraper/research';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');
  const sources = searchParams.get('sources')?.split(',') || [];
  
  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword parameter required' },
      { status: 400 }
    );
  }
  
  const research = await researchTopic(keyword, {
    sources: sources.reduce((acc, s) => ({ ...acc, [s]: true }), {}),
    timeRange: '24h'
  });
  
  return NextResponse.json(research);
}
```

## Configuration

### TypeScript Types

```typescript
// types/index.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'professional' | 'friendly' | 'humorous';
export type AIProvider = 'claude' | 'openai';

export interface ResearchConfig {
  sources: {
    techcrunch?: boolean;
    a16z?: boolean;
    twitter?: boolean;
    linkedin?: boolean;
  };
  timeRange: '24h' | '7d' | '30d';
  maxResults?: number;
}

export interface ContentConfig {
  format: ContentFormat;
  languages: Language[];
  tone: Tone;
  includeDataPoints?: boolean;
  targetPlatform?: string;
}

export interface GeneratedContent {
  title: string;
  slug: string;
  text: string;
  sections: Array<{ heading: string; content: string }>;
  metadata: {
    format: ContentFormat;
    language: Language;
    wordCount: number;
  };
}
```

## Common Patterns

### Pattern 1: Multi-language Content Generation

```typescript
async function generateMultilingualContent(topic: string) {
  const languages: Language[] = ['en', 'vi'];
  const contentByLanguage: Record<string, any> = {};
  
  for (const lang of languages) {
    contentByLanguage[lang] = await generateContent({
      topic,
      research: await researchTopic(topic),
      config: {
        format: 'toplist',
        languages: [lang],
        tone: 'professional'
      },
      aiProvider: 'claude'
    });
  }
  
  return contentByLanguage;
}
```

### Pattern 2: Batch Video Rendering

```typescript
async function batchRenderVideos(contents: any[]) {
  const renderPromises = contents.map(content => 
    renderContentVideo({
      title: content.title,
      sections: content.sections,
      branding: content.branding
    })
  );
  
  const videos = await Promise.all(renderPromises);
  return videos;
}
```

### Pattern 3: Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('Starting scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        await runContentPipeline(keyword);
        console.log(`✅ Completed: ${keyword}`);
      } catch (error) {
        console.error(`❌ Failed: ${keyword}`, error);
      }
    }
  });
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent API calls

async function generateWithRateLimit(topics: string[]) {
  const results = await Promise.all(
    topics.map(topic => 
      limit(() => runContentPipeline(topic))
    )
  );
  return results;
}
```

### Issue: Video Rendering Memory

```typescript
// Optimize Remotion rendering for memory
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  // Reduce concurrency for memory-constrained environments
  concurrency: 2,
  // Enable hardware acceleration
  chromiumOptions: {
    gl: 'angle',
  },
});
```

### Issue: Long Content Processing

```typescript
// Use streaming for real-time updates
import { OpenAI } from 'openai';

async function streamContentGeneration(prompt: string) {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: prompt }],
    stream: true,
  });
  
  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content || '';
    process.stdout.write(content);
  }
}
```

### Issue: Missing Environment Variables

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
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

## Best Practices

1. **Cache Research Data**: Store research results to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for video rendering tasks
3. **Error Handling**: Implement retry logic for API failures
4. **Monitoring**: Log all pipeline stages for debugging
5. **Content Validation**: Validate generated content before publishing

This skill provides comprehensive coverage for working with the Marketing Pipeline automation system, enabling AI agents to effectively assist developers in implementing automated content workflows.
