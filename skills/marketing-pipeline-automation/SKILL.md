---
name: marketing-pipeline-automation
description: Automated AI-powered content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content generation pipeline"
  - "create AI-powered marketing content workflow"
  - "automate content from research to video"
  - "generate content with Claude and OpenAI"
  - "build automated marketing pipeline"
  - "create content automation system"
  - "set up AI content research and generation"
  - "automate video content creation with Remotion"
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates content creation from initial research through video generation. The pipeline crawls news sources, generates articles in multiple formats and languages, and renders videos automatically.

## What It Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Scan Research**: Crawls recent content from TechCrunch, a16z, X/Twitter, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion
REMOTION_LICENSE_KEY=your_remotion_license
```

## Key Components

### 1. Research/Crawling Module

The system crawls news sources to gather fresh data:

```typescript
// Example: Triggering auto-research
interface ResearchConfig {
  sources: string[];
  timeframe: string; // e.g., "24h", "7d"
  keywords: string[];
  language?: string;
}

async function startResearch(config: ResearchConfig) {
  const { sources, timeframe, keywords, language = 'en' } = config;
  
  // Crawl sources
  const results = await Promise.all(
    sources.map(source => 
      crawlSource({ source, timeframe, keywords })
    )
  );
  
  // Extract insights
  const insights = await extractInsights(results);
  
  return {
    rawData: results,
    insights,
    timestamp: new Date().toISOString()
  };
}

// Usage
const research = await startResearch({
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h',
  keywords: ['AI', 'marketing', 'automation']
});
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI:

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Claude Integration
async function generateWithClaude(
  prompt: string, 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Generate a ${format} article about: ${prompt}
      
      Requirements:
      - Data-backed insights
      - Engaging tone
      - SEO optimized
      - Include statistics and examples`
    }]
  });

  return message.content[0].text;
}

// OpenAI Integration
async function generateWithOpenAI(
  prompt: string,
  language: 'en' | 'vi' = 'en'
) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creation expert. Generate in ${language}.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7
  });

  return completion.choices[0].message.content;
}

// Usage Example
const article = await generateWithClaude(
  'AI automation in marketing',
  'how-to'
);
```

### 3. Multi-Format Content Generation

```typescript
interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  includeVideo: boolean;
}

async function generateContent(request: ContentRequest) {
  const { topic, format, tone, languages, includeVideo } = request;
  
  // Step 1: Research phase
  const research = await startResearch({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    keywords: [topic]
  });
  
  // Step 2: Generate content for each language
  const content = await Promise.all(
    languages.map(async (lang) => {
      const prompt = buildPrompt({
        topic,
        format,
        tone,
        language: lang,
        research: research.insights
      });
      
      const article = await generateWithClaude(prompt, format);
      
      return {
        language: lang,
        content: article,
        metadata: {
          format,
          tone,
          wordCount: article.split(' ').length
        }
      };
    })
  );
  
  // Step 3: Generate video if requested
  let video = null;
  if (includeVideo) {
    video = await renderVideo(content[0].content);
  }
  
  return {
    articles: content,
    video,
    research: research.insights
  };
}

function buildPrompt(params: {
  topic: string;
  format: string;
  tone: string;
  language: string;
  research: any;
}): string {
  const { topic, format, tone, language, research } = params;
  
  return `Create a ${format} article about "${topic}" in ${language}.

Tone: ${tone}
Research Data: ${JSON.stringify(research)}

Structure:
${getFormatStructure(format)}

Include:
- Latest statistics and data points
- Real-world examples
- Actionable insights
- SEO-optimized headings`;
}

function getFormatStructure(format: string): string {
  const structures = {
    'toplist': '1. Introduction\n2. Top items (numbered list)\n3. Conclusion',
    'pov': '1. Hook\n2. Personal perspective\n3. Supporting evidence\n4. Call to action',
    'case-study': '1. Challenge\n2. Solution\n3. Implementation\n4. Results',
    'how-to': '1. Introduction\n2. Step-by-step guide\n3. Tips & best practices\n4. Conclusion'
  };
  
  return structures[format] || structures['how-to'];
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { join } from 'path';

interface VideoConfig {
  content: string;
  platform: 'reels' | 'tiktok' | 'shorts';
  duration?: number;
}

async function renderVideo(config: VideoConfig) {
  const { content, platform, duration = 30 } = config;
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const { width, height } = dimensions[platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: parseContentForVideo(content),
      width,
      height
    }
  });
  
  // Render video
  const outputPath = join(
    process.cwd(),
    `output/video-${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: parseContentForVideo(content)
    }
  });
  
  return {
    path: outputPath,
    platform,
    dimensions: { width, height }
  };
}

