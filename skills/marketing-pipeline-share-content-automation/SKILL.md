---
name: marketing-pipeline-share-content-automation
description: Automated AI content pipeline that researches, generates scripts, and creates videos using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an automated marketing content pipeline
  - generate videos from text automatically
  - create content from research to video
  - build an AI content generation system
  - automate social media content workflow
  - use Remotion for video generation from posts
  - crawl news and generate content automatically
---

# Marketing Pipeline Share Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI-powered content automation system that transforms a single keyword into fully-rendered content across multiple formats. It automatically:

1. **Researches** - Crawls recent news from TechCrunch, a16z, Twitter, LinkedIn
2. **Generates** - Creates articles in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 or OpenAI
3. **Renders** - Produces videos and infographics using Remotion
4. **Publishes** - Outputs content ready for multi-platform distribution

The system supports bilingual content (English/Vietnamese) with customizable tone and voice.

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

Create a `.env` file in the project root:

```env
# AI Providers
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Run Remotion video previews
npm run remotion:preview

# Render videos
npm run remotion:render
```

## Core Components

### 1. Research Module

The research module crawls and aggregates content from multiple sources:

```typescript
// src/services/research.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
}

export async function crawlTechNews(keyword: string): Promise<ResearchResult[]> {
  const sources = [
    { name: 'TechCrunch', endpoint: '/techcrunch/search' },
    { name: 'a16z', endpoint: '/a16z/posts' },
  ];

  const results: ResearchResult[] = [];

  for (const source of sources) {
    try {
      const response = await axios.get(`${process.env.RAPIDAPI_ENDPOINT}${source.endpoint}`, {
        params: { q: keyword, days: 1 },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
      });

      results.push(...response.data.map((item: any) => ({
        title: item.title,
        url: item.url,
        summary: item.excerpt || item.description,
        publishedAt: new Date(item.published_date),
        source: source.name,
      })));
    } catch (error) {
      console.error(`Error fetching from ${source.name}:`, error);
    }
  }

  return results;
}
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI based on research data:

```typescript
// src/services/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  research: ResearchResult[];
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
}

export async function generateWithClaude(request: ContentRequest): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const researchContext = request.research
    .map((r) => `Source: ${r.source}\nTitle: ${r.title}\nSummary: ${r.summary}`)
    .join('\n\n');

  const prompt = `You are a professional content writer. Based on the following research about "${request.keyword}", create a ${request.format} article in ${request.language} with a ${request.tone} tone.

Research:
${researchContext}

Requirements:
- Length: 800-1200 words
- Include data and statistics from the research
- Format: ${request.format}
- Language: ${request.language}
- Tone: ${request.tone}
- Include relevant headings and structure`;

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

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

export async function generateWithOpenAI(request: ContentRequest): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const researchContext = request.research
    .map((r) => `Source: ${r.source}\nTitle: ${r.title}\nSummary: ${r.summary}`)
    .join('\n\n');

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content writer creating ${request.format} articles in ${request.language} with a ${request.tone} tone.`,
      },
      {
        role: 'user',
        content: `Create an article about "${request.keyword}" based on this research:\n\n${researchContext}`,
      },
    ],
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 3. Video Generation with Remotion

Create videos from generated content:

```typescript
// src/remotion/VideoComposition.tsx
import { Composition } from 'remotion';
import { VideoTemplate } from './templates/VideoTemplate';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={VideoTemplate}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Title Here',
          content: [],
          theme: 'dark',
        }}
      />
    </>
  );
};
```

```typescript
// src/remotion/templates/VideoTemplate.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface VideoProps {
  title: string;
  content: string[];
  theme: 'dark' | 'light';
}

