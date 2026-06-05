---
name: marketing-pipeline-share-automation
description: AI-powered content pipeline for automated research, scriptwriting, social posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation pipeline
  - generate video from text automatically
  - research and write marketing content with AI
  - create social media posts with AI pipeline
  - build automated content workflow
  - generate scripts and videos from keywords
  - set up AI content automation system
  - crawl news and create content automatically
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that transforms keywords into finished content across multiple formats. It automatically researches trending topics, generates scripts in multiple languages, and renders videos - creating a full content production pipeline powered by Claude 3, OpenAI, and Remotion.

**Key capabilities:**
- Auto-crawl breaking news from TechCrunch, a16z, Twitter, LinkedIn
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Bilingual output (English & Vietnamese)
- Automatic video rendering for Reels, TikTok, Shorts
- Next.js web interface for content management

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Optional: Social Media APIs
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
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

Access the application at `http://localhost:3000`

## Core Architecture

### 1. Research Module (Auto-Scan)

The research module crawls real-time data from news sources:

```typescript
// lib/research/crawler.ts
import { fetchNewsArticles } from './sources';

interface ResearchConfig {
  keyword: string;
  timeframe: '24h' | '7d' | '30d';
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
}

export async function autoResearch(config: ResearchConfig) {
  const articles = await fetchNewsArticles({
    query: config.keyword,
    timeframe: config.timeframe,
    sources: config.sources
  });
  
  // Filter and rank by relevance
  const insights = await analyzeArticles(articles);
  
  return {
    rawData: articles,
    insights: insights,
    statistics: extractStatistics(articles),
    trends: identifyTrends(articles)
  };
}
```

**Usage example:**

```typescript
const research = await autoResearch({
  keyword: 'AI marketing automation',
  timeframe: '24h',
  sources: ['techcrunch', 'twitter']
});

console.log(research.insights);
// Returns structured insights with data-backed statistics
```

### 2. Content Generation Module

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/generation/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

export async function generateContent(config: ContentConfig) {
  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return {
    content: message.content[0].text,
    metadata: {
      format: config.format,
      language: config.language,
      wordCount: message.content[0].text.split(' ').length
    }
  };
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list format with clear rankings',
    'pov': 'Write from a specific point of view with strong opinions',
    'case-study': 'Structure as: Problem → Solution → Results',
    'how-to': 'Create step-by-step instructions with actionable tips'
  };
  
  return `
You are a ${config.tone} content writer.
Format: ${config.format} - ${formatInstructions[config.format]}
Language: ${config.language === 'en' ? 'English' : 'Vietnamese'}

Research data:
${JSON.stringify(config.researchData, null, 2)}

Create engaging, data-backed content based on this research.
  `.trim();
}
```

**Usage example:**

```typescript
const content = await generateContent({
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: research.insights
});

console.log(content.content);
// Returns formatted article ready for publishing
```

### 3. Bilingual Content Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
// lib/generation/bilingual.ts
export async function generateBilingualContent(
  researchData: any,
  format: ContentConfig['format'],
  tone: ContentConfig['tone']
) {
  const [english, vietnamese] = await Promise.all([
    generateContent({
      format,
      language: 'en',
      tone,
      researchData
    }),
    generateContent({
      format,
      language: 'vi',
      tone,
      researchData
    })
  ]);
  
  return {
    english: english.content,
    vietnamese: vietnamese.content,
    metadata: {
      format,
      tone,
      generatedAt: new Date().toISOString()
    }
  };
}
```

### 4. Video Generation with Remotion

Render videos from generated content:

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface VideoProps {
  title: string;
  points: string[];
  bgColor: string;
}

export const ContentVideo: React.FC<VideoProps> = ({ title, points, bgColor }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{ fontSize: 60, fontWeight: 'bold', padding: 40 }}>
          {title}
        </div>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <div style={{ fontSize: 40, padding: 40 }}>
            {index + 1}. {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

**Render video programmatically:**

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

export async function renderContentVideo(content: {
  title: string;
  points: string[];
}) {
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.points,
      bgColor: '#1a1a2e'
    },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.title.replace(/\s+/g, '-')}.mp4`,
  });
  
  return {
    videoPath: `out/${content.title.replace(/\s+/g, '-')}.mp4`,
    duration: composition.durationInFrames / composition.fps
  };
}
```

### 5. Complete Pipeline Workflow

Chain all modules together:

```typescript
// lib/pipeline/workflow.ts
export async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching...');
  const research = await autoResearch({
    keyword,
    timeframe: '24h',
    sources: ['techcrunch', 'twitter']
  });
  
  // Step 2: Generate bilingual content
  console.log('✍️ Generating content...');
  const content = await generateBilingualContent(
    research.insights,
    'toplist',
    'expert'
  );
  
  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(content.english);
  
  // Step 4: Render video
  console.log('🎬 Rendering video...');
  const video = await renderContentVideo({
    title: keyword,
    points: keyPoints
  });
  
  return {
    research,
    content,
    video,
    summary: {
      keyword,
      formats: ['article-en', 'article-vi', 'video'],
      completedAt: new Date().toISOString()
    }
  };
}

function extractKeyPoints(content: string): string[] {
  // Extract numbered points or sentences
  const matches = content.match(/^\d+\.\s+(.+)$/gm);
  return matches ? matches.slice(0, 5) : content.split('. ').slice(0, 5);
}
```

