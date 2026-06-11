---
name: marketing-pipeline-ai-content-automation
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from blog posts automatically
  - crawl news and create content pipelines
  - set up AI marketing content automation
  - build automated content workflow with Claude
  - create video content from AI-generated scripts
  - research and publish content automatically
  - generate multilingual marketing content with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to automatic video generation and publishing.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

1. **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Uses Claude 3 and OpenAI to generate content in multiple formats (Toplist, POV, Case Study, How-to)
3. **Multilingual Output**: Creates parallel Vietnamese and English content with customizable tone
4. **Video Rendering**: Automatically converts text content into videos and infographics using Remotion
5. **Multi-Platform Export**: Outputs videos optimized for Reels, TikTok, and Shorts

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
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social Media APIs
FACEBOOK_PAGE_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video templates
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { researchTopic } from '@/lib/crawler/research';

interface ResearchResult {
  sources: Array<{
    title: string;
    url: string;
    summary: string;
    publishedAt: string;
  }>;
  insights: string[];
  keywords: string[];
}

async function gatherResearch(topic: string): Promise<ResearchResult> {
  const research = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });
  
  return research;
}

// Usage
const data = await gatherResearch('AI marketing automation');
console.log(data.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  researchData: ResearchResult;
}

async function generateContent(config: ContentConfig): Promise<string> {
  const systemPrompt = `You are a content marketing expert. Create ${config.format} content in ${config.language} with a ${config.tone} tone.`;
  
  const userPrompt = `
Topic: ${config.topic}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Create comprehensive ${config.format} content including:
- Engaging headline
- Introduction with hook
- Main content (3-5 sections)
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: userPrompt
    }],
    system: systemPrompt,
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Generate bilingual content
const enContent = await generateContent({
  topic: 'AI Marketing Trends 2026',
  format: 'toplist',
  language: 'en',
  tone: 'professional',
  researchData: data
});

const viContent = await generateContent({
  topic: 'Xu hướng Marketing AI 2026',
  format: 'toplist',
  language: 'vi',
  tone: 'friendly',
  researchData: data
});
```

### 3. Alternative: OpenAI Content Generation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(config: ContentConfig): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `Content marketing expert creating ${config.format} in ${config.language}`
      },
      {
        role: 'user',
        content: `Topic: ${config.topic}\n\nResearch: ${JSON.stringify(config.researchData)}\n\nCreate engaging content.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const PLATFORM_SPECS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
};

async function generateVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const spec = PLATFORM_SPECS[config.platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${config.platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  content: enContent,
  title: 'Top 5 AI Marketing Trends',
  platform: 'reels',
});
```

### 5. Remotion Video Component Example

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = interpolate(
    frame,
    [0, 30, durationInFrames - 30, durationInFrames],
    [0, 1, 1, 0]
  );

  const scale = interpolate(
    frame,
    [0, 30],
    [0.8, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          color: 'white',
          padding: 60,
          fontFamily: 'Arial, sans-serif',
        }}
      >
        <h1 style={{ fontSize: 72, marginBottom: 40, fontWeight: 'bold' }}>
          {title}
        </h1>
        <div style={{ fontSize: 32, lineHeight: 1.6 }}>
          {content.split('\n\n').map((paragraph, i) => (
            <p key={i} style={{ marginBottom: 30 }}>
              {paragraph}
            </p>
          ))}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/workflow.ts
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  platforms?: ('reels' | 'tiktok' | 'shorts')[];
}

interface PipelineResult {
  research: ResearchResult;
  content: {
    [lang: string]: string;
  };
  videos?: {
    [platform: string]: string;
  };
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  // Step 1: Research
  console.log(`🔍 Researching: ${config.keyword}`);
  const research = await researchTopic({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20,
  });

  // Step 2: Generate Content
  console.log(`✍️ Generating content in ${config.languages.length} languages`);
  const content: { [lang: string]: string } = {};
  
  for (const lang of config.languages) {
    content[lang] = await generateContent({
      topic: config.keyword,
      format: config.format,
      language: lang,
      tone: 'professional',
      researchData: research,
    });
  }

  // Step 3: Generate Videos (optional)
  let videos: { [platform: string]: string } = {};
  
  if (config.generateVideo && config.platforms) {
    console.log(`🎬 Generating videos for ${config.platforms.length} platforms`);
    
    for (const platform of config.platforms) {
      videos[platform] = await generateVideo({
        content: content['en'], // Use English content for video
        title: config.keyword,
        platform,
      });
    }
  }

  return {
    research,
    content,
    videos: config.generateVideo ? videos : undefined,
  };
}

// Usage Example
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation 2026',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  platforms: ['reels', 'tiktok'],
});

console.log('English Content:', result.content.en);
console.log('Vietnamese Content:', result.content.vi);
console.log('Reels Video:', result.videos?.reels);
```

## Next.js API Routes

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/workflow';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en', 'vi'],
      generateVideo: body.generateVideo ?? false,
      platforms: body.platforms || ['reels'],
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video preview
npm run remotion:preview

# Render specific video composition
npm run remotion:render
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// Use with cron job or scheduler
import { CronJob } from 'cron';

const dailyContentJob = new CronJob(
  '0 9 * * *', // Every day at 9 AM
  async () => {
    const topics = [
      'AI Marketing Trends',
      'Content Automation Tools',
      'Social Media Strategy',
    ];
    
    for (const topic of topics) {
      await runContentPipeline({
        keyword: topic,
        format: 'toplist',
        languages: ['en', 'vi'],
        generateVideo: true,
        platforms: ['reels'],
      });
    }
  },
  null,
  true,
  'Asia/Ho_Chi_Minh'
);

dailyContentJob.start();
```

### Pattern 2: Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'how-to',
        languages: ['en'],
        generateVideo: false,
      })
    )
  );
  
  return results;
}
```

### Pattern 3: Custom Research Sources

```typescript
// Add custom crawler
import axios from 'axios';

async function crawlCustomSource(url: string) {
  const response = await axios.get(url, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
    },
  });
  
  return response.data;
}
```

## Troubleshooting

### Issue: AI API Rate Limits

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(config: ContentConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(config);
    } catch (error) {
      if (error.status === 429) {
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

### Issue: Video Rendering Memory

```typescript
// Adjust Remotion concurrency
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  concurrency: 2, // Lower for limited memory
  enforceAudioTrack: false,
});
```

### Issue: Missing Environment Variables

```typescript
// Validation helper
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

### Issue: Crawler Being Blocked

```typescript
// Add user agent rotation
const USER_AGENTS = [
  'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36',
];

async function crawlWithRotation(url: string) {
  const randomUA = USER_AGENTS[Math.floor(Math.random() * USER_AGENTS.length)];
  
  return axios.get(url, {
    headers: { 'User-Agent': randomUA },
  });
}
```

## Best Practices

1. **API Key Management**: Always use environment variables, never hardcode keys
2. **Error Handling**: Implement comprehensive try-catch blocks for all external API calls
3. **Rate Limiting**: Implement request queuing to avoid hitting API limits
4. **Content Caching**: Store generated content to avoid regeneration costs
5. **Video Optimization**: Use appropriate codecs and resolutions for target platforms
6. **Monitoring**: Log all pipeline steps for debugging and analytics

This skill provides comprehensive coverage of the Marketing Pipeline automation system, enabling AI agents to effectively assist developers in building, configuring, and troubleshooting automated content workflows.
