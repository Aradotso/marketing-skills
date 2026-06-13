---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline with Claude and Remotion
  - generate articles and videos automatically from keywords
  - crawl news sources and create content with AI
  - build automated marketing content workflow
  - create multilingual content with AI video rendering
  - use AI to research and generate social media videos
  - automate content from research to video export
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript project automates the entire content creation workflow: from researching trending topics across multiple news sources (TechCrunch, Twitter, LinkedIn), to generating multilingual articles using Claude/OpenAI, to rendering videos and infographics with Remotion. It's designed for content creators, marketers, and agencies who need to scale content production.

## What It Does

- **Auto-Research**: Crawls and analyzes recent content from major sources (24h window)
- **AI Writing**: Generates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual**: Produces content in both English and Vietnamese simultaneously
- **Video Generation**: Automatically renders videos and infographics from written content using Remotion
- **Multi-Platform**: Exports content optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# News/Data APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if using)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app directory
│   ├── lib/
│   │   ├── ai/             # AI providers (Claude, OpenAI)
│   │   ├── crawlers/       # News source crawlers
│   │   ├── generators/     # Content generation logic
│   │   └── video/          # Remotion video rendering
│   ├── components/         # React components
│   └── types/              # TypeScript definitions
├── remotion/               # Remotion video templates
└── public/                 # Static assets
```

## Core Workflow

### 1. Research & Data Collection

```typescript
import { TechCrunchCrawler } from '@/lib/crawlers/techcrunch';
import { TwitterCrawler } from '@/lib/crawlers/twitter';

async function gatherResearch(keyword: string) {
  const crawlers = [
    new TechCrunchCrawler(),
    new TwitterCrawler(process.env.TWITTER_BEARER_TOKEN!)
  ];

  const results = await Promise.all(
    crawlers.map(crawler => crawler.search(keyword, {
      timeWindow: '24h',
      limit: 10
    }))
  );

  // Aggregate and deduplicate results
  const aggregated = results.flat().reduce((acc, item) => {
    if (!acc.find(i => i.url === item.url)) {
      acc.push(item);
    }
    return acc;
  }, []);

  return aggregated;
}
```

### 2. AI Content Generation

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

interface ContentParams {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any[];
}

async function generateContent(params: ContentParams) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const prompt = buildPrompt(params);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(params: ContentParams): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a specific perspective with personal insights',
    'case-study': 'Analyze a real example with data and outcomes',
    'how-to': 'Provide step-by-step instructions with actionable tips'
  };

  const researchContext = params.researchData
    .map(item => `- ${item.title}: ${item.summary}`)
    .join('\n');

  return `
You are a ${params.tone} content creator writing in ${params.language}.

Format: ${formatInstructions[params.format]}

Topic: ${params.keyword}

Recent Research (Last 24h):
${researchContext}

Create a comprehensive article that:
1. Uses the latest data and trends from the research
2. Provides actionable insights
3. Includes specific examples and data points
4. Is optimized for social media sharing
`;
}
```

### 3. Multilingual Generation

```typescript
async function generateMultilingual(keyword: string, researchData: any[]) {
  const baseParams = {
    keyword,
    format: 'toplist' as const,
    tone: 'expert' as const,
    researchData
  };

  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ ...baseParams, language: 'en' }),
    generateContent({ ...baseParams, language: 'vi' })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/VideoComposition';

interface VideoParams {
  content: string;
  title: string;
  format: 'vertical' | 'square' | 'horizontal';
}

async function generateVideo(params: VideoParams) {
  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: require.resolve('@/remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      content: params.content,
      title: params.title
    }
  });

  // Render video
  const dimensions = {
    vertical: { width: 1080, height: 1920 },
    square: { width: 1080, height: 1080 },
    horizontal: { width: 1920, height: 1080 }
  };

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${params.title}-${params.format}.mp4`,
    ...dimensions[params.format]
  });
}
```

### 5. Remotion Video Template

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface VideoProps {
  title: string;
  content: string;
}

export const VideoComposition: React.FC<VideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  // Parse content into scenes
  const scenes = parseContentToScenes(content);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={90}>
        <TitleScene title={title} frame={frame} />
      </Sequence>
      
      {scenes.map((scene, index) => (
        <Sequence
          key={index}
          from={90 + (index * 120)}
          durationInFrames={120}
        >
          <ContentScene text={scene.text} image={scene.image} />
        </Sequence>
      ))}
      
      <Sequence from={90 + scenes.length * 120} durationInFrames={60}>
        <OutroScene />
      </Sequence>
    </AbsoluteFill>
  );
};

function parseContentToScenes(content: string) {
  const paragraphs = content.split('\n\n');
  return paragraphs.map(text => ({
    text: text.trim(),
    image: null // Could be generated or fetched
  }));
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, languages } = await req.json();

    // 1. Gather research
    const research = await gatherResearch(keyword);

    // 2. Generate content
    const content = await Promise.all(
      languages.map(lang => 
        generateContent({
          keyword,
          format,
          language: lang,
          tone: 'expert',
          researchData: research
        })
      )
    );

    // 3. Generate videos
    const videos = await Promise.all(
      content.map((text, i) =>
        generateVideo({
          content: text,
          title: keyword,
          format: 'vertical'
        })
      )
    );

    return NextResponse.json({
      success: true,
      content,
      videos
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
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    openaiKey: process.env.OPENAI_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  const result = await pipeline.execute({
    keyword,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    videoFormats: ['vertical', 'square'],
    tone: 'friendly'
  });

  console.log(`Generated ${result.articles.length} articles`);
  console.log(`Rendered ${result.videos.length} videos`);
  
  return result;
}
```

### Batch Processing

```typescript
async function batchProcess(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runFullPipeline(keyword);
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

## CLI Commands

If the project includes CLI tools:

```bash
# Generate content from keyword
npm run generate -- --keyword "AI trends 2024" --format toplist --lang en,vi

# Research only
npm run research -- --keyword "marketing automation" --sources techcrunch,twitter

# Render video from existing content
npm run render -- --input content.json --format vertical

# Run full pipeline
npm run pipeline -- --config pipeline-config.json
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  keywords.map(keyword => 
    limit(() => generateContent({ keyword, ... }))
  )
);
```

### Claude/OpenAI Errors

```typescript
async function generateWithRetry(params: ContentParams, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(params);
    } catch (error) {
      if (error.status === 429) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Use chunks for long content
async function renderLongContent(content: string) {
  const chunks = splitIntoChunks(content, 500); // 500 words per video
  
  return await Promise.all(
    chunks.map((chunk, i) =>
      generateVideo({
        content: chunk,
        title: `Part ${i + 1}`,
        format: 'vertical'
      })
    )
  );
}
```

### Crawler Blocking

```typescript
// Add delays and user agents
const crawler = new TechCrunchCrawler({
  delay: 1000,
  userAgent: 'Mozilla/5.0 (compatible; ContentBot/1.0)',
  maxRetries: 3
});
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function cachedResearch(keyword: string) {
  const cached = cache.get(keyword);
  if (cached) return cached;
  
  const results = await gatherResearch(keyword);
  cache.set(keyword, results);
  return results;
}
```

This skill enables AI agents to help developers automate content creation workflows from research through video generation using modern AI APIs and video rendering tools.