**Full pipeline execution:**

```typescript
// Usage in API route or CLI
const result = await runContentPipeline('AI content automation');

console.log('English Article:', result.content.english);
console.log('Vietnamese Article:', result.content.vietnamese);
console.log('Video Path:', result.video.videoPath);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { autoResearch } from '@/lib/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, timeframe, sources } = req.body;
  
  try {
    const research = await autoResearch({
      keyword,
      timeframe,
      sources
    });
    
    res.status(200).json(research);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { format, language, tone, researchData } = req.body;
  
  try {
    const content = await generateContent({
      format,
      language,
      tone,
      researchData
    });
    
    res.status(200).json(content);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Complete Pipeline Endpoint

```typescript
// pages/api/pipeline.ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword } = req.body;
  
  try {
    const result = await runContentPipeline(keyword);
    res.status(200).json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI marketing', 'content automation', 'video generation'];
  
  for (const keyword of keywords) {
    try {
      await runContentPipeline(keyword);
      console.log(`✅ Completed pipeline for: ${keyword}`);
    } catch (error) {
      console.error(`❌ Failed for ${keyword}:`, error);
    }
  }
});
```

### Pattern 2: Content Validation & Quality Check

```typescript
// lib/validation/quality.ts
interface QualityMetrics {
  readabilityScore: number;
  keywordDensity: number;
  sentimentScore: number;
  dataPointsCount: number;
}

export function validateContent(content: string): QualityMetrics {
  const words = content.split(/\s+/);
  const sentences = content.split(/[.!?]+/);
  
  return {
    readabilityScore: calculateReadability(sentences, words),
    keywordDensity: calculateKeywordDensity(content),
    sentimentScore: analyzeSentiment(content),
    dataPointsCount: (content.match(/\d+%|\$\d+|#\d+/g) || []).length
  };
}

// Use in pipeline
const content = await generateContent(config);
const quality = validateContent(content.content);

if (quality.readabilityScore < 60) {
  console.warn('Content may be too complex');
}
```

### Pattern 3: Multi-Platform Video Export

```typescript
// lib/video/export.ts
interface PlatformConfig {
  width: number;
  height: number;
  fps: number;
  duration: number;
}

const platforms: Record<string, PlatformConfig> = {
  'tiktok': { width: 1080, height: 1920, fps: 30, duration: 60 },
  'reels': { width: 1080, height: 1920, fps: 30, duration: 60 },
  'youtube-shorts': { width: 1080, height: 1920, fps: 30, duration: 60 },
  'twitter': { width: 1280, height: 720, fps: 30, duration: 140 }
};

export async function exportForPlatforms(
  content: { title: string; points: string[] },
  targetPlatforms: string[]
) {
  const exports = await Promise.all(
    targetPlatforms.map(platform =>
      renderMedia({
        composition: {
          ...platforms[platform],
          id: 'ContentVideo'
        },
        inputProps: content,
        outputLocation: `out/${platform}/${content.title}.mp4`
      })
    )
  );
  
  return exports;
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pRetry from 'p-retry';

export async function withRetry<T>(
  fn: () => Promise<T>,
  options = { retries: 3 }
): Promise<T> {
  return pRetry(fn, {
    ...options,
    onFailedAttempt: error => {
      console.log(
        `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
      );
    }
  });
}

// Usage
const content = await withRetry(() =>
  generateContent(config)
);
```

### Issue: Video Rendering Fails

Check Remotion bundle and composition:

```bash
# Test composition
npx remotion compositions remotion/index.ts

# Preview before rendering
npx remotion preview remotion/index.ts
```

### Issue: Missing Research Data

Add fallback for API failures:

```typescript
export async function autoResearch(config: ResearchConfig) {
  try {
    const articles = await fetchNewsArticles(config);
    return analyzeArticles(articles);
  } catch (error) {
    console.warn('Research API failed, using fallback');
    return getFallbackResearch(config.keyword);
  }
}

function getFallbackResearch(keyword: string) {
  return {
    insights: [`General insights about ${keyword}`],
    statistics: {},
    trends: []
  };
}
```

### Issue: Content Quality Issues

Implement iterative refinement:

```typescript
async function generateHighQualityContent(config: ContentConfig) {
  let attempts = 0;
  const maxAttempts = 3;
  
  while (attempts < maxAttempts) {
    const content = await generateContent(config);
    const quality = validateContent(content.content);
    
    if (quality.readabilityScore >= 70 && quality.dataPointsCount >= 3) {
      return content;
    }
    
    attempts++;
    console.log(`Quality check failed, retry ${attempts}/${maxAttempts}`);
  }
  
  throw new Error('Could not generate high-quality content');
}
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use streaming** for long-form content generation
3. **Validate content quality** before video rendering
4. **Store generated assets** in CDN for faster delivery
5. **Monitor AI costs** with usage tracking
6. **Version control prompts** for reproducibility
7. **A/B test different formats** to find best-performing content
