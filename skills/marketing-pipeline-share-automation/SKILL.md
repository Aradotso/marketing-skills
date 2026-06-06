---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the marketing content automation pipeline
  - help me automate content creation with AI research and video generation
  - configure the AI content pipeline for auto-posting
  - generate automated content from research to video
  - set up Claude and OpenAI for content automation
  - create automated marketing videos with Remotion
  - build an AI-powered content generation workflow
  - automate content research and scriptwriting pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline crawls news sources, generates multi-format content in multiple languages, and renders videos automatically.

## What This Project Does

The Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (Top Lists, POV, Case Studies, How-To) using Claude 3 and OpenAI
3. **Multi-Language Output**: Generates content in both English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders infographics and short-form videos using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Yarn or npm
yarn --version
```

### Clone and Install

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
yarn install
# or
npm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research API
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using)
DATABASE_URL=postgresql://user:password@localhost:5432/marketing_pipeline

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_here
```

### Run Development Server

```bash
yarn dev
# or
npm run dev
```

Access the application at `http://localhost:3000`

## Key Architecture

```typescript
// Project structure
marketing-pineline-share/
├── src/
│   ├── app/              // Next.js app router
│   ├── components/       // React components
│   ├── lib/
│   │   ├── ai/          // AI provider integrations
│   │   ├── research/    // Content crawling
│   │   └── video/       // Remotion video generation
│   └── types/           // TypeScript definitions
├── public/              // Static assets
└── remotion/            // Video templates
```

## Core APIs and Usage

### 1. Research & Content Crawling

```typescript
// src/lib/research/crawler.ts
import { fetchLatestNews } from '@/lib/research/crawler';

interface NewsSource {
  url: string;
  source: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeframe?: number; // hours
}

async function gatherResearch(keyword: string): Promise<ResearchData> {
  const sources: NewsSource[] = [
    { url: 'https://techcrunch.com/search/' + keyword, source: 'techcrunch', timeframe: 24 },
    { url: 'https://a16z.com/search/' + keyword, source: 'a16z', timeframe: 24 }
  ];

  const newsData = await fetchLatestNews(sources);
  
  return {
    keyword,
    articles: newsData,
    timestamp: new Date(),
    totalSources: sources.length
  };
}

// Usage
const research = await gatherResearch('AI marketing automation');
console.log(`Found ${research.articles.length} articles`);
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  researchData: ResearchData;
}

async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);
  
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

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a top 10 list with detailed explanations',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze with real examples and data',
    'how-to': 'Step-by-step tutorial with actionable tips'
  };

  return `
Topic: ${request.topic}
Format: ${formatInstructions[request.format]}
Language: ${request.language === 'vi' ? 'Vietnamese' : 'English'}
Tone: ${request.tone}

Research Data:
${JSON.stringify(request.researchData, null, 2)}

Generate compelling content based on the latest research. Include:
- Data-backed insights
- Recent trends (last 24h)
- Actionable takeaways
`;
}

// Usage example
const content = await generateContent({
  topic: 'AI Marketing Trends 2026',
  format: 'toplist',
  language: 'en',
  tone: 'professional',
  researchData: research
});
```

### 3. OpenAI Integration (Alternative)

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} format. Write in ${request.language} with a ${request.tone} tone.`
      },
      {
        role: 'user',
        content: buildPrompt(request)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  style: 'infographic' | 'text-overlay' | 'slideshow';
  platform: 'reels' | 'tiktok' | 'youtube-shorts';
}

const platformDimensions = {
  'reels': { width: 1080, height: 1920 },
  'tiktok': { width: 1080, height: 1920 },
  'youtube-shorts': { width: 1080, height: 1920 }
};

async function generateVideo(config: VideoConfig): Promise<string> {
  const compositionId = `content-video-${config.style}`;
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style
    }
  });

  const dimensions = platformDimensions[config.platform];
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
      content: config.content
    },
    ...dimensions
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  title: 'Top 10 AI Marketing Tools',
  content: content,
  style: 'infographic',
  platform: 'reels'
});
```

### 5. Full Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
interface PipelineConfig {
  keyword: string;
  contentFormats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'>;
  languages: Array<'en' | 'vi'>;
  generateVideo: boolean;
  platforms?: Array<'reels' | 'tiktok' | 'youtube-shorts'>;
}

