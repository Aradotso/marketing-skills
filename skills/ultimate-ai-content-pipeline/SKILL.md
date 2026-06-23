---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate videos automatically from blog posts
  - crawl news sources and create content with AI
  - build automated marketing content workflow
  - create multilingual content with AI research
  - render videos from text using Remotion
  - automate social media content generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete AI-powered content automation pipeline that takes you from research to published content and video generation. It automatically crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates multilingual content (English/Vietnamese) using Claude/OpenAI, and renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls recent news from multiple sources within 24 hours
- **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual Support**: Generates content in English and Vietnamese simultaneously
- **Video Generation**: Automatically renders infographics and short videos using Remotion
- **Multi-Platform Export**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
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
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/             # Static assets
```

## Key API Endpoints

### Research & Crawling

```typescript
// src/lib/crawler/research.ts
import axios from 'axios';

interface NewsSource {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
}

export async function crawlNews(keyword: string, sources: string[] = ['techcrunch', 'a16z']): Promise<NewsSource[]> {
  const results: NewsSource[] = [];
  
  for (const source of sources) {
    const response = await axios.get('https://api.rapidapi.com/news/search', {
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      },
      params: {
        q: keyword,
        source: source,
        from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString(),
      },
    });
    
    results.push(...response.data.articles);
  }
  
  return results;
}
```

### AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: string[];
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = `
You are a professional content creator. Generate a ${request.format} article about "${request.keyword}" in ${request.language} language with a ${request.tone} tone.

Research data:
${request.research.join('\n\n')}

Requirements:
- Include specific data points and statistics
- Make it engaging and actionable
- Add relevant examples
- Structure with clear headings
`;

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
```

### AI Content Generation with OpenAI

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateContentOpenAI(request: ContentRequest): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a professional content creator specializing in marketing content.',
      },
      {
        role: 'user',
        content: `Generate a ${request.format} article about "${request.keyword}" in ${request.language}...`,
      },
    ],
    temperature: 0.7,
    max_tokens: 4000,
  });

  return completion.choices[0].message.content || '';
}
```

## Complete Content Pipeline

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlNews } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderVideo } from '@/lib/video/remotion-renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🔍 Starting research for: ${config.keyword}`);
  
  // Step 1: Research
  const newsData = await crawlNews(config.keyword);
  const researchSummary = newsData.map(n => `${n.title}: ${n.url}`);
  
  console.log(`📚 Found ${newsData.length} articles`);
  
  // Step 2: Generate content for each language
  const contents: Record<string, string> = {};
  
  for (const lang of config.languages) {
    console.log(`✍️ Generating ${lang} content...`);
    
    contents[lang] = await generateContent({
      keyword: config.keyword,
      format: config.format,
      language: lang,
      tone: 'expert',
      research: researchSummary,
    });
  }
  
  // Step 3: Generate video if requested
  let videoUrl: string | null = null;
  
  if (config.generateVideo) {
    console.log(`🎬 Rendering video...`);
    
    videoUrl = await renderVideo({
      title: config.keyword,
      content: contents['en'],
      format: 'reels', // 9:16 aspect ratio
    });
  }
  
  console.log(`✅ Pipeline complete!`);
  
  return {
    contents,
    videoUrl,
    researchSources: newsData.length,
  };
}
```

## Remotion Video Rendering

```typescript
// src/lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function renderVideo(config: VideoConfig): Promise<string> {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  // Render video
  const outputLocation = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition: compositions,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return `/videos/${path.basename(outputLocation)}`;
}
```

## Remotion Video Template

```typescript
// remotion/index.ts
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Default Title',
          content: 'Default content',
        }}
      />
    </>
  );
};
```

```tsx
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  const scale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          color: 'white',
          fontSize: 80,
          fontWeight: 'bold',
          textAlign: 'center',
          padding: 40,
        }}
      >
        {title}
      </div>
    </AbsoluteFill>
  );
};
```

## Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, format, languages, generateVideo } = body;
    
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      format,
      languages: languages || ['en', 'vi'],
      generateVideo: generateVideo || false,
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## React Component Example

```tsx
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          keyword,
          format,
          languages: ['en', 'vi'],
          generateVideo: true,
        }),
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
    <div className="p-6 max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">AI Content Generator</h1>
      
      <div className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-2">Keyword</label>
          <input
            type="text"
            value={keyword}
            onChange={(e) => setKeyword(e.target.value)}
            className="w-full px-4 py-2 border rounded-lg"
            placeholder="Enter your keyword..."
          />
        </div>
        
        <div>
          <label className="block text-sm font-medium mb-2">Format</label>
          <select
            value={format}
            onChange={(e) => setFormat(e.target.value as any)}
            className="w-full px-4 py-2 border rounded-lg"
          >
            <option value="toplist">Top List</option>
            <option value="pov">POV</option>
            <option value="case-study">Case Study</option>
            <option value="how-to">How To</option>
          </select>
        </div>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white py-3 rounded-lg hover:bg-blue-700 disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>
      
      {result && (
        <div className="mt-8 space-y-4">
          <h2 className="text-2xl font-bold">Results</h2>
          <p className="text-sm text-gray-600">
            Research sources: {result.researchSources}
          </p>
          
          {Object.entries(result.contents).map(([lang, content]) => (
            <div key={lang} className="border rounded-lg p-4">
              <h3 className="font-bold mb-2">{lang.toUpperCase()}</h3>
              <div className="prose">{content as string}</div>
            </div>
          ))}
          
          {result.videoUrl && (
            <div>
              <h3 className="font-bold mb-2">Generated Video</h3>
              <video src={result.videoUrl} controls className="w-full" />
            </div>
          )}
        </div>
      )}
    </div>
  );
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

# Render a video (if using CLI)
npm run render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: false,
    });
    
    results.push(result);
    
    // Add delay to respect API rate limits
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Research Sources

```typescript
async function crawlCustomSources(keyword: string, customUrls: string[]) {
  const results = [];
  
  for (const url of customUrls) {
    // Implement custom scraping logic
    const content = await fetch(url).then(r => r.text());
    results.push({ url, content });
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// Add retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Fails

Ensure Remotion dependencies are properly installed:

```bash
npm install @remotion/bundler @remotion/renderer @remotion/cli
```

Check that ffmpeg is installed on your system:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
apt-get install ffmpeg
```

### Claude API Timeout

Increase timeout and add streaming support:

```typescript
const message = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  messages: [...],
  stream: true,
});

for await (const event of message) {
  if (event.type === 'content_block_delta') {
    console.log(event.delta);
  }
}
```

### Environment Variables Not Loading

Ensure `.env.local` is in the root directory and restart the dev server:

```bash
# Kill any running processes
pkill -f "next dev"

# Restart
npm run dev
```
