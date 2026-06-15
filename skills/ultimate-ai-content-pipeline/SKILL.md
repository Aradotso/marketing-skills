---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated research and video generation pipeline
  - create content from research to video automatically
  - use Claude and OpenAI for content automation
  - generate videos from written content with Remotion
  - build an AI-powered content workflow
  - automate social media content production
  - research and publish content automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a complete content automation system that transforms keywords into published content and videos. It crawls real-time data from TechCrunch, a16z, Twitter/X, and LinkedIn, generates articles in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls fresh news and insights from major tech sources (24h data)
- **Multi-Format AI Writing**: Generates articles in various formats (toplist, POV, case study, how-to)
- **Bilingual Content**: Creates English and Vietnamese versions simultaneously
- **Video Generation**: Automatically renders infographics and short videos from content
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

Create a `.env.local` file in the root directory:

```env
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping and data collection
│   │   └── video/       # Remotion video generation
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Modules

### 1. Research Module

Automatically crawls and analyzes content from multiple sources:

```typescript
// src/lib/research/crawler.ts
import axios from 'axios';

interface ResearchResult {
  source: string;
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
}

export async function crawlTechNews(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<ResearchResult[]> {
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const response = await axios.get(
      `https://api.rapidapi.com/news/${source}`,
      {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
        params: {
          query: keyword,
          timeframe,
        },
      }
    );
    
    results.push(...response.data.articles);
  }
  
  return results;
}
```

### 2. AI Content Generation

Generate articles using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

export async function generateContent(
  researchData: string[],
  keyword: string,
  config: ContentConfig
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const prompt = buildPrompt(researchData, keyword, config);
  
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
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(
  data: string[],
  keyword: string,
  config: ContentConfig
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with 5-10 items',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and outcomes',
    'how-to': 'Provide step-by-step instructions',
  };
  
  return `
You are a ${config.tone} content writer creating a ${config.format} article.

Keyword: ${keyword}
Format: ${formatInstructions[config.format]}
Language: ${config.language}
Target Audience: ${config.targetAudience}

Research Data:
${data.join('\n\n')}

Create a comprehensive, engaging article with:
- Catchy headline
- Strong introduction
- Data-backed insights
- Actionable takeaways
- SEO optimization
`;
}
```

**Alternative with OpenAI:**

```typescript
export async function generateContentOpenAI(
  researchData: string[],
  keyword: string,
  config: ContentConfig
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${config.tone} content writer specializing in ${config.format} articles.`,
      },
      {
        role: 'user',
        content: buildPrompt(researchData, keyword, config),
      },
    ],
    temperature: 0.7,
    max_tokens: 4096,
  });
  
  return completion.choices[0].message.content || '';
}
```

### 3. Video Generation with Remotion

Transform content into video:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import { z } from 'zod';

export const contentVideoSchema = z.object({
  title: z.string(),
  points: z.array(z.string()),
  bgColor: z.string(),
  textColor: z.string(),
});

export const ContentVideo: React.FC<z.infer<typeof contentVideoSchema>> = ({
  title,
  points,
  bgColor,
  textColor,
}) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{
          fontSize: 60,
          color: textColor,
          textAlign: 'center',
          marginTop: 100,
          opacity: Math.min(1, frame / 30),
        }}>
          {title}
        </div>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <div style={{
            fontSize: 40,
            color: textColor,
            padding: 40,
            opacity: Math.min(1, (frame - (60 + index * 90)) / 20),
          }}>
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
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  title: string,
  points: string[],
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts')
  );
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title,
      points,
      bgColor: '#1a1a1a',
      textColor: '#ffffff',
    },
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      points,
      bgColor: '#1a1a1a',
      textColor: '#ffffff',
    },
  });
  
  return outputPath;
}
```

## Complete Pipeline Workflow

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlTechNews } from '../research/crawler';
import { generateContent } from '../ai/content-generator';
import { renderContentVideo } from '../video/renderer';
import { extractKeyPoints } from '../utils/content-parser';

export interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

