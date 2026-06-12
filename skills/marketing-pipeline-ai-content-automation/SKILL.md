---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline for marketing
  - generate videos from blog content automatically
  - research and create content with Claude API
  - build automated marketing content workflow
  - create AI-powered content with Remotion videos
  - scrape news and generate social media posts
  - automate content from research to publication
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a TypeScript-based automation system that transforms a single keyword into complete content packages including research, scripts, and rendered videos. It leverages Claude 3/OpenAI for content generation and Remotion for automated video creation.

## What It Does

This system provides an end-to-end content creation pipeline:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter, LinkedIn
2. **AI Content Generation**: Creates multiple content formats (toplists, POV articles, case studies, how-tos) in multiple languages
3. **Automated Video Rendering**: Converts written content into videos/infographics using Remotion
4. **Multi-Platform Export**: Outputs optimized content for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install

# Set up environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following required variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Content scraping logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
import { scrapeLatestNews } from '@/lib/scraper/news-scraper';

async function gatherResearch(keyword: string) {
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const research = await scrapeLatestNews({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });
  
  return research;
}

// Example output structure
interface ResearchResult {
  articles: Array<{
    title: string;
    url: string;
    summary: string;
    publishedAt: Date;
    source: string;
  }>;
  insights: string[];
  trends: string[];
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Create a ${format} article about ${topic} in ${language}.
  Use the following research data: ${researchData}
  
  Requirements:
  - Data-backed insights
  - Engaging tone
  - SEO optimized
  - Include relevant statistics`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. OpenAI Alternative Implementation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentOpenAI(
  topic: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creator with a ${tone} tone. Generate engaging marketing content.`
      },
      {
        role: 'user',
        content: `Write about: ${topic}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  content: string,
  outputPath: string
) {
  // Bundle Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      theme: 'modern'
    }
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps,
  });

  return outputPath;
}
```

### 5. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    formats: ['toplist', 'how-to'],
    generateVideo: true
  });

  try {
    // Step 1: Research
    const research = await pipeline.research(keyword);
    console.log(`Found ${research.articles.length} articles`);

    // Step 2: Generate content
    const content = await pipeline.generateContent({
      topic: keyword,
      research,
      format: 'toplist',
      language: 'en'
    });

    // Step 3: Create video
    const video = await pipeline.renderVideo({
      content,
      platform: 'youtube-shorts', // or 'tiktok', 'reels'
      duration: 60
    });

    // Step 4: Export
    return {
      article: content,
      videoPath: video.outputPath,
      metadata: {
        keyword,
        createdAt: new Date(),
        sources: research.articles.length
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video
npm run render -- --props='{"title":"My Video"}'

# Type checking
npm run type-check

# Linting
npm run lint
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runFullPipeline(keyword))
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);

  const failed = results
    .filter(r => r.status === 'rejected')
    .map((r, i) => ({ keyword: keywords[i], error: r.reason }));

  return { successful, failed };
}
```

### Custom Content Templates

```typescript
interface ContentTemplate {
  format: string;
  structure: string[];
  tone: string;
  targetLength: number;
}

const templates: Record<string, ContentTemplate> = {
  'product-launch': {
    format: 'announcement',
    structure: ['hook', 'problem', 'solution', 'features', 'cta'],
    tone: 'exciting',
    targetLength: 800
  },
  'thought-leadership': {
    format: 'pov',
    structure: ['observation', 'analysis', 'prediction', 'advice'],
    tone: 'expert',
    targetLength: 1200
  }
};

async function generateFromTemplate(
  keyword: string,
  templateName: keyof typeof templates
) {
  const template = templates[templateName];
  const prompt = buildPromptFromTemplate(keyword, template);
  return await generateContent(prompt, template.format, 'en');
}
```

### Video Style Customization

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoProps {
  title: string;
  points: string[];
  theme: 'modern' | 'minimal' | 'vibrant';
}

export const ContentVideo: React.FC<VideoProps> = ({ title, points, theme }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill className={`theme-${theme}`}>
      <div style={{ opacity }}>
        <h1>{title}</h1>
        {points.map((point, i) => (
          <p key={i} style={{ 
            opacity: frame > i * fps ? 1 : 0,
            transition: 'opacity 0.5s'
          }}>
            {point}
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
import pRetry from 'p-retry';

async function generateWithRetry(prompt: string) {
  return pRetry(
    async () => {
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }]
      });
    },
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
}
```

### Video Rendering Memory Issues

```typescript
// Reduce memory usage for large batches
const renderConfig = {
  concurrency: 1, // Render one video at a time
  frameRange: [0, 150], // Limit to 5 seconds at 30fps
  quality: 80 // Reduce quality slightly
};
```

### Content Quality Validation

```typescript
function validateContent(content: string): boolean {
  const checks = {
    minLength: content.length > 300,
    hasHeadings: content.includes('#'),
    hasData: /\d+%|\d+[kKmMbB]/.test(content),
    notEmpty: content.trim().length > 0
  };

  const passed = Object.values(checks).filter(Boolean).length;
  return passed >= 3; // At least 3 checks must pass
}
```

### Scraping Error Handling

```typescript
async function safeScrape(url: string) {
  try {
    const response = await fetch(url, {
      headers: { 'User-Agent': 'Mozilla/5.0...' },
      signal: AbortSignal.timeout(10000) // 10s timeout
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    return await response.text();
  } catch (error) {
    console.error(`Failed to scrape ${url}:`, error);
    return null;
  }
}
```

## Integration Example

Complete Next.js API route for content generation:

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, generateVideo } = await request.json();

    const pipeline = new ContentPipeline({
      aiProvider: 'claude',
      languages: [language],
      formats: [format],
      generateVideo
    });

    const result = await pipeline.run(keyword);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

This skill enables AI coding agents to help developers build and customize automated content marketing pipelines using this TypeScript-based system.
