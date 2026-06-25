---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate marketing content from keywords
  - create videos from blog posts automatically
  - scrape and analyze news for content ideas
  - build content pipeline with Claude and OpenAI
  - render social media videos with Remotion
  - automate research and script generation
  - create multilingual content with AI
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation system that transforms a single keyword into fully researched articles and videos. It crawls recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI, and automatically renders videos using Remotion.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install

# Set up environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Database (if applicable)
DATABASE_URL=your_database_url
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js 13+ app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── public/              # Static assets
├── remotion/            # Video templates
└── package.json
```

## Core Features & Usage

### 1. Research & Crawling

The system automatically crawls news sources to gather recent information:

```typescript
// src/lib/crawler/news-scraper.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
  source: string;
}

export async function scrapeNewsByKeyword(
  keyword: string,
  options?: {
    timeRange?: '24h' | '7d' | '30d';
    sources?: string[];
    language?: 'en' | 'vi';
  }
): Promise<NewsArticle[]> {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY);
  
  const articles = await rapidAPI.searchNews({
    q: keyword,
    time_range: options?.timeRange || '24h',
    sources: options?.sources?.join(','),
    lang: options?.language || 'en'
  });

  return articles.map(article => ({
    title: article.title,
    url: article.url,
    publishedAt: article.published_at,
    content: article.description,
    source: article.source_name
  }));
}

// Usage example
const articles = await scrapeNewsByKeyword('AI automation', {
  timeRange: '24h',
  sources: ['techcrunch', 'a16z'],
  language: 'en'
});
```

### 2. AI Content Generation

Generate content using Claude or OpenAI based on research:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type ToneOfVoice = 'expert' | 'friendly' | 'humorous';

interface GenerateContentParams {
  keyword: string;
  research: string[];
  format: ContentFormat;
  tone: ToneOfVoice;
  language: 'en' | 'vi';
  provider?: 'claude' | 'openai';
}

export async function generateContent(
  params: GenerateContentParams
): Promise<{ title: string; content: string; metadata: any }> {
  const { keyword, research, format, tone, language, provider = 'claude' } = params;

  const prompt = buildPrompt(keyword, research, format, tone, language);

  if (provider === 'claude') {
    return generateWithClaude(prompt, language);
  } else {
    return generateWithOpenAI(prompt, language);
  }
}

async function generateWithClaude(prompt: string, language: string) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
    system: `You are an expert content writer creating ${language === 'vi' ? 'Vietnamese' : 'English'} marketing content.`
  });

  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';

  return parseContentResponse(content);
}

async function generateWithOpenAI(prompt: string, language: string) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content writer creating ${language === 'vi' ? 'Vietnamese' : 'English'} marketing content.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    max_tokens: 4096,
  });

  const content = completion.choices[0]?.message?.content || '';
  return parseContentResponse(content);
}

function buildPrompt(
  keyword: string,
  research: string[],
  format: ContentFormat,
  tone: ToneOfVoice,
  language: string
): string {
  return `
Create a ${format} article about "${keyword}" in ${language}.

Recent research data:
${research.join('\n\n')}

Requirements:
- Format: ${format}
- Tone: ${tone}
- Include data-backed insights
- Use recent examples (last 24-48 hours)
- Make it engaging and actionable
${language === 'vi' ? '- Write naturally in Vietnamese' : '- Write in clear English'}

Output as JSON:
{
  "title": "Article title",
  "subtitle": "Engaging subtitle",
  "content": "Full article content with sections",
  "keyPoints": ["key point 1", "key point 2"],
  "cta": "Call to action"
}
`;
}

function parseContentResponse(response: string) {
  try {
    const jsonMatch = response.match(/\{[\s\S]*\}/);
    if (jsonMatch) {
      const parsed = JSON.parse(jsonMatch[0]);
      return {
        title: parsed.title,
        content: parsed.content,
        metadata: {
          subtitle: parsed.subtitle,
          keyPoints: parsed.keyPoints,
          cta: parsed.cta
        }
      };
    }
  } catch (error) {
    console.error('Failed to parse AI response:', error);
  }
  
  return {
    title: 'Generated Content',
    content: response,
    metadata: {}
  };
}
```

### 3. Video Generation with Remotion

Automatically render videos from content:

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  title: string;
  keyPoints: string[];
  template: 'reels' | 'tiktok' | 'shorts';
  duration?: number;
}

export async function renderContentVideo(
  content: VideoRenderOptions
): Promise<string> {
  const { title, keyPoints, template, duration = 30 } = content;

  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts'),
    () => undefined,
    {
      webpackOverride: (config) => config,
    }
  );

  // Get composition
  const compositionId = `${template}-template`;
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title,
      keyPoints,
      duration: duration * 30, // 30 fps
    },
  });

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
      title,
      keyPoints,
    },
  });

  return outputLocation;
}

