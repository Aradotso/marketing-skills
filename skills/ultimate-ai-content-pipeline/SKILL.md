---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - automate content creation pipeline
  - generate marketing content with AI
  - create videos from articles automatically
  - research and write blog posts with AI
  - build automated content workflow
  - set up AI content generation system
  - scrape news and generate content
  - remotion video generation from content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline crawls recent news, generates multi-format articles using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-scans** news sources (TechCrunch, a16z, Twitter/X, LinkedIn) for trending topics
- **Generates** multi-format content (listicles, POV articles, case studies, how-tos) in English and Vietnamese
- **Renders** videos and infographics automatically using Remotion
- **Optimizes** output for multiple platforms (Reels, TikTok, Shorts)

The system uses Next.js for the frontend, TypeScript for the core logic, and integrates with OpenAI, Anthropic (Claude), and RapidAPI.

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
cp .env.example .env.local
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Core Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (OpenAI, Claude)
│   │   ├── research/    # Web scraping & research
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Research & Scraping

### Crawl News Sources

```typescript
import { researchTopic } from '@/lib/research/crawler';

interface ResearchResult {
  sources: Array<{
    title: string;
    url: string;
    excerpt: string;
    publishedAt: string;
    source: string;
  }>;
  insights: string[];
  trends: string[];
}

async function gatherResearch(topic: string): Promise<ResearchResult> {
  const research = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });
  
  return research;
}

// Usage
const data = await gatherResearch('AI content marketing');
console.log(`Found ${data.sources.length} articles`);
```

### Custom News Scraper

```typescript
import axios from 'axios';
import { parseHTML } from '@/lib/utils/parser';

interface ScraperConfig {
  url: string;
  selectors: {
    title: string;
    content: string;
    date: string;
  };
}

async function scrapeArticle(config: ScraperConfig) {
  const response = await axios.get(config.url);
  const parsed = parseHTML(response.data, config.selectors);
  
  return {
    title: parsed.title,
    content: parsed.content,
    publishedAt: new Date(parsed.date)
  };
}
```

## AI Content Generation

### Generate Article with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'listicle' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: ResearchResult;
}

async function generateArticle(req: ContentRequest): Promise<string> {
  const prompt = `
You are a content marketing expert. Generate a ${req.format} article about "${req.topic}".

Tone: ${req.tone}
Language: ${req.language}
Use these recent insights: ${JSON.stringify(req.researchData.insights)}

Format requirements:
- Include headline and subheadings
- Use data points from research
- ${req.format === 'listicle' ? 'Create 7-10 items' : ''}
- ${req.format === 'how-to' ? 'Step-by-step instructions' : ''}
- Optimize for SEO
`;

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
```

### Generate with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(req: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${req.tone} content marketer writing in ${req.language}.`
      },
      {
        role: 'user',
        content: `Create a ${req.format} about ${req.topic} using these insights: ${JSON.stringify(req.researchData)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

## Video Generation with Remotion

### Define Video Composition

```typescript
// remotion/compositions/ArticleVideo.tsx
import { Composition } from 'remotion';
import { ArticleScene } from './scenes/ArticleScene';

export const ArticleVideo: React.FC<{
  title: string;
  points: string[];
  backgroundColor: string;
}> = ({ title, points, backgroundColor }) => {
  return (
    <div style={{ backgroundColor, width: '100%', height: '100%' }}>
      <ArticleScene title={title} points={points} />
    </div>
  );
};

export const articleComposition = {
  id: 'ArticleVideo',
  component: ArticleVideo,
  durationInFrames: 300, // 10 seconds at 30fps
  fps: 30,
  width: 1080,
  height: 1920, // Vertical for Reels/TikTok
};
```

### Render Video Programmatically

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  title: string;
  points: string[];
  outputPath: string;
}

async function renderArticleVideo(options: VideoRenderOptions) {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: {
      title: options.title,
      points: options.points,
      backgroundColor: '#1a1a2e',
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: options.outputPath,
    inputProps: composition.props,
  });

  return options.outputPath;
}

// Usage
const videoPath = await renderArticleVideo({
  title: 'Top 10 AI Marketing Tools',
  points: ['Tool 1', 'Tool 2', 'Tool 3'],
  outputPath: './output/video.mp4'
});
```

## Complete Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateArticle } from '@/lib/ai/claude';
import { renderArticleVideo } from '@/lib/video/remotion';
import { parseMarkdownToPoints } from '@/lib/utils/parser';

interface PipelineConfig {
  topic: string;
  format: 'listicle' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  generateVideo: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword: config.topic,
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h',
  });

  // Step 2: Generate content
  console.log('✍️ Generating article...');
  const article = await generateArticle({
    topic: config.topic,
    format: config.format,
    tone: 'expert',
    language: config.language,
    researchData: research,
  });

  // Step 3: Save article
  const articlePath = `./content/${Date.now()}-${config.topic.replace(/\s/g, '-')}.md`;
  await fs.writeFile(articlePath, article, 'utf-8');
  console.log(`📄 Article saved: ${articlePath}`);

  // Step 4: Generate video (optional)
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    const points = parseMarkdownToPoints(article);
    const videoPath = await renderArticleVideo({
      title: config.topic,
      points: points.slice(0, 5), // First 5 points
      outputPath: `./videos/${Date.now()}.mp4`,
    });
    console.log(`🎥 Video saved: ${videoPath}`);
    
    return { article: articlePath, video: videoPath };
  }

  return { article: articlePath };
}

