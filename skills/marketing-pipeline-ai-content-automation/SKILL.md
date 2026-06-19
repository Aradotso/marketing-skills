---
name: marketing-pipeline-ai-content-automation
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - generate video content from research automatically
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - build content from research to video generation
  - automate blog posts and video creation with AI
  - use Remotion to generate marketing videos
  - crawl news and create content automatically
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating multi-format articles, to rendering videos automatically using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

The Marketing Pipeline automates:
- **Auto-Research**: Crawls and analyzes data from TechCrunch, a16z, Twitter/X, LinkedIn for trending topics
- **AI Content Generation**: Creates diverse content formats (toplists, POV articles, case studies, how-tos) in multiple languages
- **Video Rendering**: Automatically converts written content into infographics and short-form videos optimized for Reels, TikTok, Shorts
- **Multi-Platform Export**: Delivers ready-to-publish content across social media platforms

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

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# RapidAPI for content scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion (if using cloud rendering)
REMOTION_LICENSE_KEY=your_remotion_license_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video preview
npm run remotion:preview

# Render video
npm run remotion:render
```

## Core Architecture

### 1. Research Module (Content Scraping)

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface ScrapedArticle {
  title: string;
  url: string;
  publishedAt: string;
  summary: string;
  source: string;
}

export async function scrapeNewsSource(
  keyword: string,
  source: 'techcrunch' | 'twitter' | 'linkedin'
): Promise<ScrapedArticle[]> {
  const rapidAPIHost = process.env.RAPIDAPI_HOST;
  const rapidAPIKey = process.env.RAPIDAPI_KEY;

  try {
    const response = await axios.get(
      `https://${rapidAPIHost}/search`,
      {
        params: { q: keyword, source },
        headers: {
          'X-RapidAPI-Key': rapidAPIKey,
          'X-RapidAPI-Host': rapidAPIHost,
        },
      }
    );

    return response.data.articles.map((article: any) => ({
      title: article.title,
      url: article.url,
      publishedAt: article.publishedAt,
      summary: article.summary,
      source: article.source,
    }));
  } catch (error) {
    console.error('Scraping error:', error);
    return [];
  }
}

export async function aggregateResearch(keyword: string) {
  const sources = ['techcrunch', 'twitter', 'linkedin'] as const;
  
  const results = await Promise.all(
    sources.map(source => scrapeNewsSource(keyword, source))
  );

  return results.flat();
}
```

### 2. AI Content Generation

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

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  research: string[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
}

export async function generateContentClaude(
  request: ContentRequest
): Promise<string> {
  const systemPrompt = buildSystemPrompt(request.format, request.tone);
  const userPrompt = buildUserPrompt(request.keyword, request.research, request.language);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

export async function generateContentOpenAI(
  request: ContentRequest
): Promise<string> {
  const systemPrompt = buildSystemPrompt(request.format, request.tone);
  const userPrompt = buildUserPrompt(request.keyword, request.research, request.language);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}

function buildSystemPrompt(format: ContentFormat, tone: Tone): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write a point-of-view opinion piece with strong perspectives',
    'case-study': 'Develop a detailed case study with data and analysis',
    'how-to': 'Create a step-by-step tutorial with actionable instructions',
  };

  const toneInstructions = {
    'expert': 'Use authoritative, data-driven language with industry terminology',
    'friendly': 'Write in a conversational, approachable style',
    'humorous': 'Inject wit and humor while maintaining professionalism',
  };

  return `You are an expert content writer. ${formatInstructions[format]}. ${toneInstructions[tone]}. Use the provided research data to create accurate, up-to-date content.`;
}

function buildUserPrompt(keyword: string, research: string[], language: Language): string {
  const langInstruction = language === 'vi' 
    ? 'Write in Vietnamese' 
    : 'Write in English';

  return `
${langInstruction}

Topic: ${keyword}

Research data:
${research.join('\n\n')}

Create comprehensive content based on this research. Include:
- Engaging headline
- Introduction hook
- Main content with subheadings
- Data points and statistics from research
- Actionable insights
- Strong conclusion
`;
}
```

