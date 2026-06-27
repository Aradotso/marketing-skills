---
name: ultimate-ai-content-pipeline
description: Automated content marketing pipeline for research, script generation, video creation and publishing using Claude/OpenAI
triggers:
  - how do I automate content creation with AI
  - generate marketing content from research to video
  - set up automated content pipeline with Claude
  - create videos automatically from content
  - automate social media content generation
  - build AI content workflow with Remotion
  - scrape and generate content with OpenAI
  - how to use ultimate AI content pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that handles research, content generation, and video rendering. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and creates videos using Remotion.

## What It Does

- **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for latest content
- **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos)
- **Multi-language**: Generates Vietnamese and English versions simultaneously
- **Video Generation**: Renders videos and infographics using Remotion
- **Auto-Publishing**: Integrates with social platforms for scheduling

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
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Publishing
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Video rendering
npm run render
```

## Core API Usage

### 1. Content Research Module

```typescript
import { ResearchService } from './services/research';

const researcher = new ResearchService({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch latest content on a topic
const researchData = await researcher.scanTopic({
  keyword: 'AI automation',
  timeframe: '24h',
  language: 'en'
});

console.log(researchData.insights);
// Returns: { articles: [...], trends: [...], keyPoints: [...] }
```

### 2. Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Generate a ${format} article about ${topic} based on this research data...`
    }],
  });

  return message.content[0].text;
}

// Generate different formats
const listArticle = await generateContent('AI trends', 'top 10 list');
const caseStudy = await generateContent('AI trends', 'case study');
const howTo = await generateContent('AI trends', 'how-to guide');
```

### 3. Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer. Generate engaging marketing content.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}

// Different tones
const professionalContent = await generateWithGPT(researchData, 'professional');
const casualContent = await generateWithGPT(researchData, 'friendly and casual');
```

### 4. Bilingual Content Generation

```typescript
interface ContentOutput {
  english: string;
  vietnamese: string;
  metadata: {
    format: string;
    tone: string;
    keywords: string[];
  };
}

async function generateBilingualContent(
  topic: string,
  format: 'listicle' | 'pov' | 'case-study' | 'how-to'
): Promise<ContentOutput> {
  
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(topic, format, 'en'),
    generateContent(topic, format, 'vi')
  ]);

  return {
    english: englishContent,
    vietnamese: vietnameseContent,
    metadata: {
      format,
      tone: 'professional',
      keywords: extractKeywords(topic)
    }
  };
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(contentData: any) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/video/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: contentData.title,
      points: contentData.keyPoints,
      duration: 30 // seconds
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${contentData.slug}.mp4`,
    inputProps: composition.inputProps,
  });

  return `out/${contentData.slug}.mp4`;
}
```

### 6. Complete Pipeline Workflow

```typescript
interface PipelineConfig {
  topic: string;
  format: 'listicle' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  autoPublish: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  const research = await researcher.scanTopic({
    keyword: config.topic,
    timeframe: '24h'
  });

  // Step 2: Generate content
  const content = await generateBilingualContent(
    config.topic,
    config.format
  );

  // Step 3: Generate video (optional)
  let videoPath;
  if (config.generateVideo) {
    videoPath = await generateVideo({
      title: config.topic,
      keyPoints: research.keyPoints,
      slug: slugify(config.topic)
    });
  }

  // Step 4: Publish (optional)
  if (config.autoPublish) {
    await publishContent({
      content: content.english,
      videoPath,
      platforms: ['facebook', 'twitter', 'linkedin']
    });
  }

  return {
    research,
    content,
    videoPath,
    publishStatus: config.autoPublish ? 'published' : 'draft'
  };
}

// Usage
const result = await runContentPipeline({
  topic: 'AI Marketing Automation 2026',
  format: 'listicle',
  languages: ['en', 'vi'],
  generateVideo: true,
  autoPublish: false
});
```

## Common Patterns

### Research Aggregation

```typescript
async function aggregateResearch(keywords: string[]) {
  const allResearch = await Promise.all(
    keywords.map(kw => researcher.scanTopic({ keyword: kw, timeframe: '24h' }))
  );

  return {
    combined: allResearch.flat(),
    topTrends: extractTopTrends(allResearch),
    commonThemes: findCommonThemes(allResearch)
  };
}
```

### Content Scheduling

```typescript
interface ScheduledContent {
  content: ContentOutput;
  publishAt: Date;
  platforms: string[];
}

async function scheduleContent(schedule: ScheduledContent) {
  // Store in database with publish time
  await db.scheduledPosts.create({
    data: {
      content: schedule.content.english,
      publishAt: schedule.publishAt,
      platforms: schedule.platforms,
      status: 'scheduled'
    }
  });
}
```

### Batch Processing

```typescript
async function batchGenerateContent(topics: string[]) {
  const batchSize = 3; // Process 3 at a time to avoid rate limits
  
  for (let i = 0; i < topics.length; i += batchSize) {
    const batch = topics.slice(i, i + batchSize);
    
    await Promise.all(
      batch.map(topic => runContentPipeline({
        topic,
        format: 'listicle',
        languages: ['en', 'vi'],
        generateVideo: true,
        autoPublish: false
      }))
    );
    
    // Wait between batches
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
}
```

## Video Templates (Remotion)

```typescript
// src/video/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ padding: 60, color: 'white' }}>
        <h1 style={{ fontSize: 48, opacity: Math.min(1, frame / 30) }}>
          {title}
        </h1>
        {points.map((point, idx) => (
          <p
            key={idx}
            style={{
              fontSize: 24,
              opacity: Math.max(0, Math.min(1, (frame - (idx * fps * 2)) / 30))
            }}
          >
            • {point}
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
// Implement retry logic with exponential backoff
async function apiCallWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos sequentially instead of parallel
async function renderVideosSequentially(contents: any[]) {
  const results = [];
  
  for (const content of contents) {
    const video = await generateVideo(content);
    results.push(video);
    
    // Clear memory between renders
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Claude/OpenAI Token Limits

```typescript
// Split long content into chunks
function splitIntoChunks(text: string, maxTokens = 4000): string[] {
  const chunks = [];
  const sentences = text.split('. ');
  let currentChunk = '';
  
  for (const sentence of sentences) {
    if ((currentChunk + sentence).length < maxTokens * 4) {
      currentChunk += sentence + '. ';
    } else {
      chunks.push(currentChunk);
      currentChunk = sentence + '. ';
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearch(data: any) {
  return data.filter(item => 
    item.content && 
    item.content.length > 100 &&
    item.publishedAt &&
    new Date(item.publishedAt) > new Date(Date.now() - 86400000) // 24h
  );
}
```

## Next.js API Routes

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { topic, format, languages } = req.body;

  try {
    const result = await runContentPipeline({
      topic,
      format,
      languages,
      generateVideo: true,
      autoPublish: false
    });

    res.status(200).json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

This pipeline system enables complete content automation from research through publishing, leveraging the best AI models for high-quality, multi-format, multilingual content creation.