function parseContentForVideo(content: string) {
  // Extract key points from content
  const lines = content.split('\n');
  const keyPoints = lines
    .filter(line => line.match(/^#{1,3}\s/) || line.match(/^\d+\./))
    .slice(0, 5);
  
  return {
    title: keyPoints[0] || 'Content Video',
    points: keyPoints.slice(1),
    duration: 30
  };
}
```

### 5. Complete Pipeline Execution

```typescript
// Full pipeline from keyword to published content
async function runContentPipeline(keyword: string) {
  console.log(`Starting pipeline for keyword: ${keyword}`);
  
  try {
    // Step 1: Research
    console.log('Phase 1: Auto-Research...');
    const research = await startResearch({
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeframe: '24h',
      keywords: [keyword]
    });
    
    // Step 2: Generate content
    console.log('Phase 2: Content Generation...');
    const contentResult = await generateContent({
      topic: keyword,
      format: 'how-to',
      tone: 'professional',
      languages: ['en', 'vi'],
      includeVideo: true
    });
    
    // Step 3: Render videos for each platform
    console.log('Phase 3: Video Rendering...');
    const videos = await Promise.all(
      ['reels', 'tiktok', 'shorts'].map(platform =>
        renderVideo({
          content: contentResult.articles[0].content,
          platform: platform as 'reels' | 'tiktok' | 'shorts'
        })
      )
    );
    
    // Step 4: Prepare for publishing
    console.log('Phase 4: Preparing output...');
    const output = {
      keyword,
      timestamp: new Date().toISOString(),
      research: research.insights,
      articles: contentResult.articles,
      videos: videos,
      status: 'ready_to_publish'
    };
    
    // Save to database or file system
    await saveOutput(output);
    
    console.log('Pipeline completed successfully!');
    return output;
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

async function saveOutput(output: any) {
  // Save to database or file system
  const fs = require('fs').promises;
  const path = join(process.cwd(), 'output', `${output.keyword}-${Date.now()}.json`);
  
  await fs.writeFile(path, JSON.stringify(output, null, 2));
  console.log(`Output saved to: ${path}`);
}

// Usage
const result = await runContentPipeline('AI marketing automation');
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

  const { keyword, format, languages, includeVideo } = req.body;

  try {
    const result = await generateContent({
      topic: keyword,
      format,
      tone: 'professional',
      languages,
      includeVideo
    });

    res.status(200).json(result);
  } catch (error) {
    console.error('Generation error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run specific pipeline
npm run pipeline -- --keyword "AI automation"
```

## Common Patterns

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Schedule daily content generation at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'marketing automation', 'content strategy'];
  
  for (const keyword of keywords) {
    await runContentPipeline(keyword);
  }
});
```

### Batch Processing

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline(keyword);
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Format Templates

```typescript
interface CustomFormat {
  name: string;
  structure: string[];
  requirements: string[];
}

function createCustomFormat(format: CustomFormat): string {
  return `
Format: ${format.name}

Structure:
${format.structure.map((s, i) => `${i + 1}. ${s}`).join('\n')}

Requirements:
${format.requirements.map(r => `- ${r}`).join('\n')}
`;
}
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
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### Video Rendering Issues

```typescript
// Ensure proper Remotion configuration
export const RemotionConfig = {
  codec: 'h264',
  crf: 20, // Quality (lower = better)
  pixelFormat: 'yuv420p',
  numberOfGifLoops: 0,
  everyNthFrame: 1,
  verbose: true
};
```

### Content Quality Validation

```typescript
function validateContent(content: string): boolean {
  const minWordCount = 300;
  const words = content.split(/\s+/).length;
  
  if (words < minWordCount) {
    throw new Error(`Content too short: ${words} words (min: ${minWordCount})`);
  }
  
  // Check for meaningful headings
  const headings = content.match(/^#{1,3}\s.+$/gm);
  if (!headings || headings.length < 3) {
    throw new Error('Insufficient content structure');
  }
  
  return true;
}
```

## Best Practices

1. **Always validate environment variables** before running the pipeline
2. **Implement error handling** at each pipeline stage
3. **Use rate limiting** to respect API quotas
4. **Cache research data** to avoid redundant crawling
5. **Monitor token usage** for AI APIs
6. **Test video renders** with small samples before batch processing
7. **Version control your prompts** for consistent output quality
