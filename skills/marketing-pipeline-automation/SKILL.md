---
name: marketing-pipeline-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline for automated posts
  - create automated content workflow with Claude and OpenAI
  - generate videos automatically from written content
  - build AI content pipeline for social media
  - automate research and scriptwriting for marketing
  - use Remotion to render marketing videos
  - create end-to-end content automation system
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an **Ultimate AI Content Pipeline** that automates the entire content creation workflow: from research and scriptwriting to automatic video generation and social media posting. Built with TypeScript, Next.js, and integrating Claude 3, OpenAI, and Remotion for video rendering.

## What This Project Does

The marketing pipeline automation system provides:

1. **Auto-Scan Research**: Automatically crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn
2. **AI Content Generation**: Creates diverse content formats (toplist, POV, case studies, how-tos) in multiple languages using Claude/OpenAI
3. **Automatic Video Rendering**: Converts written content into infographics and short videos using Remotion
4. **Multi-Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts, and other platforms

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
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Key Architecture Components

### 1. Research Module (Auto-Scan)

The research module crawls news sources and extracts insights:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  type: 'news' | 'social' | 'blog';
  priority: number;
}

export async function scanSources(keyword: string, hours: number = 24) {
  const sources: ResearchSource[] = [
    { url: 'https://techcrunch.com', type: 'news', priority: 1 },
    { url: 'https://a16z.com', type: 'blog', priority: 2 },
  ];

  const results = await Promise.all(
    sources.map(source => crawlSource(source, keyword, hours))
  );

  return aggregateInsights(results);
}

async function crawlSource(
  source: ResearchSource,
  keyword: string,
  hours: number
) {
  // Use RapidAPI or custom crawler
  const response = await axios.get(
    `https://api.rapidapi.com/news/search`,
    {
      params: { q: keyword, hours },
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      },
    }
  );

  return {
    source: source.url,
    articles: response.data.articles,
    insights: extractInsights(response.data),
  };
}

function extractInsights(data: any) {
  // Extract key points, statistics, quotes
  return {
    keyPoints: data.articles.map((a: any) => a.summary),
    statistics: extractStats(data.articles),
    trending: data.trendingTopics,
  };
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

export async function generateContent(config: ContentConfig) {
  const prompt = buildPrompt(config);

  // Use Claude for complex content
  if (config.format === 'case-study' || config.format === 'pov') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return {
      content: message.content[0].text,
      metadata: extractMetadata(message.content[0].text),
    };
  }

  // Use OpenAI for simpler formats
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator for marketing.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return {
    content: completion.choices[0].message.content,
    metadata: extractMetadata(completion.choices[0].message.content),
  };
}

function buildPrompt(config: ContentConfig): string {
  const { format, language, tone, researchData } = config;

  const basePrompt = `Create a ${format} article in ${language} with a ${tone} tone.
  
Research Data:
${JSON.stringify(researchData, null, 2)}

Requirements:
- Include data-backed insights
- Use specific statistics and examples
- Optimize for social media engagement
${language === 'vi' ? '- Write in Vietnamese' : '- Write in English'}
`;

  return basePrompt;
}
```

### 3. Video Generation with Remotion

Convert content to video format:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function renderContentVideo(config: VideoConfig) {
  const { content, format, duration } = config;

  // Bundle the Remotion video
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride,
  });

  // Get composition
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: parseContentForVideo(content),
      format,
    },
  });

  const composition = compositions[0];

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: parseContentForVideo(content),
      format,
    },
  });

  return {
    videoPath: outputLocation,
    duration: composition.durationInFrames / composition.fps,
  };
}

function parseContentForVideo(content: string) {
  // Extract key points, headlines, statistics for video scenes
  const lines = content.split('\n').filter(l => l.trim());
  
  return {
    title: lines[0],
    keyPoints: lines.slice(1, 6),
    cta: lines[lines.length - 1],
  };
}
```

### 4. Remotion Video Component

Create the actual video template:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: {
    title: string;
    keyPoints: string[];
    cta: string;
  };
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  content,
  format,
}) => {
  const frame = useCurrentFrame();

  const dimensions =
    format === 'reels' || format === 'tiktok'
      ? { width: 1080, height: 1920 }
      : { width: 1080, height: 1920 };

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 40,
          }}
        >
          <h1
            style={{
              color: 'white',
              fontSize: 72,
              textAlign: 'center',
              fontWeight: 'bold',
            }}
          >
            {content.title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Key Points */}
      {content.keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 40,
            }}
          >
            <div
              style={{
                color: 'white',
                fontSize: 48,
                textAlign: 'center',
                backgroundColor: 'rgba(0,0,0,0.5)',
                padding: 30,
                borderRadius: 20,
              }}
            >
              <div style={{ fontSize: 72, marginBottom: 20 }}>
                {index + 1}
              </div>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}

      {/* CTA */}
      <Sequence
        from={60 + content.keyPoints.length * 90}
        durationInFrames={60}
      >
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 40,
          }}
        >
          <h2
            style={{
              color: '#00ff00',
              fontSize: 56,
              textAlign: 'center',
            }}
          >
            {content.cta}
          </h2>
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Workflow

