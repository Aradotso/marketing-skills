---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "automate content creation with AI research and video generation"
  - "build automated marketing content pipeline"
  - "generate videos from written content automatically"
  - "crawl news and create content with Claude"
  - "set up AI content automation system"
  - "create content pipeline with Remotion video rendering"
  - "automate research to video content workflow"
  - "build marketing automation with AI and video"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete automated content creation pipeline that researches trending topics from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates multi-format written content using Claude 3 or OpenAI, and automatically renders videos using Remotion. This TypeScript/Next.js system reduces content production time by up to 90%.

## What This Does

- **Auto-Research**: Crawls real-time data from news sources and social media (last 24h)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) in Vietnamese and English
- **Video Rendering**: Automatically converts written content into videos and infographics using Remotion
- **Multi-Platform Export**: Outputs optimized video formats for Reels, TikTok, and Shorts

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

```env
# AI Provider Keys (choose one or both)
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media API Keys (optional, for crawling)
TWITTER_BEARER_TOKEN=your_twitter_token_here
LINKEDIN_ACCESS_TOKEN=your_linkedin_token_here

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here

# Database (if using)
DATABASE_URL=your_database_url_here

# Application
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

# Render videos (Remotion)
npm run remotion:render
```

## Core Architecture

### 1. Research Module

The research module crawls and analyzes content from multiple sources:

```typescript
// src/lib/research/crawler.ts
import { crawlTechCrunch, crawlTwitter, crawlLinkedIn } from './sources';

interface ResearchResult {
  title: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
  dataPoints: Array<{ metric: string; value: string }>;
}

export async function conductResearch(keyword: string): Promise<ResearchResult[]> {
  const sources = await Promise.all([
    crawlTechCrunch(keyword),
    crawlTwitter(keyword),
    crawlLinkedIn(keyword),
  ]);

  return sources
    .flat()
    .filter((item) => isRecent(item.publishedAt, 24)) // Last 24 hours
    .sort((a, b) => b.publishedAt.getTime() - a.publishedAt.getTime());
}

function isRecent(date: Date, hours: number): boolean {
  const now = new Date();
  const diff = now.getTime() - date.getTime();
  return diff <= hours * 60 * 60 * 1000;
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with customizable formats:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  research: ResearchResult[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
  keyword: string;
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const { research, format, language, tone, keyword } = request;
  
  const prompt = buildPrompt(research, format, language, tone, keyword);
  
  // Using Claude (preferred)
  if (process.env.ANTHROPIC_API_KEY) {
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

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }
  
  // Fallback to OpenAI
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing content.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(
  research: ResearchResult[],
  format: ContentFormat,
  language: Language,
  tone: Tone,
  keyword: string
): string {
  const researchContext = research
    .map((r) => `- ${r.title}: ${r.content.substring(0, 200)}...`)
    .join('\n');

  const formatInstructions = {
    toplist: 'Create a numbered list article with clear rankings and explanations',
    pov: 'Write from a unique perspective with personal insights and opinions',
    'case-study': 'Present a detailed case study with problem, solution, and results',
    'how-to': 'Create a step-by-step tutorial with actionable instructions',
  };

  return `
Create a ${format} article about "${keyword}" in ${language === 'vi' ? 'Vietnamese' : 'English'}.
Tone: ${tone}

Research data from the last 24 hours:
${researchContext}

Instructions:
- ${formatInstructions[format]}
- Include specific data points and insights from the research
- Make it engaging and actionable
- Optimize for social media sharing
- Length: 800-1200 words
`;
}
```

### 3. Dual Language Support

Generate content in both languages simultaneously:

```typescript
// src/lib/ai/multi-language.ts
export async function generateBilingualContent(
  request: Omit<ContentRequest, 'language'>
): Promise<{ en: string; vi: string }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ ...request, language: 'en' }),
    generateContent({ ...request, language: 'vi' }),
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}
```

### 4. Video Generation with Remotion

Convert content into video format:

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  sections: Array<{ heading: string; text: string }>;
  dataPoints: Array<{ metric: string; value: string }>;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  sections,
  dataPoints,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title sequence (0-3 seconds) */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60,
          }}
        >
          <h1
            style={{
              fontSize: 72,
              color: 'white',
              textAlign: 'center',
              fontWeight: 'bold',
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Content sections (3 seconds each) */}
      {sections.map((section, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 3)}
          durationInFrames={fps * 3}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
            }}
          >
            <h2 style={{ fontSize: 48, color: '#4CAF50', marginBottom: 20 }}>
              {section.heading}
            </h2>
            <p style={{ fontSize: 32, color: 'white', textAlign: 'center' }}>
              {section.text}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}

      {/* Data points visualization */}
      <Sequence
        from={fps * (3 + sections.length * 3)}
        durationInFrames={fps * 4}
      >
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60,
          }}
        >
          <div style={{ display: 'flex', flexDirection: 'column', gap: 20 }}>
            {dataPoints.map((dp, i) => (
              <div key={i} style={{ display: 'flex', alignItems: 'center' }}>
                <span style={{ fontSize: 40, color: '#4CAF50', marginRight: 20 }}>
                  {dp.value}
                </span>
                <span style={{ fontSize: 32, color: 'white' }}>{dp.metric}</span>
              </div>
            ))}
          </div>
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

