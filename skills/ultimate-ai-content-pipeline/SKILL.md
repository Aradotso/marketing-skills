---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to script to video using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up content pipeline with research and video generation
  - generate blog posts and videos from keywords automatically
  - use Claude and OpenAI for content automation
  - create automated marketing content with Remotion
  - build AI content workflow from research to publishing
  - scrape news and generate content automatically
  - render videos from blog content using AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that transforms a single keyword into fully-researched articles and rendered videos. It crawls recent news from sources like TechCrunch, Twitter, and LinkedIn, generates multi-format content using Claude/OpenAI, and automatically renders videos using Remotion.

## What This Project Does

This pipeline automates the entire content creation workflow:

1. **Research Phase**: Automatically crawls and scrapes fresh data from major news sources (last 24h)
2. **Content Generation**: Uses Claude 3 or OpenAI to write articles in multiple formats (Toplist, POV, Case Study, How-to)
3. **Multi-language Support**: Generates content in both English and Vietnamese simultaneously
4. **Video Rendering**: Converts written content into video infographics using Remotion
5. **Multi-platform Optimization**: Exports videos in formats suitable for Reels, TikTok, and Shorts

## Installation

### Prerequisites

- Node.js 18+ and npm/yarn
- API keys for:
  - Anthropic Claude (`ANTHROPIC_API_KEY`)
  - OpenAI (`OPENAI_API_KEY`)
  - RapidAPI for web scraping (`RAPIDAPI_KEY`)

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment template
cp .env.example .env

# Edit .env with your API keys
# ANTHROPIC_API_KEY=your_key_here
# OPENAI_API_KEY=your_key_here
# RAPIDAPI_KEY=your_key_here
```

### Environment Configuration

Create a `.env` file in the project root:

```env
# AI Providers
ANTHROPIC_API_KEY=
OPENAI_API_KEY=

# Research/Scraping
RAPIDAPI_KEY=

# Database (if applicable)
DATABASE_URL=

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Video Rendering
REMOTION_LICENSE_KEY=
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key APIs and Usage Patterns

### 1. Research & Scraping Module

```typescript
// lib/scraper/news-crawler.ts
import { crawlNews } from '@/lib/scraper/news-crawler';

interface NewsSource {
  name: string;
  url: string;
  selector: string;
}

export async function researchKeyword(keyword: string) {
  const sources: NewsSource[] = [
    { name: 'TechCrunch', url: 'techcrunch.com', selector: '.post-block' },
    { name: 'Twitter', url: 'x.com', selector: '[data-testid="tweet"]' },
    { name: 'LinkedIn', url: 'linkedin.com', selector: '.feed-shared-update' }
  ];

  const articles = await crawlNews({
    keyword,
    sources,
    timeRange: '24h',
    maxResults: 20
  });

  return articles;
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

export async function generateContent(
  keyword: string,
  researchData: any[],
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  provider: 'claude' | 'openai' = 'claude'
) {
  const prompt = buildPrompt(keyword, researchData, format, language);

  if (provider === 'claude') {
    const response = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt
        }
      ]
    });

    return response.content[0].text;
  } else {
    const response = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content writer and marketer.'
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      max_tokens: 4096
    });

    return response.choices[0].message.content;
  }
}

function buildPrompt(
  keyword: string,
  research: any[],
  format: string,
  language: string
): string {
  const langInstructions = language === 'vi' 
    ? 'Viết bằng tiếng Việt, giọng điệu chuyên nghiệp nhưng dễ hiểu'
    : 'Write in English with a professional yet accessible tone';

  return `
Topic: ${keyword}

Research Data:
${research.map(r => `- ${r.title}: ${r.summary}`).join('\n')}

Task: Write a ${format} article about ${keyword}.
${langInstructions}

Format Instructions:
${getFormatInstructions(format)}

Requirements:
- Include data and statistics from the research
- Add actionable insights
- Make it engaging and shareable
- Length: 1200-1500 words
`;
}

function getFormatInstructions(format: string): string {
  const instructions = {
    'toplist': '- Create a numbered list (Top 5-10)\n- Each item should have title, explanation, and example',
    'pov': '- Start with a strong opinion/perspective\n- Use "I believe" or "My view"\n- Back with evidence and reasoning',
    'case-study': '- Structure: Problem → Solution → Results\n- Include specific metrics and outcomes',
    'how-to': '- Step-by-step guide format\n- Include prerequisites and expected outcomes'
  };

  return instructions[format] || '';
}
```

### 3. Complete Content Pipeline

```typescript
// lib/content/pipeline.ts
import { researchKeyword } from '@/lib/scraper/news-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';

export interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  aiProvider: 'claude' | 'openai';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for keyword: ${config.keyword}`);

  // Step 1: Research
  console.log('📡 Crawling research data...');
  const research = await researchKeyword(config.keyword);
  console.log(`✅ Found ${research.length} articles`);

  // Step 2: Generate content in all languages
  const content = {};
  for (const lang of config.languages) {
    console.log(`🧠 Generating ${lang} content...`);
    content[lang] = await generateContent(
      config.keyword,
      research,
      config.format,
      lang,
      config.aiProvider
    );
    console.log(`✅ ${lang} content generated (${content[lang].length} chars)`);
  }

  // Step 3: Render video if requested
  let videos = {};
  if (config.generateVideo) {
    for (const lang of config.languages) {
      console.log(`🎬 Rendering ${lang} video...`);
      videos[lang] = await renderVideo({
        content: content[lang],
        format: config.videoFormat || 'reels',
        language: lang
      });
      console.log(`✅ ${lang} video rendered: ${videos[lang].url}`);
    }
  }

  return {
    research,
    content,
    videos,
    metadata: {
      keyword: config.keyword,
      format: config.format,
      timestamp: new Date().toISOString()
    }
  };
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  language: 'en' | 'vi';
}

