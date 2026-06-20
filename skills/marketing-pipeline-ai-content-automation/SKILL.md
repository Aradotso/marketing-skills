---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create content using Claude and OpenAI APIs
  - automate content creation from keyword to video
  - use the marketing pipeline for content generation
  - set up automated content research and posting
  - build AI-powered content automation workflow
  - configure the ultimate AI content pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that transforms keywords into complete content pieces with automated research, multi-format content generation (articles, social posts), and video rendering. Built with TypeScript, Next.js, and integrates Claude 3, OpenAI, and Remotion for video generation.

## What It Does

The Ultimate AI Content Pipeline automates:

1. **Auto Research** - Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation** - Creates articles in multiple formats (toplists, POV, case studies, how-to) using Claude/OpenAI
3. **Multi-language Support** - Generates content in both English and Vietnamese
4. **Video Generation** - Automatically renders infographics and short videos using Remotion
5. **Multi-platform Optimization** - Outputs content optimized for Reels, TikTok, Shorts

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
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/           # AI integration (Claude, OpenAI)
│   │   ├── research/     # Web scraping & research
│   │   ├── video/        # Remotion video generation
│   │   └── content/      # Content formatting
│   ├── types/            # TypeScript definitions
│   └── config/           # Configuration files
├── public/               # Static assets
└── remotion/             # Remotion video templates
```

## Core Usage Patterns

### 1. Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the app at `http://localhost:3000`

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(keyword: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const systemPrompt = `You are an expert content creator specializing in ${format} format content.`;
  
  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Create a ${format} article about: ${keyword}. Include data-backed insights and trending information.`
      }
    ]
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(keyword: string, language: 'en' | 'vi') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content writer creating engaging articles in ${language === 'vi' ? 'Vietnamese' : 'English'}.`
      },
      {
        role: 'user',
        content: `Write a comprehensive article about: ${keyword}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000,
  });

  return completion.choices[0].message.content;
}
```

### 4. Automated Research Pipeline

```typescript
interface ResearchSource {
  name: string;
  url: string;
  type: 'news' | 'social' | 'blog';
}

async function autoResearch(keyword: string): Promise<ResearchData[]> {
  const sources: ResearchSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com', type: 'news' },
    { name: 'a16z', url: 'https://a16z.com', type: 'blog' },
  ];

  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await fetch(`https://api.rapidapi.com/search`, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          'X-RapidAPI-Host': 'your-api-host',
        },
        method: 'POST',
        body: JSON.stringify({ query: keyword, source: source.url }),
      });

      return response.json();
    })
  );

  return results.flat();
}

interface ResearchData {
  title: string;
  excerpt: string;
  url: string;
  publishedAt: string;
  source: string;
}
```

### 5. Content Pipeline Orchestration

```typescript
interface ContentPipeline {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  includeVideo: boolean;
}

