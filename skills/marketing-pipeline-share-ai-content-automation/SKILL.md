---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate automated marketing videos from text
  - create content workflow with research and video rendering
  - build automated content generation system
  - use Remotion to render videos from AI content
  - automate social media content creation pipeline
  - research to video content automation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, an automated content creation system that handles the entire workflow from research, script generation, to video rendering using AI (Claude 3, OpenAI) and Remotion.

## What It Does

The marketing-pipeline-share project provides a complete content automation pipeline:

1. **Auto-Research**: Crawls and analyzes data from news sources (TechCrunch, a16z, X/Twitter, LinkedIn)
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates English and Vietnamese versions
4. **Video Rendering**: Automatically renders videos and infographics using Remotion
5. **Platform Optimization**: Exports video formats for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Configuration

Create a `.env.local` file in the project root:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

Reference environment variables in code:

```typescript
const anthropicApiKey = process.env.ANTHROPIC_API_KEY;
const openaiApiKey = process.env.OPENAI_API_KEY;
const rapidApiKey = process.env.RAPIDAPI_KEY;
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview (for video editing)
npm run remotion:preview

# Render video
npm run remotion:render
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
// lib/research/crawler.ts
import { searchNews } from './sources/techcrunch';
import { searchTwitter } from './sources/twitter';

interface ResearchData {
  keyword: string;
  sources: Array<{
    title: string;
    url: string;
    content: string;
    publishedAt: string;
  }>;
}

async function conductResearch(keyword: string): Promise<ResearchData> {
  const [newsData, socialData] = await Promise.all([
    searchNews(keyword, process.env.RAPIDAPI_KEY),
    searchTwitter(keyword, process.env.RAPIDAPI_KEY)
  ]);

  return {
    keyword,
    sources: [...newsData, ...socialData]
  };
}

// Usage
const research = await conductResearch('AI marketing automation');
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchData;
}

async function generateContent(config: ContentConfig): Promise<string> {
  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with at least 5 items',
    'pov': 'Write from a personal perspective with unique insights',
    'case-study': 'Analyze a specific case with data and outcomes',
    'how-to': 'Provide step-by-step instructions'
  };

  return `
You are a ${config.tone} content writer.
Format: ${formatInstructions[config.format]}
Language: ${config.language === 'vi' ? 'Vietnamese' : 'English'}

Research data:
${JSON.stringify(config.researchData, null, 2)}

Generate comprehensive content based on this research.
`;
}
```

### 3. AI Content Generation with OpenAI

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentOpenAI(config: ContentConfig): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${config.tone} content writer specializing in ${config.format} format.`
      },
      {
        role: 'user',
        content: buildPrompt(config)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
}

async function renderContentVideo(config: VideoConfig): Promise<string> {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format
    },
  });

  // Render video
  const outputLocation = path.resolve(`./public/videos/${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format
    },
  });

  return outputLocation;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Sequence } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  content, 
  format 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  const contentLines = content.split('\n').filter(line => line.trim());

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill 
          style={{ 
            justifyContent: 'center', 
            alignItems: 'center',
            opacity 
          }}
        >
          <h1 style={{ 
            color: 'white', 
            fontSize: 60,
            textAlign: 'center',
            padding: 40
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {contentLines.map((line, index) => (
        <Sequence 
          key={index} 
          from={60 + (index * 90)} 
          durationInFrames={90}
        >
          <AbsoluteFill 
            style={{ 
              justifyContent: 'center', 
              alignItems: 'center',
              padding: 40
            }}
          >
            <p style={{ 
              color: 'white', 
              fontSize: 40,
              textAlign: 'center',
              lineHeight: 1.5
            }}>
              {line}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { conductResearch } from '../research/crawler';
import { generateContent } from '../ai/claude-generator';
import { renderContentVideo } from '../video/remotion-renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

interface PipelineResult {
  content: string;
  videoPath?: string;
  research: ResearchData;
}

async function runContentPipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await conductResearch(config.keyword);

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent({
    format: config.format,
    language: config.language,
    tone: config.tone,
    researchData: research
  });

  // Step 3: Render Video (optional)
  let videoPath: string | undefined;
  if (config.generateVideo && config.videoFormat) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo({
      content,
      title: config.keyword,
      format: config.videoFormat
    });
  }

  console.log('✅ Pipeline complete!');
  return { content, videoPath, research };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  generateVideo: true,
  videoFormat: 'reels'
});
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const config = req.body;
    const result = await runContentPipeline(config);
    
    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Content generation failed',
      message: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { conductResearch } from '@/lib/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword } = req.query;

  if (!keyword || typeof keyword !== 'string') {
    return res.status(400).json({ error: 'Keyword is required' });
  }

  try {
    const research = await conductResearch(keyword);
    res.status(200).json({ success: true, data: research });
  } catch (error) {
    res.status(500).json({ 
      error: 'Research failed',
      message: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple content variations
async function generateBatchContent(
  keyword: string,
  formats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'>
): Promise<Record<string, string>> {
  const research = await conductResearch(keyword);
  
  const contentPromises = formats.map(format =>
    generateContent({
      format,
      language: 'en',
      tone: 'expert',
      researchData: research
    })
  );

  const contents = await Promise.all(contentPromises);
  
  return formats.reduce((acc, format, index) => {
    acc[format] = contents[index];
    return acc;
  }, {} as Record<string, string>);
}
```

### Scheduled Content Pipeline

```typescript
// Schedule daily content generation
import cron from 'node-cron';

function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 6 AM
  cron.schedule('0 6 * * *', async () => {
    for (const keyword of keywords) {
      try {
        await runContentPipeline({
          keyword,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
          generateVideo: true,
          videoFormat: 'reels'
        });
      } catch (error) {
        console.error(`Failed for keyword: ${keyword}`, error);
      }
    }
  });
}
```

### Content Versioning

```typescript
// Generate bilingual content
async function generateBilingualContent(
  keyword: string,
  format: ContentConfig['format']
) {
  const research = await conductResearch(keyword);
  
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      format,
      language: 'en',
      tone: 'expert',
      researchData: research
    }),
    generateContent({
      format,
      language: 'vi',
      tone: 'expert',
      researchData: research
    })
  ]);

  return { en: englishContent, vi: vietnameseContent };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function generateContentWithRetry(
  config: ContentConfig,
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(config);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const waitTime = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1} after ${waitTime}ms`);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// For large videos, render in chunks
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(config: VideoConfig) {
  // Increase Node.js memory limit
  // Run with: node --max-old-space-size=4096 your-script.js
  
  // Or render frames individually for very large projects
  const frames = await renderFrames({
    // ... config
    concurrency: 1, // Reduce concurrency
    frameRange: [0, 100] // Render in chunks
  });
}
```

### Missing Environment Variables

```typescript
// Validate environment variables on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
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

### Remotion Composition Not Found

```typescript
// Ensure composition is properly registered
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './ContentVideo';

registerRoot(() => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920} // 9:16 for vertical video
        defaultProps={{
          title: 'Default Title',
          content: 'Default content',
          format: 'reels'
        }}
      />
    </>
  );
});
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Implement request queuing** for AI API calls to manage rate limits
3. **Store generated content** in a database for reuse and versioning
4. **Monitor API costs** - Claude and OpenAI usage can add up quickly
5. **Test video compositions** using `npm run remotion:preview` before rendering
6. **Use TypeScript strictly** - enable all type checking for better reliability
7. **Log pipeline steps** for debugging and monitoring production runs