const VIDEO_DIMENSIONS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

export async function renderVideo(config: VideoConfig) {
  // Parse content into video segments
  const segments = parseContentToSegments(config.content);

  // Bundle Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      segments,
      language: config.language
    }
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${config.language}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      segments,
      language: config.language,
      ...VIDEO_DIMENSIONS[config.format]
    }
  });

  return {
    url: outputLocation.replace(process.cwd() + '/public', ''),
    format: config.format,
    language: config.language
  };
}

function parseContentToSegments(content: string) {
  // Split content into video-friendly segments
  const paragraphs = content.split('\n\n').filter(p => p.trim());
  
  return paragraphs.map((text, index) => ({
    id: index,
    text: text.trim(),
    duration: Math.max(3, Math.min(8, text.length / 100)) // 3-8 seconds
  }));
}
```

### 5. Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/content/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const {
      keyword,
      format = 'toplist',
      languages = ['en', 'vi'],
      aiProvider = 'claude',
      generateVideo = false,
      videoFormat = 'reels'
    } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format,
      languages,
      aiProvider,
      generateVideo,
      videoFormat
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: error.message || 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

### 6. Frontend Usage Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          aiProvider: 'claude',
          generateVideo: true,
          videoFormat: 'reels'
        })
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full"
      />
      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="mt-4 bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h3 className="font-bold">Content Generated!</h3>
          <div className="mt-4">
            <h4>English Version:</h4>
            <div className="p-4 bg-gray-100 rounded">
              {result.content.en}
            </div>
          </div>
          {result.videos?.en && (
            <div className="mt-4">
              <h4>Video:</h4>
              <video src={result.videos.en.url} controls className="w-full" />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { runContentPipeline } from '@/lib/content/pipeline';

const keywords = [
  'AI Marketing Trends 2024',
  'Content Automation Tools',
  'Video Marketing Strategy'
];

async function batchGenerate() {
  for (const keyword of keywords) {
    console.log(`\n📝 Processing: ${keyword}`);
    
    const result = await runContentPipeline({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      aiProvider: 'claude',
      generateVideo: true,
      videoFormat: 'reels'
    });

    // Save to database or file system
    await saveResult(keyword, result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
}

batchGenerate();
```

### Custom Content Formats

```typescript
// Add custom format handler
export async function generateNewsletterContent(
  keyword: string,
  research: any[]
) {
  const prompt = `
Create a newsletter about ${keyword}.

Structure:
1. Hook/Opening (1 sentence)
2. Main Insight (2-3 paragraphs)
3. Key Takeaways (bullet points)
4. Call to Action

Research:
${research.map(r => `- ${r.title}: ${r.summary}`).join('\n')}
`;

  return await generateContent(keyword, research, 'custom', 'en', 'claude');
}
```

## Configuration

### AI Provider Selection

Choose between Claude and OpenAI based on your needs:

```typescript
// Claude: Better for long-form, nuanced content
const claudeConfig = {
  aiProvider: 'claude',
  model: 'claude-3-5-sonnet-20241022',
  maxTokens: 4096
};

// OpenAI: Faster, good for structured content
const openaiConfig = {
  aiProvider: 'openai',
  model: 'gpt-4-turbo-preview',
  maxTokens: 4096
};
```

### Video Output Settings

```typescript
// remotion/config.ts
export const VIDEO_CONFIG = {
  fps: 30,
  durationInFrames: 300, // 10 seconds at 30fps
  codec: 'h264',
  audioCodec: 'aac',
  quality: 90,
  dimensions: {
    reels: { width: 1080, height: 1920 },
    landscape: { width: 1920, height: 1080 },
    square: { width: 1080, height: 1080 }
  }
};
```

## Troubleshooting

### API Rate Limiting

```typescript
// Add rate limiting wrapper
async function withRateLimit<T>(
  fn: () => Promise<T>,
  delayMs: number = 1000
): Promise<T> {
  const result = await fn();
  await new Promise(resolve => setTimeout(resolve, delayMs));
  return result;
}

// Usage
const content = await withRateLimit(
  () => generateContent(keyword, research, 'toplist', 'en'),
  2000
);
```

### Error Handling

```typescript
// Robust error handling
export async function safeGenerateContent(...args) {
  const maxRetries = 3;
  let lastError;

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(...args);
    } catch (error) {
      lastError = error;
      console.error(`Attempt ${i + 1} failed:`, error.message);
      
      if (error.status === 429) {
        // Rate limit - wait longer
        await new Promise(r => setTimeout(r, (i + 1) * 5000));
      } else if (error.status >= 500) {
        // Server error - retry
        await new Promise(r => setTimeout(r, 2000));
      } else {
        // Client error - don't retry
        throw error;
      }
    }
  }

  throw lastError;
}
```

### Memory Issues with Video Rendering

```typescript
// Use streaming for large videos
import { renderFrames } from '@remotion/renderer';

export async function renderVideoStreaming(config: VideoConfig) {
  const frames = await renderFrames({
    composition,
    onProgress: ({ renderedFrames, totalFrames }) => {
      console.log(`Progress: ${renderedFrames}/${totalFrames}`);
    },
    onDownload: (src) => {
      console.log('Downloading frame:', src);
    }
  });

  return frames;
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run batch content generation
npm run generate:batch

# Render video only
npm run video:render

# Type checking
npm run type-check

# Linting
npm run lint
```

This skill enables AI coding agents to effectively use the Ultimate AI Content Pipeline for automated content creation, from research through to video generation.