Orchestrate the entire pipeline:

```typescript
// lib/pipeline/orchestrator.ts
import { scanSources } from '../research/crawler';
import { generateContent } from '../ai/content-generator';
import { renderContentVideo } from '../video/renderer';

export async function runContentPipeline(keyword: string) {
  console.log(`Starting pipeline for keyword: ${keyword}`);

  // Step 1: Research
  console.log('Step 1: Scanning sources...');
  const researchData = await scanSources(keyword, 24);

  // Step 2: Generate Content (both languages)
  console.log('Step 2: Generating content...');
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData,
    }),
    generateContent({
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData,
    }),
  ]);

  // Step 3: Render Videos
  console.log('Step 3: Rendering videos...');
  const [reelsVideo, tiktokVideo] = await Promise.all([
    renderContentVideo({
      content: englishContent.content,
      format: 'reels',
      duration: 30,
    }),
    renderContentVideo({
      content: vietnameseContent.content,
      format: 'tiktok',
      duration: 30,
    }),
  ]);

  return {
    research: researchData,
    content: {
      english: englishContent,
      vietnamese: vietnameseContent,
    },
    videos: {
      reels: reelsVideo,
      tiktok: tiktokVideo,
    },
  };
}
```

## Next.js API Routes

### Start Pipeline Endpoint

```typescript
// pages/api/pipeline/start.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '../../../lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword } = req.body;

  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }

  try {
    const result = await runContentPipeline(keyword);
    return res.status(200).json(result);
  } catch (error: any) {
    console.error('Pipeline error:', error);
    return res.status(500).json({ error: error.message });
  }
}
```

## Common Usage Patterns

### Run Full Pipeline

```typescript
import { runContentPipeline } from './lib/pipeline/orchestrator';

// Execute complete workflow
const result = await runContentPipeline('AI Marketing Tools 2024');

console.log('Content generated:', result.content);
console.log('Videos rendered:', result.videos);
```

### Generate Content Only

```typescript
import { generateContent } from './lib/ai/content-generator';
import { scanSources } from './lib/research/crawler';

const research = await scanSources('startup funding', 24);
const content = await generateContent({
  format: 'case-study',
  language: 'en',
  tone: 'expert',
  researchData: research,
});
```

### Render Video from Existing Content

```typescript
import { renderContentVideo } from './lib/video/renderer';

const video = await renderContentVideo({
  content: 'Your pre-written content here...',
  format: 'shorts',
  duration: 60,
});
```

## Configuration

### Content Formats

- `toplist`: Top 10/5 style articles
- `pov`: Point of view / opinion pieces
- `case-study`: In-depth analysis
- `how-to`: Tutorial/guide format

### Video Formats

- `reels`: Instagram Reels (1080x1920)
- `tiktok`: TikTok videos (1080x1920)
- `shorts`: YouTube Shorts (1080x1920)

### Tone Options

- `expert`: Professional, authoritative
- `friendly`: Conversational, approachable
- `humorous`: Light, entertaining

## Troubleshooting

### API Rate Limits

```typescript
// Add retry logic for API calls
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      if (error.status === 429) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Failures

Ensure Remotion dependencies are properly installed:

```bash
npm install @remotion/bundler @remotion/renderer @remotion/cli
```

### Memory Issues with Large Videos

```typescript
// Optimize video rendering settings
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  chromiumOptions: {
    // Reduce memory usage
    headless: true,
    args: ['--no-sandbox', '--disable-dev-shm-usage'],
  },
  concurrency: 1, // Process one frame at a time
});
```

### Research Data Quality

Implement content filtering:

```typescript
function filterQualityContent(articles: any[]) {
  return articles.filter(article => {
    return (
      article.wordCount > 500 &&
      article.engagement > 100 &&
      !article.isSpam
    );
  });
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run tests
npm test

# Render a single video (Remotion CLI)
npx remotion render ContentVideo output.mp4

# Preview Remotion composition
npx remotion preview
```