async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for: ${config.keyword}`);

  // Step 1: Research
  console.log('📡 Gathering research...');
  const research = await gatherResearch(config.keyword);

  // Step 2: Generate content for each format and language
  const contentOutputs = [];
  
  for (const format of config.contentFormats) {
    for (const language of config.languages) {
      console.log(`🧠 Generating ${format} in ${language}...`);
      
      const content = await generateContent({
        topic: config.keyword,
        format,
        language,
        tone: 'professional',
        researchData: research
      });

      contentOutputs.push({
        format,
        language,
        content,
        metadata: {
          wordCount: content.split(' ').length,
          generatedAt: new Date()
        }
      });
    }
  }

  // Step 3: Generate videos if requested
  const videoOutputs = [];
  
  if (config.generateVideo && config.platforms) {
    for (const output of contentOutputs) {
      if (output.language === 'en') { // Only generate videos for English
        for (const platform of config.platforms) {
          console.log(`🎬 Rendering video for ${platform}...`);
          
          const videoPath = await generateVideo({
            title: `${config.keyword} - ${output.format}`,
            content: output.content,
            style: 'infographic',
            platform
          });

          videoOutputs.push({
            platform,
            format: output.format,
            path: videoPath
          });
        }
      }
    }
  }

  console.log('✅ Pipeline complete!');
  
  return {
    research,
    content: contentOutputs,
    videos: videoOutputs,
    summary: {
      totalArticles: contentOutputs.length,
      totalVideos: videoOutputs.length,
      completedAt: new Date()
    }
  };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI Content Marketing 2026',
  contentFormats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  platforms: ['reels', 'tiktok']
});

console.log(`Generated ${result.summary.totalArticles} articles and ${result.summary.totalVideos} videos`);
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormats: body.formats || ['toplist'],
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo ?? false,
      platforms: body.platforms || ['reels']
    });

    return NextResponse.json({
      success: true,
      data: result
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

### Client-Side Usage

```typescript
// Example React component
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate() {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: 'AI Marketing Automation',
        formats: ['toplist', 'how-to'],
        languages: ['en', 'vi'],
        generateVideo: true,
        platforms: ['reels']
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

  return (
    <div>
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result?.data && (
        <div>
          <h3>Generated {result.data.summary.totalArticles} articles</h3>
          <h3>Rendered {result.data.summary.totalVideos} videos</h3>
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Custom Research Sources

```typescript
// src/config/research-sources.ts
export const researchSources = {
  tech: [
    'https://techcrunch.com',
    'https://a16z.com',
    'https://www.theverge.com'
  ],
  business: [
    'https://hbr.org',
    'https://www.forbes.com',
    'https://www.entrepreneur.com'
  ],
  marketing: [
    'https://www.marketingprofs.com',
    'https://contentmarketinginstitute.com',
    'https://blog.hubspot.com'
  ]
};

// Use in crawler
import { researchSources } from '@/config/research-sources';

const techResearch = await fetchFromSources(researchSources.tech, keyword);
```

### Content Templates

```typescript
// src/config/content-templates.ts
export const contentTemplates = {
  toplist: {
    structure: ['introduction', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10,
    includeNumbers: true
  },
  'case-study': {
    structure: ['problem', 'solution', 'results', 'takeaways'],
    requireData: true,
    minWordCount: 1500
  },
  'how-to': {
    structure: ['overview', 'prerequisites', 'steps', 'tips', 'conclusion'],
    stepFormat: 'numbered',
    includeVisuals: true
  }
};
```

## Remotion Video Templates

```typescript
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface InfographicProps {
  title: string;
  content: string;
}

export const Infographic: React.FC<InfographicProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e', opacity }}>
      <div style={{ padding: 60, color: 'white' }}>
        <h1 style={{ fontSize: 80, marginBottom: 40 }}>{title}</h1>
        <p style={{ fontSize: 40, lineHeight: 1.6 }}>{content}</p>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for AI providers
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const contents = await Promise.all(
  formats.map(format => 
    limit(() => generateContent({ ...config, format }))
  )
);
```

### Video Rendering Fails

```bash
# Ensure FFmpeg is installed
ffmpeg -version

# Install if missing (macOS)
brew install ffmpeg

# Linux
sudo apt-get install ffmpeg
```

### Memory Issues with Large Content

```typescript
// Chunk processing for large batches
async function processInChunks<T>(items: T[], chunkSize: number, processor: (item: T) => Promise<any>) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
    
    // Allow garbage collection between chunks
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  
  return results;
}
```

### Environment Variables Not Loading

```typescript
// Verify env vars are loaded
import { config } from 'dotenv';
config({ path: '.env.local' });

if (!process.env.OPENAI_API_KEY) {
  throw new Error('OPENAI_API_KEY is not set');
}
```

## Production Deployment

```bash
# Build for production
yarn build

# Start production server
yarn start

# Or deploy to Vercel
vercel --prod
```

### Environment Variables in Production

Ensure these are set in your hosting platform (Vercel, Netlify, etc.):
- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`
- `RAPIDAPI_KEY`
- `DATABASE_URL` (if using database)
- `REMOTION_LICENSE_KEY` (for video rendering)

This skill provides comprehensive coverage of the Marketing Pipeline Share automation system, enabling AI coding agents to help developers implement automated content research, generation, and video production workflows.
