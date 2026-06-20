---
name: marketing-pipeline-ai-content-automation
description: Automates end-to-end content creation from research to video generation using Claude/OpenAI and Remotion for marketing pipelines
triggers:
  - how do I set up the AI content pipeline
  - automate content research and video generation
  - use Claude to generate marketing content automatically
  - create automated content pipeline with AI
  - generate videos from content with Remotion
  - build automated marketing content workflow
  - set up AI-powered content creation system
  - automate content from research to publication
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research and scriptwriting to automatic posting and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline automates:
- **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
- **Multi-Format Content**: Generates content in various formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Multi-Language**: Produces content in both English and Vietnamese with customizable tone
- **Video Generation**: Automatically renders infographics and short videos using Remotion
- **Auto-Publishing**: Schedules and posts content to social media platforms

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

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media (optional)
FACEBOOK_ACCESS_TOKEN=your_fb_token_here
LINKEDIN_ACCESS_TOKEN=your_linkedin_token_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── research/         # Content research crawlers
│   ├── video/            # Remotion video generation
│   └── publishers/       # Social media publishing
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Core API Usage

### 1. Content Research

```typescript
import { researchTopic } from '@/lib/research/crawler';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });
  
  return {
    insights: research.insights,
    dataPoints: research.dataPoints,
    trends: research.trends,
    sources: research.sources
  };
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

async function generateContentWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `
Generate a ${format} article based on this research:
${JSON.stringify(research, null, 2)}

Language: ${language}
Tone: Professional yet engaging
Include: Data points, actionable insights, and trending information
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
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

// Usage
const content = await generateContentWithClaude(
  researchData,
  'toplist',
  'en'
);
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  topic: string,
  style: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${style} content creator specializing in marketing.`
      },
      {
        role: 'user',
        content: `Create a comprehensive article about: ${topic}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(contentData: {
  title: string;
  points: string[];
  images: string[];
}) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: contentData.title,
      points: contentData.points,
      images: contentData.images,
    },
  });

  // Render video
  const outputLocation = path.resolve(`./output/${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: contentData.title,
      points: contentData.points,
      images: contentData.images,
    },
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  title: '5 AI Marketing Trends in 2026',
  points: [
    'Hyper-personalization at scale',
    'AI-powered video generation',
    'Automated content pipelines',
    'Real-time sentiment analysis',
    'Predictive customer behavior'
  ],
  images: ['/img1.jpg', '/img2.jpg', '/img3.jpg']
});
```

### 5. Complete Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { publishToFacebook } from '@/lib/publishers/facebook';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContentWithClaude(
      research,
      'toplist',
      'en'
    );

    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(content);

    // Step 4: Generate video
    console.log('🎬 Creating video...');
    const videoPath = await generateVideo({
      title: keyword,
      points: keyPoints,
      images: research.images || []
    });

    // Step 5: Publish
    console.log('📤 Publishing...');
    await publishToFacebook({
      message: content.substring(0, 500),
      videoPath,
      scheduledTime: new Date(Date.now() + 3600000) // 1 hour later
    });

    return {
      success: true,
      content,
      videoPath,
      publishedAt: new Date()
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - enhance based on your needs
  const lines = content.split('\n');
  return lines
    .filter(line => line.match(/^\d+\.|^-|^•/))
    .slice(0, 5)
    .map(line => line.replace(/^\d+\.|^-|^•/, '').trim());
}
```

## API Routes (Next.js)

### Create Content API Endpoint

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

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
    console.error('API error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Research API Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');
  const sources = searchParams.get('sources')?.split(',') || ['techcrunch'];

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword parameter required' },
      { status: 400 }
    );
  }

  const research = await researchTopic({
    keyword,
    sources,
    timeRange: '24h'
  });

  return NextResponse.json(research);
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npm run remotion:render -- --props='{"title":"Test Video"}'
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLanguageContent(keyword: string) {
  const research = await researchTopic({ keyword });

  const [englishContent, vietnameseContent] = await Promise.all([
    generateContentWithClaude(research, 'toplist', 'en'),
    generateContentWithClaude(research, 'toplist', 'vi')
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
    research
  };
}
```

### Batch Content Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Custom Remotion Composition

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
      }}
    >
      <h1 style={{ color: 'white', fontSize: 80 }}>{title}</h1>
      <div style={{ marginTop: 40 }}>
        {points.map((point, i) => (
          <p
            key={i}
            style={{
              color: 'white',
              fontSize: 40,
              opacity: frame > (i + 1) * fps ? 1 : 0,
            }}
          >
            {i + 1}. {point}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries reached');
}

// Usage
const content = await withRetry(() => 
  generateContentWithClaude(research, 'toplist', 'en')
);
```

### Memory Issues with Video Rendering

```typescript
// Use serverless rendering for large videos
import { renderMediaOnLambda } from '@remotion/lambda';

async function renderOnCloud(composition: any) {
  const { renderId, bucketName } = await renderMediaOnLambda({
    region: 'us-east-1',
    functionName: process.env.REMOTION_LAMBDA_FUNCTION!,
    composition: composition.id,
    serveUrl: composition.serveUrl,
    codec: 'h264',
    inputProps: composition.inputProps,
  });

  return { renderId, bucketName };
}
```

### Debugging Research Crawlers

```typescript
// Enable verbose logging
async function debugResearch(keyword: string) {
  console.log(`[DEBUG] Starting research for: ${keyword}`);
  
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch'],
    debug: true // if supported
  });

  console.log('[DEBUG] Research results:', {
    insightsCount: research.insights?.length || 0,
    sourcesUsed: research.sources,
    dataPoints: research.dataPoints?.length || 0
  });

  return research;
}
```

This skill equips AI agents to effectively work with the Marketing Pipeline AI Content Automation system, from setup through advanced automation workflows.
