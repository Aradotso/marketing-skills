---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scripting, and video generation with Claude/OpenAI integration
triggers:
  - how do I automate content creation with this pipeline
  - set up the marketing pipeline for automatic content generation
  - generate videos from text using this content automation system
  - create automated social media content with AI research
  - configure the content pipeline with Claude and OpenAI
  - use remotion to render videos from content automatically
  - crawl news sources and generate content automatically
  - set up the ultimate AI content pipeline
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based automation system that handles the complete content creation workflow: from news research and scraping, to AI-powered content generation (Claude/OpenAI), to automatic video rendering with Remotion.

## What This Project Does

The Marketing Pipeline automates:
1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter/X, LinkedIn) for recent content
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Generation**: Automatically renders videos and infographics using Remotion
5. **Social Media Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database (if using)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── scraper/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Content Research & Scraping

```typescript
import { researchTopic } from '@/lib/scraper/research';

async function gatherNewsData(keyword: string) {
  const researchData = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h', // Last 24 hours
    limit: 10
  });

  return researchData;
}

// Example usage
const data = await gatherNewsData('AI automation');
console.log(data.articles); // Array of scraped articles
console.log(data.insights); // Extracted insights
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  topic: string,
  format: 'toplist' | 'pov' | 'casestudy' | 'howto',
  language: 'en' | 'vi'
) {
  const prompt = `
You are an expert content writer. Create a ${format} article about "${topic}".
Language: ${language}
Include data-backed insights and recent trends.
Format the output as structured JSON with: title, introduction, sections, conclusion.
`;

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

  const content = message.content[0];
  if (content.type === 'text') {
    return JSON.parse(content.text);
  }
}

// Example usage
const article = await generateContentWithClaude(
  'Marketing Automation Trends 2026',
  'toplist',
  'en'
);
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(
  researchData: any[],
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content writer with a ${tone} tone. Create engaging content based on research data.`
      },
      {
        role: 'user',
        content: JSON.stringify(researchData)
      }
    ],
    response_format: { type: 'json_object' }
  });

  return JSON.parse(completion.choices[0].message.content || '{}');
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';
import path from 'path';

async function generateVideo(contentData: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride,
  });

  // Select composition
  const compositionId = 'ContentVideo';
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: contentData.title,
      sections: contentData.sections,
      style: 'modern'
    },
  });

  // Render video
  const outputLocation = `out/${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}

// Example usage
const videoPath = await generateVideo({
  title: '5 Marketing Trends in 2026',
  sections: [
    { heading: 'AI Automation', content: '...' },
    { heading: 'Personalization', content: '...' }
  ]
});
```

### 5. Complete Pipeline Integration

```typescript
import { researchTopic } from '@/lib/scraper/research';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/renderer';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h'
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContentWithClaude(
      keyword,
      'toplist',
      'en'
    );

    // Step 3: Create Video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo(content);

    return {
      research,
      content,
      videoPath,
      status: 'success'
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline('AI Content Marketing');
```

## Development Server

```bash
# Run Next.js development server
npm run dev

# Access at http://localhost:3000
```

## API Routes (Next.js)

### Create Content API Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    const result = await runContentPipeline(keyword);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Usage from Client

```typescript
// Client-side usage
async function generateContent(keyword: string) {
  const response = await fetch('/api/generate', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      keyword,
      format: 'toplist',
      language: 'en'
    })
  });

  const result = await response.json();
  return result.data;
}
```

## Remotion Video Templates

### Basic Video Composition

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  sections: Array<{ heading: string; content: string }>;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, sections }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        padding: 80, 
        opacity,
        color: 'white',
        fontSize: 48,
        fontWeight: 'bold'
      }}>
        {title}
      </div>
      {sections.map((section, idx) => (
        <div key={idx} style={{ 
          padding: 40,
          opacity: frame > (idx + 1) * fps ? 1 : 0,
          transition: 'opacity 0.5s'
        }}>
          <h2>{section.heading}</h2>
          <p>{section.content}</p>
        </div>
      ))}
    </AbsoluteFill>
  );
};
```

### Register Composition

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Default Title',
          sections: []
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateBilingualContent(topic: string) {
  const [enContent, viContent] = await Promise.all([
    generateContentWithClaude(topic, 'toplist', 'en'),
    generateContentWithClaude(topic, 'toplist', 'vi')
  ]);

  return {
    english: enContent,
    vietnamese: viContent
  };
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'Marketing automation', 'SaaS growth'];
  
  for (const keyword of keywords) {
    await runContentPipeline(keyword);
  }
});
```

### Error Handling & Retry Logic

```typescript
async function runPipelineWithRetry(
  keyword: string,
  maxRetries = 3
) {
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await runContentPipeline(keyword);
    } catch (error) {
      lastError = error;
      console.log(`Attempt ${i + 1} failed, retrying...`);
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  
  throw lastError;
}
```

## Troubleshooting

### API Key Issues
- Ensure all API keys are set in `.env.local`
- Verify keys have proper permissions (Claude, OpenAI, RapidAPI)
- Check API rate limits and quotas

### Remotion Rendering Errors
```bash
# Install required dependencies
npm install @remotion/cli @remotion/bundler @remotion/renderer

# Check ffmpeg installation
remotion versions
```

### Memory Issues with Large Videos
```typescript
// Reduce video resolution or duration
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: compositionId,
  inputProps: {
    // Reduce complexity
    maxSections: 5,
    simplifiedGraphics: true
  },
});
```

### Scraping/Research Failures
- Verify RapidAPI subscription is active
- Check if source websites have changed their structure
- Implement fallback data sources
- Add request throttling to avoid rate limits

```typescript
// Add delay between requests
async function scrapeWithDelay(sources: string[]) {
  const results = [];
  for (const source of sources) {
    results.push(await scrapeSource(source));
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
  }
  return results;
}
```

### TypeScript Errors
```bash
# Regenerate types
npm run type-check

# Update dependencies
npm update
```

## Production Deployment

```bash
# Build for production
npm run build

# Start production server
npm run start
```

For video rendering in production, consider using Remotion Lambda or a dedicated rendering service to handle the computational load.
