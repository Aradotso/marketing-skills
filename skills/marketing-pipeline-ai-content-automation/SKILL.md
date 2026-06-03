---
name: marketing-pipeline-ai-content-automation
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I set up the AI content pipeline system
  - automate content creation from research to video
  - generate videos from articles using remotion
  - crawl news and create content with AI
  - set up automated marketing content workflow
  - create multilingual content with Claude API
  - build content automation pipeline with TypeScript
  - generate social media videos automatically
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers use the **Ultimate AI Content Pipeline** - an end-to-end content automation system that crawls news sources, generates articles in multiple formats and languages using Claude/OpenAI, and automatically renders videos using Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Uses Claude 3 or OpenAI to generate articles in multiple formats (Toplist, POV, Case Study, How-to)
3. **Multilingual Output**: Generates content simultaneously in English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders infographics and short-form videos using Remotion
5. **Multi-platform Export**: Exports videos optimized for Reels, TikTok, Shorts

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

## Configuration

Create a `.env` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for News Crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_CODEC=h264
REMOTION_CRF=23
```

Environment variables should reference:
- `process.env.ANTHROPIC_API_KEY` for Claude API
- `process.env.OPENAI_API_KEY` for OpenAI
- `process.env.RAPIDAPI_KEY` for news API access

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI content generation
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key Components & Usage

### 1. News Crawling & Research

```typescript
// src/lib/crawler/newsScanner.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  title: string;
  publishedAt: string;
  content: string;
}

export async function crawlNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsSource[]> {
  const options = {
    method: 'GET',
    url: 'https://news-api.p.rapidapi.com/search',
    params: {
      q: keyword,
      sources: sources.join(','),
      language: 'en',
      sortBy: 'publishedAt'
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'news-api.p.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles.map((article: any) => ({
      url: article.url,
      title: article.title,
      publishedAt: article.publishedAt,
      content: article.content || article.description
    }));
  } catch (error) {
    console.error('Error crawling news:', error);
    throw new Error('Failed to crawl news sources');
  }
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  researchData: string[];
}

export async function generateContent(
  options: ContentOptions
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const formatPrompts = {
    'toplist': 'Create a numbered list article',
    'pov': 'Write from a unique perspective',
    'case-study': 'Analyze a specific case with data',
    'how-to': 'Create a step-by-step guide'
  };

  const prompt = `
You are a content creator specializing in ${options.format} articles.

Keyword: ${options.keyword}
Format: ${formatPrompts[options.format]}
Language: ${options.language}
Tone: ${options.tone}

Research Data:
${options.researchData.join('\n\n')}

Create a compelling article that:
1. Uses the latest data from research
2. Follows the ${options.format} format
3. Maintains a ${options.tone} tone
4. Is written in ${options.language}
5. Includes actionable insights
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openaiGenerator.ts
import OpenAI from 'openai';

export async function generateContentOpenAI(
  options: ContentOptions
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: 'You are an expert content creator.'
    }, {
      role: 'user',
      content: `Create a ${options.format} article about ${options.keyword}...`
    }],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/videoRenderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  format: 'reels' | 'tiktok' | 'shorts';
}

const dimensionMap = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      ...dimensionMap[config.format]
    }
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps
  });

  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, interpolate, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  content 
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = interpolate(
    frame,
    [0, 30, durationInFrames - 30, durationInFrames],
    [0, 1, 1, 0]
  );

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        justifyContent: 'center',
        alignItems: 'center',
        opacity
      }}
    >
      <div style={{ padding: '60px', maxWidth: '80%' }}>
        <h1 
          style={{ 
            color: '#fff', 
            fontSize: '72px',
            fontWeight: 'bold',
            marginBottom: '40px',
            textAlign: 'center'
          }}
        >
          {title}
        </h1>
        {content.map((text, index) => (
          <p
            key={index}
            style={{
              color: '#e0e0e0',
              fontSize: '48px',
              marginBottom: '30px',
              lineHeight: 1.5
            }}
          >
            {text}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
// src/lib/pipeline/contentPipeline.ts
import { crawlNews } from '../crawler/newsScanner';
import { generateContent } from '../ai/contentGenerator';
import { renderContentVideo } from '../video/videoRenderer';

export async function runContentPipeline(
  keyword: string,
  outputDir: string
): Promise<{ article: string; videoPath: string }> {
  // Step 1: Crawl news
  console.log('Step 1: Crawling news sources...');
  const articles = await crawlNews(keyword, ['techcrunch', 'a16z']);
  
  const researchData = articles.map(
    article => `${article.title}\n${article.content}`
  );

  // Step 2: Generate content
  console.log('Step 2: Generating AI content...');
  const article = await generateContent({
    keyword,
    format: 'toplist',
    language: 'en',
    tone: 'professional',
    researchData
  });

  // Step 3: Generate Vietnamese version
  console.log('Step 3: Generating Vietnamese version...');
  const articleVi = await generateContent({
    keyword,
    format: 'toplist',
    language: 'vi',
    tone: 'professional',
    researchData
  });

  // Step 4: Extract key points for video
  const keyPoints = article
    .split('\n')
    .filter(line => line.match(/^\d+\./))
    .slice(0, 5);

  // Step 5: Render video
  console.log('Step 4: Rendering video...');
  const videoPath = await renderContentVideo(
    {
      title: keyword,
      content: keyPoints,
      format: 'reels'
    },
    `${outputDir}/output.mp4`
  );

  return { article, videoPath };
}
```

## Next.js API Routes

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/contentPipeline';
import path from 'path';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const outputDir = path.join(process.cwd(), 'public', 'outputs');
    const result = await runContentPipeline(keyword, outputDir);

    return NextResponse.json({
      success: true,
      article: result.article,
      videoUrl: `/outputs/${path.basename(result.videoPath)}`
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
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

# Render video only (Remotion CLI)
npx remotion render ContentVideo output.mp4
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple articles simultaneously
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => runContentPipeline(keyword, './outputs'))
  );
  return results;
}
```

### Custom Video Templates

```typescript
// Create different video styles
const videoStyles = {
  minimal: { backgroundColor: '#fff', textColor: '#000' },
  dark: { backgroundColor: '#1a1a2e', textColor: '#fff' },
  brand: { backgroundColor: '#4A90E2', textColor: '#fff' }
};

export function createStyledVideo(style: keyof typeof videoStyles) {
  const colors = videoStyles[style];
  // Apply to Remotion component
}
```

### Scheduling Content

```typescript
// Schedule content generation
import cron from 'node-cron';

cron.schedule('0 9 * * *', async () => {
  console.log('Running daily content generation...');
  await runContentPipeline('AI trends', './outputs');
});
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
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS=--max-old-space-size=4096 npm run dev
```

### Missing Environment Variables

```typescript
// Validate required env vars on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

### Remotion Bundling Errors

```typescript
// Clear Remotion cache
import { clearCache } from '@remotion/renderer';

await clearCache();
```

This skill provides comprehensive coverage of the Marketing Pipeline AI Content Automation system, enabling AI agents to help developers implement automated content workflows from research through video generation.