// Usage example
const videoPath = await renderContentVideo({
  title: 'Top 5 AI Tools for 2024',
  keyPoints: [
    'ChatGPT for writing',
    'Midjourney for images',
    'Runway for videos',
    'ElevenLabs for voice',
    'Jasper for marketing'
  ],
  template: 'reels',
  duration: 30
});
```

### 4. Complete Pipeline Integration

Combine all features into a single workflow:

```typescript
// src/lib/pipeline/content-pipeline.ts
import { scrapeNewsByKeyword } from '@/lib/crawler/news-scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render-video';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoTemplate?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting content pipeline for: ${config.keyword}`);

  // Step 1: Research
  console.log('📡 Scraping news and research...');
  const articles = await scrapeNewsByKeyword(config.keyword, {
    timeRange: '24h',
    sources: ['techcrunch', 'a16z']
  });

  const research = articles.map(a => 
    `${a.title}\nSource: ${a.source}\n${a.content}`
  );

  // Step 2: Generate content for each language
  console.log('🧠 Generating content with AI...');
  const contentResults = [];

  for (const language of config.languages) {
    const result = await generateContent({
      keyword: config.keyword,
      research,
      format: config.format,
      tone: config.tone,
      language,
      provider: 'claude'
    });

    contentResults.push({
      language,
      ...result
    });
  }

  // Step 3: Generate video if requested
  let videoPath;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    const mainContent = contentResults.find(c => c.language === 'en') || contentResults[0];
    
    videoPath = await renderContentVideo({
      title: mainContent.title,
      keyPoints: mainContent.metadata.keyPoints || [],
      template: config.videoTemplate || 'reels',
      duration: 30
    });
  }

  console.log('✅ Pipeline complete!');
  
  return {
    content: contentResults,
    video: videoPath,
    research: articles
  };
}

// Example usage
const result = await runContentPipeline({
  keyword: 'AI automation trends 2024',
  format: 'toplist',
  tone: 'expert',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoTemplate: 'reels'
});
```

## API Routes (Next.js)

### POST /api/generate

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, languages, generateVideo } = body;

    // Validate input
    if (!keyword || !format || !tone) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      tone,
      languages: languages || ['en'],
      generateVideo: generateVideo || false,
      videoTemplate: 'reels'
    });

    return NextResponse.json({
      success: true,
      data: result
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### Client-side usage:

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (formData: FormData) => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: formData.get('keyword'),
          format: formData.get('format'),
          tone: formData.get('tone'),
          languages: ['en', 'vi'],
          generateVideo: formData.get('generateVideo') === 'on'
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
    <div className="content-generator">
      <form action={handleGenerate}>
        <input name="keyword" placeholder="Enter keyword..." required />
        <select name="format">
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How To</option>
        </select>
        <select name="tone">
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>
        <label>
          <input type="checkbox" name="generateVideo" />
          Generate Video
        </label>
        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="results">
          {result.data.content.map((c, i) => (
            <div key={i}>
              <h3>{c.title}</h3>
              <p>{c.language.toUpperCase()}</p>
              <div dangerouslySetInnerHTML={{ __html: c.content }} />
            </div>
          ))}
          {result.data.video && (
            <video src={result.data.video} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### 1. Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'expert',
        languages: ['en'],
        generateVideo: false
      })
    )
  );
  return results;
}
```

### 2. Custom Research Sources

```typescript
import { scrapeNewsByKeyword } from '@/lib/crawler/news-scraper';

async function getCustomResearch(keyword: string) {
  const [techNews, businessNews, socialMedia] = await Promise.all([
    scrapeNewsByKeyword(keyword, { sources: ['techcrunch', 'a16z'] }),
    scrapeNewsByKeyword(keyword, { sources: ['forbes', 'bloomberg'] }),
    scrapeTwitterPosts(keyword) // Custom function
  ]);

  return [...techNews, ...businessNews, ...socialMedia];
}
```

### 3. Scheduled Content Generation

```typescript
// Using node-cron or similar
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await getTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline({
      keyword: topic,
      format: 'toplist',
      tone: 'expert',
      languages: ['en', 'vi'],
      generateVideo: true
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pRetry from 'p-retry';

async function generateWithRetry(params: GenerateContentParams) {
  return pRetry(
    () => generateContent(params),
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
}
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps: { title, keyPoints },
  timeoutInMilliseconds: 300000 // 5 minutes
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
async function generateLargeContent(keyword: string) {
  const research = await scrapeNewsByKeyword(keyword);
  
  // Split research into chunks
  const chunkSize = 10;
  const chunks = [];
  
  for (let i = 0; i < research.length; i += chunkSize) {
    chunks.push(research.slice(i, i + chunkSize));
  }
  
  // Process each chunk
  const results = [];
  for (const chunk of chunks) {
    const result = await generateContent({
      keyword,
      research: chunk.map(a => a.content),
      format: 'toplist',
      tone: 'expert',
      language: 'en'
    });
    results.push(result);
  }
  
  return results;
}
```

### Claude API Context Length

```typescript
// Summarize research before sending to Claude
async function summarizeResearch(articles: NewsArticle[]) {
  const summaries = await Promise.all(
    articles.map(async article => {
      const response = await anthropic.messages.create({
        model: 'claude-3-haiku-20240307', // Faster model for summarization
        max_tokens: 500,
        messages: [{
          role: 'user',
          content: `Summarize this in 2-3 sentences: ${article.content}`
        }]
      });
      return response.content[0].type === 'text' 
        ? response.content[0].text 
        : '';
    })
  );
  
  return summaries;
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run tests
npm test

# Lint code
npm run lint
```

## Key Configuration Files

### next.config.js
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    serverActions: true,
  },
  webpack: (config) => {
    // Remotion requires special webpack config
    config.resolve.alias = {
      ...config.resolve.alias,
      'remotion': path.resolve(__dirname, 'node_modules/remotion'),
    };
    return config;
  },
};

module.exports = nextConfig;
```

### tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

This skill enables AI agents to effectively use the Marketing Pipeline Share project for automated content creation, from research to video generation.
