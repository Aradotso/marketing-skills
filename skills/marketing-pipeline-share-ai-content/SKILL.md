---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content pipeline"
  - "create AI content workflow with research and video"
  - "generate content from research to video automatically"
  - "build marketing automation pipeline with AI"
  - "automate content creation with Claude and OpenAI"
  - "create video content from AI-generated articles"
  - "set up content research and generation system"
  - "build automated marketing content workflow"
---

# Marketing Pipeline Share - AI Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline**, a comprehensive TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to automated posting and video generation.

## What This Project Does

Marketing Pipeline Share is an all-in-one content automation system that:

- **Auto-scans and researches** trending topics from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Creates bilingual content** (English and Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

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

## Environment Configuration

Create a `.env` file with the following variables:

```env
# AI Services
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key_here

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities and services
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Content research modules
│   │   ├── video/       # Video generation with Remotion
│   │   └── content/     # Content generation logic
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research Module

```typescript
import { researchTopic } from '@/lib/research/scanner';

// Scan and research a topic
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 20
  });

  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    dataPoints: research.statistics
  };
}
```

### 2. Content Generation with AI

```typescript
import { Anthropic } from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

// Initialize AI clients
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// Generate content with Claude
async function generateContentWithClaude(
  research: Research,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const prompt = `Based on this research data:
${JSON.stringify(research, null, 2)}

Create a ${format} article in both English and Vietnamese.
Include data-backed insights and trending information.`;

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

  return message.content[0].text;
}

// Alternative: Generate with OpenAI
async function generateContentWithOpenAI(
  research: Research,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer. Create engaging content based on research data.`
      },
      {
        role: 'user',
        content: JSON.stringify(research)
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: Content) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: 'reels' // or 'tiktok', 'shorts'
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `output/${content.id}.mp4`,
    inputProps: composition.props,
  });

  return `output/${content.id}.mp4`;
}
```

### 4. Complete Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research/scanner';
import { generateContent } from '@/lib/ai/generator';
import { createVideo } from '@/lib/video/renderer';
import { publishContent } from '@/lib/publish/auto-post';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      research,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'expert'
    });

    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const videoPath = await createVideo({
      content: content.en,
      style: 'reels',
      aspectRatio: '9:16'
    });

    // Step 4: Publish
    console.log('📤 Publishing content...');
    await publishContent({
      content,
      video: videoPath,
      platforms: ['facebook', 'instagram', 'tiktok'],
      schedule: 'immediate'
    });

    return {
      success: true,
      content,
      videoPath
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## Common Patterns

### Multi-Format Content Generation

```typescript
import { generateMultiFormat } from '@/lib/content/generator';

async function createMultiFormatContent(keyword: string) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'] as const;
  
  const allContent = await Promise.all(
    formats.map(format => 
      generateMultiFormat({
        keyword,
        format,
        aiProvider: 'claude', // or 'openai'
        bilingual: true
      })
    )
  );

  return allContent;
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'Marketing automation', 'Content strategy'];
  
  for (const keyword of keywords) {
    await runContentPipeline(keyword);
  }
});
```

### Custom Video Templates

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  style: 'reels' | 'tiktok' | 'shorts';
}> = ({ title, points, style }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 40 }}>
        <h1>{title}</h1>
        {points.map((point, i) => (
          <p key={i}>{point}</p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## CLI Commands

```bash
# Development
npm run dev              # Start Next.js dev server
npm run build           # Build for production
npm run start           # Start production server

# Research
npm run research -- --keyword "AI trends"

# Content generation
npm run generate -- --format toplist --lang en,vi

# Video rendering
npm run render-video -- --input content.json

# Full pipeline
npm run pipeline -- --keyword "Marketing automation"
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scanner';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  const research = await researchTopic({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeframe: timeframe || '24h'
  });

  return NextResponse.json(research);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/generator';

export async function POST(request: NextRequest) {
  const { research, format, languages, tone } = await request.json();
  
  const content = await generateContent({
    research,
    format,
    languages,
    tone
  });

  return NextResponse.json(content);
}
```

## Troubleshooting

### AI API Rate Limits

```typescript
// Implement retry logic
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4000,
        messages: [{ role: 'user', content: prompt }]
      });
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Use chunked rendering for long videos
import { renderFrames } from '@remotion/renderer';

async function renderInChunks(composition: any, chunkSize = 300) {
  const totalFrames = composition.durationInFrames;
  
  for (let i = 0; i < totalFrames; i += chunkSize) {
    await renderFrames({
      composition,
      frameRange: [i, Math.min(i + chunkSize, totalFrames)],
      // ... other options
    });
  }
}
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearch(research: Research) {
  return {
    articles: research.articles.filter(a => 
      a.publishedAt && 
      new Date(a.publishedAt) > new Date(Date.now() - 24 * 60 * 60 * 1000)
    ),
    insights: research.insights.filter(i => i.confidence > 0.7),
    statistics: research.statistics.filter(s => s.source && s.value)
  };
}
```

## Best Practices

1. **Always validate research data** before content generation
2. **Cache research results** to avoid redundant API calls
3. **Use environment variables** for all API keys and secrets
4. **Implement rate limiting** for AI API calls
5. **Monitor video rendering** memory usage for long compositions
6. **Test content in both languages** before publishing
7. **Schedule pipelines** during off-peak hours to avoid rate limits