// Execute pipeline
const result = await runContentPipeline({
  topic: 'AI Marketing Automation Trends 2024',
  format: 'listicle',
  language: 'en',
  generateVideo: true,
});
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateArticle } from '@/lib/ai/claude';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { topic, format, language } = body;

    // Validate input
    if (!topic || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Run pipeline
    const research = await researchTopic({
      keyword: topic,
      sources: ['techcrunch'],
      timeRange: '24h',
    });

    const article = await generateArticle({
      topic,
      format,
      tone: 'expert',
      language: language || 'en',
      researchData: research,
    });

    return NextResponse.json({
      success: true,
      article,
      sources: research.sources.length,
    });
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Render Video Endpoint

```typescript
// src/app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderArticleVideo } from '@/lib/video/remotion';

export async function POST(request: NextRequest) {
  try {
    const { title, points } = await request.json();

    const outputPath = `./public/videos/${Date.now()}.mp4`;
    await renderArticleVideo({ title, points, outputPath });

    return NextResponse.json({
      success: true,
      videoUrl: outputPath.replace('./public', ''),
    });
  } catch (error) {
    console.error('Render error:', error);
    return NextResponse.json(
      { error: 'Failed to render video' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(topic => 
      runContentPipeline({
        topic,
        format: 'listicle',
        language: 'en',
        generateVideo: false,
      })
    )
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  console.log(`✅ Success: ${successful.length}`);
  console.log(`❌ Failed: ${failed.length}`);

  return { successful, failed };
}
```

### Multi-language Content

```typescript
async function generateMultiLanguage(topic: string) {
  const languages = ['en', 'vi'] as const;
  
  const articles = await Promise.all(
    languages.map(lang =>
      generateArticle({
        topic,
        format: 'pov',
        tone: 'friendly',
        language: lang,
        researchData: await researchTopic({ keyword: topic }),
      })
    )
  );

  return {
    en: articles[0],
    vi: articles[1],
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
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

### Video Rendering Memory Issues

```typescript
// For large video renders, use lower concurrency
import { renderMedia } from '@remotion/renderer';

await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  concurrency: 2, // Lower concurrency for memory-constrained environments
  imageFormat: 'jpeg', // Use JPEG instead of PNG to reduce memory
});
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

This skill enables AI agents to effectively use the Ultimate AI Content Pipeline for automated content creation, from research through video generation.