Render the video:

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  props: ContentVideoProps,
  outputPath: string
): Promise<string> {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: props,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: props,
  });

  return outputPath;
}
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/pipeline/content-pipeline.ts
interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: Tone;
  generateVideo: boolean;
  platforms: Array<'reels' | 'tiktok' | 'shorts'>;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🔍 Starting research for: ${config.keyword}`);
  
  // Step 1: Research
  const research = await conductResearch(config.keyword);
  console.log(`✅ Found ${research.length} recent articles`);

  // Step 2: Generate content in both languages
  console.log('🤖 Generating content...');
  const content = await generateBilingualContent({
    research,
    format: config.format,
    tone: config.tone,
    keyword: config.keyword,
  });
  console.log('✅ Content generated in EN and VI');

  // Step 3: Parse content for video
  const parsedContent = parseContentForVideo(content.en);

  // Step 4: Generate video if requested
  let videoUrls: string[] = [];
  if (config.generateVideo) {
    console.log('🎬 Rendering videos...');
    
    for (const platform of config.platforms) {
      const aspectRatio = getAspectRatio(platform);
      const outputPath = `./output/${config.keyword}-${platform}.mp4`;
      
      await renderContentVideo(
        {
          ...parsedContent,
          aspectRatio,
        },
        outputPath
      );
      
      videoUrls.push(outputPath);
    }
    
    console.log(`✅ Videos rendered: ${videoUrls.join(', ')}`);
  }

  return {
    content,
    research,
    videos: videoUrls,
  };
}

function parseContentForVideo(content: string) {
  const lines = content.split('\n').filter((line) => line.trim());
  const title = lines[0].replace(/^#\s+/, '');
  
  const sections: Array<{ heading: string; text: string }> = [];
  const dataPoints: Array<{ metric: string; value: string }> = [];

  // Simple parsing logic (enhance as needed)
  let currentSection = { heading: '', text: '' };
  
  for (let i = 1; i < lines.length; i++) {
    const line = lines[i];
    
    if (line.startsWith('##')) {
      if (currentSection.heading) {
        sections.push(currentSection);
      }
      currentSection = { heading: line.replace(/^##\s+/, ''), text: '' };
    } else {
      currentSection.text += line + ' ';
    }
  }
  
  if (currentSection.heading) {
    sections.push(currentSection);
  }

  return { title, sections, dataPoints };
}

function getAspectRatio(platform: 'reels' | 'tiktok' | 'shorts') {
  const ratios = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  return ratios[platform];
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
      tone: body.tone || 'friendly',
      generateVideo: body.generateVideo ?? true,
      platforms: body.platforms || ['reels', 'tiktok', 'shorts'],
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Usage Example

```typescript
// Example client-side usage
async function createContent() {
  const response = await fetch('/api/pipeline', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword: 'AI automation trends 2024',
      format: 'toplist',
      tone: 'expert',
      generateVideo: true,
      platforms: ['reels', 'tiktok'],
    }),
  });

  const result = await response.json();
  console.log('English content:', result.content.en);
  console.log('Vietnamese content:', result.content.vi);
  console.log('Videos:', result.videos);
}
```

## Common Patterns

### Scheduled Content Generation

```typescript
// src/lib/scheduler/cron.ts
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    for (const keyword of keywords) {
      await runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'friendly',
        generateVideo: true,
        platforms: ['reels', 'tiktok', 'shorts'],
      });
    }
  });
}
```

### Batch Processing

```typescript
export async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map((keyword) =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'expert',
        generateVideo: false,
        platforms: [],
      })
    )
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for API calls
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent API calls

export async function conductResearchWithRateLimit(keywords: string[]) {
  return Promise.all(
    keywords.map((keyword) => limit(() => conductResearch(keyword)))
  );
}
```

### Video Rendering Memory Issues

If you encounter memory issues during video rendering:

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

### Claude/OpenAI Timeout Issues

```typescript
// Add timeout and retry logic
async function generateContentWithRetry(
  request: ContentRequest,
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise((resolve) => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearch(research: ResearchResult[]): ResearchResult[] {
  return research.filter((item) => {
    return (
      item.content.length > 100 && // Minimum content length
      item.insights.length > 0 && // Must have insights
      item.source !== 'unknown' // Valid source
    );
  });
}
```

## Performance Optimization

Enable caching for research results:

```typescript
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour cache

export async function conductResearchCached(keyword: string) {
  const cached = cache.get<ResearchResult[]>(keyword);
  if (cached) return cached;

  const results = await conductResearch(keyword);
  cache.set(keyword, results);
  return results;
}
```

This pipeline automates the entire content creation workflow from trend research to video publishing, making it ideal for content creators, marketers, and businesses looking to scale their content production.