async function runContentPipeline(config: ContentPipeline) {
  // Step 1: Research
  console.log('Starting research phase...');
  const researchData = await autoResearch(config.keyword);

  // Step 2: Generate content
  console.log('Generating content...');
  const content = await generateContent(config.keyword, config.format);

  // Step 3: Translate if needed
  let translatedContent = content;
  if (config.language === 'vi') {
    translatedContent = await translateContent(content, 'vi');
  }

  // Step 4: Generate video if requested
  let videoUrl = null;
  if (config.includeVideo) {
    console.log('Rendering video...');
    videoUrl = await renderVideo(translatedContent);
  }

  return {
    content: translatedContent,
    research: researchData,
    videoUrl,
    metadata: {
      keyword: config.keyword,
      format: config.format,
      language: config.language,
      createdAt: new Date().toISOString(),
    },
  };
}
```

### 6. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderVideo(content: string): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.split('\n')[0],
      content: content,
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `video-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: content.split('\n')[0],
      content: content,
    },
  });

  return outputLocation;
}
```

### 7. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, includeVideo } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      includeVideo: includeVideo || false,
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 8. React Component Example

```typescript
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          includeVideo: true,
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
        className="bg-blue-500 text-white px-4 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h2 className="text-xl font-bold mb-2">Generated Content</h2>
          <div className="prose">{result.content}</div>
          {result.videoUrl && (
            <video src={result.videoUrl} controls className="mt-4 w-full" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Configuration Options

### Content Formats

- `toplist` - Numbered lists with rankings
- `pov` - Point of view / opinion pieces
- `case-study` - Detailed case study analysis
- `how-to` - Step-by-step tutorials

### Supported Languages

- `en` - English
- `vi` - Vietnamese (Tiếng Việt)

### Tone/Voice Options

```typescript
type ToneOption = 'expert' | 'friendly' | 'humorous' | 'professional';

function getSystemPromptForTone(tone: ToneOption): string {
  const tones = {
    expert: 'You are a subject matter expert writing authoritative content.',
    friendly: 'You are a friendly writer creating approachable content.',
    humorous: 'You are a witty writer creating entertaining content.',
    professional: 'You are a business professional writing formal content.',
  };
  return tones[tone];
}
```

## Common Workflows

### Full Content Pipeline

```typescript
// Complete workflow from keyword to published content
async function fullPipeline(keyword: string) {
  const pipeline = await runContentPipeline({
    keyword,
    format: 'toplist',
    language: 'en',
    includeVideo: true,
  });

  // Save to database
  await saveContent(pipeline);

  // Schedule for publishing
  await schedulePost(pipeline, new Date(Date.now() + 3600000)); // 1 hour later

  return pipeline;
}
```

### Multi-language Content Generation

```typescript
async function generateMultiLanguage(keyword: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    runContentPipeline({ keyword, format: 'toplist', language: 'en', includeVideo: false }),
    runContentPipeline({ keyword, format: 'toplist', language: 'vi', includeVideo: false }),
  ]);

  return { englishContent, vietnameseContent };
}
```

## Troubleshooting

### API Key Issues

**Problem**: `Authentication failed` errors

**Solution**: Verify environment variables are properly set:

```bash
# Check if env vars are loaded
node -e "console.log(process.env.ANTHROPIC_API_KEY ? 'Set' : 'Not set')"
```

### Rate Limiting

**Problem**: Too many API requests

**Solution**: Implement rate limiting:

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  keywords.map(keyword => 
    limit(() => generateContent(keyword, 'toplist'))
  )
);
```

### Video Rendering Fails

**Problem**: Remotion rendering errors

**Solution**: Ensure ffmpeg is installed and check logs:

```bash
# Install ffmpeg
brew install ffmpeg  # macOS
# or
sudo apt-get install ffmpeg  # Ubuntu

# Check Remotion logs
npm run remotion:preview
```

### Content Quality Issues

**Problem**: Generated content lacks depth

**Solution**: Enhance prompts with research data:

```typescript
async function enhancedGeneration(keyword: string) {
  const research = await autoResearch(keyword);
  
  const contextualPrompt = `
    Based on the following recent research:
    ${research.map(r => `- ${r.title}: ${r.excerpt}`).join('\n')}
    
    Create a comprehensive article about: ${keyword}
  `;

  return generateContent(contextualPrompt, 'case-study');
}
```

### Memory Issues with Large Content

**Problem**: Node.js heap out of memory

**Solution**: Increase Node memory limit:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

## Build and Deploy

```bash
# Build for production
npm run build

# Start production server
npm start

# Run with PM2 for production
pm2 start npm --name "content-pipeline" -- start
```

## Additional Resources

- Refer to `HUONG_DAN_CAI_DAT.md` for detailed Vietnamese installation guide
- Check Next.js documentation for deployment options
- Review Remotion docs for advanced video customization
- Anthropic Claude API docs for prompt engineering best practices