export const VideoTemplate: React.FC<VideoProps> = ({ title, content, theme }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: theme === 'dark' ? '#000' : '#fff' }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{ 
          display: 'flex', 
          justifyContent: 'center', 
          alignItems: 'center',
          fontSize: 60,
          color: theme === 'dark' ? '#fff' : '#000',
          padding: 40,
        }}>
          {title}
        </div>
      </Sequence>
      
      {content.map((text, index) => (
        <Sequence key={index} from={60 + index * 60} durationInFrames={60}>
          <div style={{
            display: 'flex',
            justifyContent: 'center',
            alignItems: 'center',
            fontSize: 40,
            color: theme === 'dark' ? '#fff' : '#000',
            padding: 40,
          }}>
            {text}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 4. Full Pipeline Execution

Orchestrate the entire content creation pipeline:

```typescript
// src/services/pipeline.ts
import { crawlTechNews } from './research';
import { generateWithClaude } from './content-generator';
import { renderVideo } from './video-renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const research = await crawlTechNews(config.keyword);
  console.log(`Found ${research.length} sources`);

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await generateWithClaude({
    keyword: config.keyword,
    research,
    format: config.format,
    language: config.language,
    tone: config.tone,
  });

  // Step 3: Extract video segments (optional)
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    const videoSegments = extractKeyPoints(content);
    const videoPath = await renderVideo({
      title: config.keyword,
      segments: videoSegments,
      outputPath: `./output/${config.keyword}-${Date.now()}.mp4`,
    });
    console.log(`Video saved to: ${videoPath}`);
  }

  return {
    content,
    research,
    metadata: {
      keyword: config.keyword,
      format: config.format,
      language: config.language,
      createdAt: new Date(),
    },
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - split by paragraphs and take first sentence of each
  return content
    .split('\n\n')
    .filter((p) => p.trim().length > 0)
    .map((p) => p.split('.')[0] + '.')
    .slice(0, 5);
}
```

### 5. API Routes (Next.js)

```typescript
// src/pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/services/pipeline';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, format, language, tone, generateVideo } = req.body;

    if (!keyword) {
      return res.status(400).json({ error: 'Keyword is required' });
    }

    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      tone: tone || 'expert',
      generateVideo: generateVideo || false,
    });

    return res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return res.status(500).json({ 
      error: 'Content generation failed',
      details: error instanceof Error ? error.message : 'Unknown error',
    });
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { runContentPipeline } from '../src/services/pipeline';

const keywords = [
  'artificial intelligence trends',
  'blockchain technology',
  'sustainable energy',
];

async function batchGenerate() {
  for (const keyword of keywords) {
    console.log(`\nProcessing: ${keyword}`);
    
    try {
      await runContentPipeline({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        generateVideo: true,
      });
      
      // Wait between requests to avoid rate limits
      await new Promise((resolve) => setTimeout(resolve, 5000));
    } catch (error) {
      console.error(`Failed for ${keyword}:`, error);
    }
  }
}

batchGenerate();
```

### Custom Content Templates

```typescript
// src/templates/custom-template.ts
export interface ContentTemplate {
  name: string;
  structure: string[];
  systemPrompt: string;
}

export const templates: Record<string, ContentTemplate> = {
  'startup-analysis': {
    name: 'Startup Analysis',
    structure: ['Executive Summary', 'Market Opportunity', 'Team', 'Traction', 'Risks', 'Verdict'],
    systemPrompt: 'You are a venture capital analyst writing detailed startup assessments.',
  },
  'product-review': {
    name: 'Product Review',
    structure: ['Overview', 'Features', 'Pros', 'Cons', 'Pricing', 'Alternatives', 'Verdict'],
    systemPrompt: 'You are a tech reviewer providing honest, detailed product assessments.',
  },
};
```

## Troubleshooting

### API Rate Limits

```typescript
// src/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delayMs: number;

  constructor(requestsPerMinute: number) {
    this.delayMs = 60000 / requestsPerMinute;
  }

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;

    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) {
        await fn();
        await new Promise((resolve) => setTimeout(resolve, this.delayMs));
      }
    }

    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute

await limiter.add(() => generateWithClaude(request));
```

### Video Rendering Errors

```bash
# Install required system dependencies for Remotion
# On Ubuntu/Debian:
sudo apt-get update
sudo apt-get install -y chromium-browser ffmpeg

# On macOS:
brew install chromium ffmpeg

# Set environment variable for Chromium path
export PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
```

### Memory Issues with Large Content

```typescript
// Implement streaming for large content generation
import { OpenAIStream } from 'ai';

export async function generateContentStream(request: ContentRequest) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [/* your messages */],
    stream: true,
  });

  const stream = OpenAIStream(response);
  return stream;
}
```

## Best Practices

1. **Always validate API keys** before running pipelines
2. **Cache research results** to avoid redundant API calls
3. **Use environment-specific configs** for different stages
4. **Implement retry logic** for API failures
5. **Monitor token usage** to control costs
6. **Store generated content** in a database for reuse
7. **Use queues** for long-running video rendering tasks

## CLI Commands

```bash
# Generate content from keyword
npm run generate -- --keyword "AI trends" --format toplist --lang en

# Render video from existing content
npm run render:video -- --input ./content/article.json --output ./videos/

# Batch process from CSV
npm run batch -- --input keywords.csv --config batch-config.json

# Preview Remotion compositions
npm run remotion:preview

# Render specific composition
npx remotion render ContentVideo output.mp4 --props='{"title":"Test"}'
```
