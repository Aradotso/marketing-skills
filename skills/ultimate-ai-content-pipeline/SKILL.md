---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese and English content
triggers:
  - how do I generate AI-driven content from research to video
  - automate content pipeline with Claude and OpenAI
  - create Vietnamese and English content automatically
  - generate videos from blog posts using Remotion
  - set up auto-research content system
  - build content automation with AI crawling
  - create multilingual marketing content with AI
  - automate TechCrunch and Twitter content research
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end automated content generation system that crawls news sources (TechCrunch, a16z, Twitter/X, LinkedIn), generates multilingual content (Vietnamese & English) using Claude/OpenAI, and automatically renders videos/infographics with Remotion. Built with TypeScript and Next.js.

## What It Does

This pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls fresh data from major tech news sources within the last 24 hours
2. **AI Content Generation**: Creates diverse formats (listicles, POV pieces, case studies, how-tos) in multiple languages
3. **Video Rendering**: Automatically generates videos and infographics using Remotion
4. **Multi-Platform Export**: Outputs content optimized for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```env
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Remotion configuration
REMOTION_CONCURRENCY=4
REMOTION_QUALITY=80

# Database (if needed)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News source crawlers
│   │   ├── generators/  # Content generators
│   │   └── remotion/    # Video rendering
│   ├── utils/           # Utility functions
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key API Usage

### 1. Auto-Research Crawler

```typescript
import { crawlTechNews } from '@/lib/crawlers/techcrunch';
import { crawlTwitter } from '@/lib/crawlers/twitter';

// Crawl TechCrunch for recent AI news
async function gatherResearch(keyword: string) {
  const techCrunchData = await crawlTechNews({
    keyword,
    timeRange: '24h',
    limit: 10
  });

  const twitterData = await crawlTwitter({
    query: keyword,
    maxResults: 20,
    lang: 'en'
  });

  return {
    articles: techCrunchData,
    tweets: twitterData
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateContent(research: any, format: string, language: string) {
  const prompt = `
You are a content expert. Based on this research data:
${JSON.stringify(research)}

Create a ${format} format article in ${language} language.
Target audience: Tech-savvy marketers
Tone: Professional yet engaging
Include: Data-backed insights, actionable takeaways
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(research: any, options: {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
}) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${options.tone} content creator specializing in ${options.format} articles.`
      },
      {
        role: 'user',
        content: `Create a ${options.format} in ${options.language} based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Remotion Video Generation

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion.config';

async function generateVideo(content: {
  title: string;
  points: string[];
  images: string[];
}) {
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.points,
      images: content.images
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.title}.mp4`,
    concurrency: parseInt(process.env.REMOTION_CONCURRENCY || '4')
  });
}
```

### 5. Complete Pipeline

```typescript
import { gatherResearch } from '@/lib/crawlers';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/remotion';
import { extractKeyPoints } from '@/lib/utils/content-parser';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Gathering research...');
  const research = await gatherResearch(keyword);

  // Step 2: Generate bilingual content
  console.log('✍️ Generating content...');
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, 'toplist', 'en'),
    generateContent(research, 'toplist', 'vi')
  ]);

  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(englishContent);

  // Step 4: Generate video
  console.log('🎬 Rendering video...');
  await generateVideo({
    title: keyword,
    points: keyPoints,
    images: research.articles.slice(0, 5).map(a => a.imageUrl)
  });

  return {
    en: englishContent,
    vi: vietnameseContent,
    video: `out/${keyword}.mp4`
  };
}
```

## Common Patterns

### Content Format Templates

```typescript
type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';

const formatPrompts: Record<ContentFormat, string> = {
  'toplist': 'Create a numbered list article with compelling headlines for each point',
  'pov': 'Write from a unique perspective with strong opinions and personal insights',
  'case-study': 'Analyze real-world examples with data, challenges, and outcomes',
  'how-to': 'Provide step-by-step actionable instructions with screenshots and tips'
};

function buildPrompt(format: ContentFormat, research: any): string {
  return `${formatPrompts[format]}\n\nResearch data:\n${JSON.stringify(research, null, 2)}`;
}
```

### Multilingual Generation

```typescript
async function generateBilingualContent(
  research: any,
  format: ContentFormat
) {
  const basePrompt = buildPrompt(format, research);

  const [en, vi] = await Promise.all([
    generateContent(basePrompt, 'en', 'professional'),
    generateContent(basePrompt, 'vi', 'friendly')
  ]);

  return { en, vi };
}
```

### News Source Configuration

```typescript
interface CrawlerConfig {
  source: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  endpoint: string;
  headers: Record<string, string>;
}

const crawlerConfigs: Record<string, CrawlerConfig> = {
  techcrunch: {
    source: 'techcrunch',
    endpoint: 'https://techcrunch.com/wp-json/wp/v2/posts',
    headers: {
      'User-Agent': 'Mozilla/5.0'
    }
  },
  twitter: {
    source: 'twitter',
    endpoint: 'https://api.twitter.com/2/tweets/search/recent',
    headers: {
      'Authorization': `Bearer ${process.env.TWITTER_BEARER_TOKEN}`
    }
  }
};
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video
npm run remotion:render

# Preview Remotion composition
npm run remotion:preview
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  const { keyword, format, languages } = await request.json();

  try {
    const result = await runContentPipeline(keyword);
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Usage from Frontend

```typescript
async function generateContentFromUI(keyword: string) {
  const response = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi']
    })
  });

  const data = await response.json();
  return data;
}
```

## Remotion Video Template Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface Props {
  title: string;
  points: string[];
  images: string[];
}

export const ContentVideo: React.FC<Props> = ({ title, points, images }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{ 
          fontSize: 60, 
          color: 'white', 
          textAlign: 'center',
          padding: 40
        }}>
          {title}
        </div>
      </Sequence>

      {points.map((point, idx) => (
        <Sequence 
          key={idx} 
          from={60 + idx * 90} 
          durationInFrames={90}
        >
          <div style={{ padding: 40, color: 'white' }}>
            <h2>{idx + 1}. {point}</h2>
            {images[idx] && <img src={images[idx]} alt="" />}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Add rate limiting with delay
async function crawlWithRateLimit(sources: string[]) {
  const results = [];
  for (const source of sources) {
    const data = await crawl(source);
    results.push(data);
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
  }
  return results;
}
```

### Claude API Errors

```typescript
// Add retry logic
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(prompt);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

### Remotion Memory Issues

```bash
# Increase Node.js memory
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render

# Reduce concurrency
REMOTION_CONCURRENCY=1 npm run remotion:render
```

### Missing API Keys

```typescript
// Validate environment variables on startup
function validateEnv() {
  const required = ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'RAPIDAPI_KEY'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}

validateEnv();
```

## Performance Optimization

```typescript
// Parallel processing
async function optimizedPipeline(keyword: string) {
  const [research, images] = await Promise.all([
    gatherResearch(keyword),
    fetchStockImages(keyword)
  ]);

  const [enContent, viContent] = await Promise.all([
    generateContent(research, 'toplist', 'en'),
    generateContent(research, 'toplist', 'vi')
  ]);

  // Video generation can run in background
  generateVideo({ title: keyword, points: [], images }).catch(console.error);

  return { en: enContent, vi: viContent };
}
```

This skill enables AI coding agents to help developers set up and use the Ultimate AI Content Pipeline for automated, multilingual content generation with integrated video rendering.