export async function runContentPipeline(
  config: PipelineConfig
) {
  console.log('🔍 Step 1: Researching...');
  const researchResults = await crawlTechNews(config.keyword, '24h');
  const researchData = researchResults.map(r => r.content);
  
  const outputs = [];
  
  for (const language of config.languages) {
    console.log(`✍️ Step 2: Generating ${language} content...`);
    const content = await generateContent(
      researchData,
      config.keyword,
      {
        format: config.format,
        language,
        tone: config.tone,
        targetAudience: config.targetAudience,
      }
    );
    
    outputs.push({
      language,
      content,
      metadata: {
        keyword: config.keyword,
        format: config.format,
        createdAt: new Date(),
      },
    });
    
    if (config.generateVideo) {
      console.log(`🎬 Step 3: Rendering ${language} video...`);
      const keyPoints = extractKeyPoints(content, 5);
      const title = extractTitle(content);
      
      const videoPath = await renderContentVideo(
        title,
        keyPoints,
        `./output/video-${language}-${Date.now()}.mp4`
      );
      
      outputs[outputs.length - 1].videoPath = videoPath;
    }
  }
  
  console.log('✅ Pipeline complete!');
  return outputs;
}

function extractTitle(content: string): string {
  const lines = content.split('\n');
  return lines[0].replace(/^#\s*/, '').trim();
}
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo ?? false,
      tone: body.tone || 'expert',
      targetAudience: body.targetAudience || 'general audience',
    });
    
    return NextResponse.json({
      success: true,
      data: result,
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

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

// Generate a toplist article in English
const result = await runContentPipeline({
  keyword: 'AI automation tools 2024',
  format: 'toplist',
  languages: ['en'],
  generateVideo: false,
  tone: 'expert',
  targetAudience: 'startup founders',
});

console.log(result[0].content);
```

### Bilingual Content with Video

```typescript
// Generate content in both languages with video
const bilingualResult = await runContentPipeline({
  keyword: 'marketing automation trends',
  format: 'case-study',
  languages: ['en', 'vi'],
  generateVideo: true,
  tone: 'friendly',
  targetAudience: 'digital marketers',
});

// Access English version
console.log(bilingualResult[0].content);
console.log(bilingualResult[0].videoPath);

// Access Vietnamese version
console.log(bilingualResult[1].content);
console.log(bilingualResult[1].videoPath);
```

### Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        languages: ['en', 'vi'],
        generateVideo: true,
        tone: 'expert',
        targetAudience: 'marketers',
      }),
    });
    
    const data = await response.json();
    setResult(data.data);
    setLoading(false);
  };
  
  return (
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full mb-4"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-6">
          {result.map((item, idx) => (
            <div key={idx} className="mb-6">
              <h3 className="font-bold">
                {item.language.toUpperCase()} Version
              </h3>
              <pre className="bg-gray-100 p-4 rounded">
                {item.content}
              </pre>
              {item.videoPath && (
                <p className="text-green-600">
                  Video: {item.videoPath}
                </p>
              )}
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (local preview)
npm run remotion:preview

# Render video to file
npm run remotion:render
```

## Troubleshooting

### API Rate Limits

```typescript
// Add retry logic for API calls
async function apiCallWithRetry(
  fn: () => Promise<any>,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, 1000 * (i + 1))
      );
    }
  }
}
```

### Video Rendering Issues

Ensure ffmpeg is installed:
```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows
# Download from https://ffmpeg.org/download.html
```

### Memory Issues with Large Content

```typescript
// Process in batches
async function processBatch<T>(
  items: T[],
  batchSize: number,
  processor: (item: T) => Promise<any>
) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(processor)
    );
    results.push(...batchResults);
  }
  
  return results;
}
```

### Environment Variables Not Loading

Ensure `.env.local` is in the root directory and restart the dev server:

```bash
# Kill existing processes
pkill -f next

# Restart
npm run dev
```

## Best Practices

1. **Cache Research Results**: Store crawled data to avoid repeated API calls
2. **Queue Video Rendering**: Use a job queue for video generation to avoid blocking
3. **Validate Input**: Always validate user input before processing
4. **Monitor Costs**: Track API usage for Claude, OpenAI, and RapidAPI
5. **Version Content**: Store generated content with metadata for tracking