### 3. Video Generation with Remotion

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface VideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<VideoProps> = ({ title, points, brandColor }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1 style={{ color: brandColor, fontSize: 80, textAlign: 'center', padding: 40 }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 2)}
          durationInFrames={fps * 2}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
            }}
          >
            <div style={{ color: '#fff', fontSize: 48, textAlign: 'center' }}>
              <div style={{ color: brandColor, fontSize: 72, marginBottom: 20 }}>
                {index + 1}
              </div>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  title: string,
  points: string[],
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: { title, points, brandColor: '#FF6B6B' },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: { title, points, brandColor: '#FF6B6B' },
  });

  return outputPath;
}
```

### 4. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { aggregateResearch } from '../research/scraper';
import { generateContentClaude, ContentFormat, Language, Tone } from '../ai/content-generator';
import { renderContentVideo } from '../video/renderer';
import path from 'path';

export interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  generateVideo: boolean;
}

export interface PipelineResult {
  content: string;
  videoPath?: string;
  researchSources: number;
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  // Step 1: Research
  console.log(`[Pipeline] Researching: ${config.keyword}`);
  const researchData = await aggregateResearch(config.keyword);
  const researchSummaries = researchData.map(
    article => `${article.title}\n${article.summary}\nSource: ${article.source}`
  );

  // Step 2: Generate Content
  console.log(`[Pipeline] Generating ${config.format} content in ${config.language}`);
  const content = await generateContentClaude({
    keyword: config.keyword,
    research: researchSummaries,
    format: config.format,
    language: config.language,
    tone: config.tone,
  });

  const result: PipelineResult = {
    content,
    researchSources: researchData.length,
  };

  // Step 3: Generate Video (optional)
  if (config.generateVideo && config.format === 'toplist') {
    console.log('[Pipeline] Rendering video');
    const points = extractTopListPoints(content);
    const videoPath = path.join(
      process.cwd(),
      'public/videos',
      `${config.keyword.replace(/\s+/g, '-')}-${Date.now()}.mp4`
    );

    result.videoPath = await renderContentVideo(
      config.keyword,
      points,
      videoPath
    );
  }

  return result;
}

function extractTopListPoints(content: string): string[] {
  // Extract numbered points from content
  const matches = content.match(/\d+\.\s+(.+?)(?=\n\d+\.|\n\n|$)/gs);
  return matches 
    ? matches.map(m => m.replace(/^\d+\.\s+/, '').trim()).slice(0, 5)
    : [];
}
```

### 5. API Route Example (Next.js)

```typescript
// pages/api/generate-content.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline, PipelineConfig } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, language, tone, generateVideo } = req.body;

  if (!keyword || !format || !language || !tone) {
    return res.status(400).json({ error: 'Missing required parameters' });
  }

  try {
    const config: PipelineConfig = {
      keyword,
      format,
      language,
      tone,
      generateVideo: generateVideo ?? false,
    };

    const result = await runContentPipeline(config);

    return res.status(200).json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return res.status(500).json({ 
      error: 'Content generation failed',
      message: error instanceof Error ? error.message : 'Unknown error',
    });
  }
}
```

## Common Usage Patterns

### Pattern 1: Generate Toplist Article with Video

```typescript
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

const result = await runContentPipeline({
  keyword: 'Top AI Tools 2026',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  generateVideo: true,
});

console.log('Article:', result.content);
console.log('Video path:', result.videoPath);
```

### Pattern 2: Multi-Language Content Generation

```typescript
const languages = ['en', 'vi'] as const;

const results = await Promise.all(
  languages.map(lang =>
    runContentPipeline({
      keyword: 'Marketing Automation Trends',
      format: 'pov',
      language: lang,
      tone: 'friendly',
      generateVideo: false,
    })
  )
);

const [englishContent, vietnameseContent] = results;
```

### Pattern 3: Batch Processing Multiple Keywords

```typescript
const keywords = [
  'AI Content Creation',
  'Social Media Automation',
  'Video Marketing Tips',
];

for (const keyword of keywords) {
  const result = await runContentPipeline({
    keyword,
    format: 'how-to',
    language: 'en',
    tone: 'friendly',
    generateVideo: true,
  });

  // Save to database or export
  await saveContent(keyword, result);
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting:

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function batchGenerate(keywords: string[]) {
  return Promise.all(
    keywords.map(keyword =>
      limit(() => runContentPipeline({ /* config */ }))
    )
  );
}
```

### Video Rendering Errors

If Remotion fails to render:

```bash
# Ensure ffmpeg is installed
brew install ffmpeg  # macOS
apt-get install ffmpeg  # Ubuntu

# Check Remotion license
echo $REMOTION_LICENSE_KEY
```

### Memory Issues with Large Content

```typescript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev"
  }
}
```

### Claude API Timeout

```typescript
const message = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  timeout: 60000, // 60 seconds
  messages: [/* ... */],
});
```

## Best Practices

1. **Caching Research Data**: Cache scraped articles to avoid redundant API calls
2. **Content Validation**: Validate AI-generated content before publishing
3. **Video Optimization**: Compress videos for faster uploads
4. **Error Handling**: Always wrap API calls in try-catch blocks
5. **Environment Variables**: Never commit API keys; use `.env.local`

This skill provides comprehensive automation for content marketing workflows, enabling rapid creation of research-backed, multi-format content with automated video generation.
