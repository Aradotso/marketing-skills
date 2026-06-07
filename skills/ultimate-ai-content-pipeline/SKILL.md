---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - create automated content pipeline with AI
  - set up AI content generation workflow
  - generate videos from text content automatically
  - crawl news and create social media content
  - automate content research and scriptwriting
  - build AI-powered marketing content system
  - use Remotion to generate videos from articles
  - create multilingual content with Claude API
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

The Ultimate AI Content Pipeline is a complete automation system for content creation that handles everything from research (crawling TechCrunch, Twitter, LinkedIn) to scriptwriting (using Claude/OpenAI) to video generation (using Remotion). It creates multilingual content (English/Vietnamese) in various formats (listicles, POV, case studies) and automatically renders videos optimized for Reels, TikTok, and Shorts.

## Installation

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (optional)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion
```

## Core Architecture

### 1. Research Pipeline

The research module crawls and analyzes content from multiple sources:

```typescript
// lib/research/crawler.ts
import { OpenAI } from 'openai';

interface ResearchSource {
  name: string;
  url: string;
  type: 'news' | 'social' | 'blog';
}

async function crawlSources(keyword: string, sources: ResearchSource[]) {
  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await fetch(source.url, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
      });
      return response.json();
    })
  );
  
  return results.flat();
}

async function analyzeResearch(rawData: any[]) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'Extract key insights and trends from research data',
      },
      {
        role: 'user',
        content: JSON.stringify(rawData),
      },
    ],
  });
  
  return completion.choices[0].message.content;
}
```

### 2. Content Generation with Claude/OpenAI

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });
  
  return message.content[0].text;
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with detailed explanations',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze a real-world example with data and outcomes',
    'how-to': 'Provide step-by-step actionable instructions',
  };
  
  return `
    Create ${config.language === 'vi' ? 'Vietnamese' : 'English'} content about: ${config.keyword}
    Format: ${formatInstructions[config.format]}
    Tone: ${config.tone}
    Use this research data: ${config.researchData}
    
    Include:
    - Compelling headline
    - Introduction with hook
    - Main content with subheadings
    - Data-backed insights
    - Actionable takeaways
    - Call-to-action
  `;
}
```

### 3. Multilingual Content Generation

```typescript
// lib/content/multilingual.ts
async function generateMultilingualContent(keyword: string, researchData: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData,
    }),
    generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData,
    }),
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/Video.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoProps {
  title: string;
  points: string[];
  theme: 'dark' | 'light';
}

export const ContentVideo: React.FC<VideoProps> = ({ title, points, theme }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: theme === 'dark' ? '#1a1a1a' : '#ffffff',
        justifyContent: 'center',
        alignItems: 'center',
        fontFamily: 'Inter, sans-serif',
      }}
    >
      <div style={{ opacity, textAlign: 'center', padding: 40 }}>
        <h1 style={{ 
          fontSize: 64, 
          color: theme === 'dark' ? '#fff' : '#000',
          marginBottom: 40,
        }}>
          {title}
        </h1>
        {points.map((point, i) => {
          const pointFrame = frame - (i * fps);
          const pointOpacity = Math.min(1, Math.max(0, pointFrame / 20));
          
          return (
            <p
              key={i}
              style={{
                fontSize: 32,
                color: theme === 'dark' ? '#ccc' : '#333',
                opacity: pointOpacity,
                marginBottom: 20,
              }}
            >
              {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
}

async function renderContentVideo(config: RenderConfig) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: extractTitle(config.content),
      points: extractKeyPoints(config.content),
      theme: 'dark',
    },
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${config.format}.mp4`
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

function extractTitle(content: string): string {
  const lines = content.split('\n');
  return lines.find(line => line.startsWith('#'))?.replace('#', '').trim() || '';
}

function extractKeyPoints(content: string): string[] {
  const lines = content.split('\n');
  return lines
    .filter(line => line.match(/^\d+\.|^-|^•/))
    .map(line => line.replace(/^\d+\.|^-|^•/, '').trim())
    .slice(0, 5);
}
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline/orchestrator.ts
interface PipelineConfig {
  keyword: string;
  sources: ResearchSource[];
  contentFormats: ContentConfig['format'][];
  videoFormats: RenderConfig['format'][];
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const rawResearch = await crawlSources(config.keyword, config.sources);
  const insights = await analyzeResearch(rawResearch);
  
  // Step 2: Content Generation
  console.log('✍️ Generating content...');
  const contentPieces = await Promise.all(
    config.contentFormats.map(format =>
      generateMultilingualContent(config.keyword, insights)
    )
  );
  
  // Step 3: Video Rendering
  console.log('🎬 Rendering videos...');
  const videos = await Promise.all(
    contentPieces.flatMap(content =>
      config.videoFormats.map(format =>
        renderContentVideo({
          content: content.en,
          format,
        })
      )
    )
  );
  
  console.log('✅ Pipeline complete!');
  
  return {
    insights,
    content: contentPieces,
    videos,
  };
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI automation trends 2024',
  sources: [
    { name: 'TechCrunch', url: 'https://api.techcrunch.com/...', type: 'news' },
    { name: 'Twitter', url: 'https://api.twitter.com/...', type: 'social' },
  ],
  contentFormats: ['toplist', 'how-to'],
  videoFormats: ['reels', 'tiktok'],
});
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      sources: getDefaultSources(),
      contentFormats: [format],
      videoFormats: ['reels'],
    });
    
    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

// Run daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline({
      keyword: topic,
      sources: getDefaultSources(),
      contentFormats: ['toplist'],
      videoFormats: ['reels', 'shorts'],
    });
  }
});
```

### Batch Processing

```typescript
// lib/batch/processor.ts
async function processBatch(keywords: string[]) {
  const batchSize = 3;
  const results = [];
  
  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(keyword => runContentPipeline({
        keyword,
        sources: getDefaultSources(),
        contentFormats: ['toplist'],
        videoFormats: ['reels'],
      }))
    );
    results.push(...batchResults);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits
- Implement exponential backoff for API calls
- Use queue system (Bull, BullMQ) for processing
- Cache research data to avoid redundant API calls

### Remotion Rendering Issues
- Ensure sufficient memory (4GB+ recommended)
- Use `--overwrite` flag to replace existing videos
- Check Chrome/Chromium installation for headless rendering

### Content Quality
- Fine-tune prompts for specific niches
- Add example outputs in system prompts
- Use temperature parameter to control creativity (0.7-0.9 recommended)

### Video Performance
- Optimize bundle size by removing unused components
- Use `--concurrency` flag for parallel rendering
- Preload fonts and assets to prevent rendering delays

### Database Connection
- Use connection pooling for better performance
- Implement retry logic for transient failures
- Store generated content with metadata for tracking
