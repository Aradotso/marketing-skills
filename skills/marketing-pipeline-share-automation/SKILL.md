---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, auto-posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI pipeline
  - set up automated marketing content workflow
  - generate videos from content automatically
  - research and create content with Claude API
  - build content automation system
  - create AI-powered content pipeline
  - automate social media content generation
  - set up content research to video pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end automated content creation system that handles the entire workflow from research to video generation. It crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates content in multiple formats and languages using Claude 3/OpenAI, and automatically renders videos using Remotion.

**Key Capabilities:**
- Auto-scan research from major news sources (24h data)
- AI-driven content generation in multiple formats (Toplist, POV, Case Study, How-to)
- Bilingual support (English/Vietnamese) with customizable tone
- Automatic video/infographic rendering for social platforms
- Auto-posting to Facebook pages
- Next.js-based UI with TypeScript

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

## Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media
FACEBOOK_PAGE_ACCESS_TOKEN=your_facebook_token_here
FACEBOOK_PAGE_ID=your_page_id_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Video rendering (Remotion)
npm run render
```

Access the application at `http://localhost:3000`

## Core Architecture

### 1. Research Module (Auto-Scan)

The research module crawls and analyzes content from various sources:

```typescript
// services/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  name: string;
  url: string;
  selector?: string;
}

export async function crawlNewsSources(keyword: string): Promise<Article[]> {
  const sources: ResearchSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com/search/' },
    { name: 'a16z', url: 'https://a16z.com/' },
  ];

  const articles: Article[] = [];

  for (const source of sources) {
    try {
      const response = await axios.get(source.url + keyword, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
      });

      // Parse and extract articles
      const parsedArticles = parseArticles(response.data, source.name);
      articles.push(...parsedArticles);
    } catch (error) {
      console.error(`Error crawling ${source.name}:`, error);
    }
  }

  return articles;
}

interface Article {
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
  source: string;
}

function parseArticles(data: any, sourceName: string): Article[] {
  // Implementation specific to each source
  return data.articles.map((article: any) => ({
    title: article.title,
    url: article.url,
    content: article.description,
    publishedAt: new Date(article.publishedAt),
    source: sourceName,
  }));
}
```

### 2. AI Content Generation

Generate content using Claude 3 or OpenAI:

```typescript
// services/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentGenerationOptions {
  keyword: string;
  researchData: Article[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
  provider?: 'claude' | 'openai';
}

export async function generateContent(
  options: ContentGenerationOptions
): Promise<string> {
  const {
    keyword,
    researchData,
    format,
    language,
    tone,
    provider = 'claude',
  } = options;

  const prompt = buildPrompt(keyword, researchData, format, language, tone);

  if (provider === 'claude') {
    return await generateWithClaude(prompt);
  } else {
    return await generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string): Promise<string> {
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

async function generateWithOpenAI(prompt: string): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
    max_tokens: 4096,
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(
  keyword: string,
  researchData: Article[],
  format: ContentFormat,
  language: Language,
  tone: Tone
): string {
  const languageInstruction =
    language === 'vi' ? 'Write in Vietnamese' : 'Write in English';
  const toneInstruction = getToneInstruction(tone);
  const formatInstruction = getFormatInstruction(format);

  const researchContext = researchData
    .map((article) => `- ${article.title}: ${article.content}`)
    .join('\n');

  return `
${languageInstruction}. ${toneInstruction}.

Create a ${format} article about: ${keyword}

Based on this recent research (last 24h):
${researchContext}

${formatInstruction}

Include data-backed insights and specific examples from the research.
  `.trim();
}

function getToneInstruction(tone: Tone): string {
  const tones = {
    expert: 'Use an authoritative, professional tone with industry terminology',
    friendly: 'Use a conversational, approachable tone',
    humorous: 'Use a light-hearted, entertaining tone with subtle humor',
  };
  return tones[tone];
}

function getFormatInstruction(format: ContentFormat): string {
  const formats = {
    toplist:
      'Structure as a numbered list with clear headings and explanations for each item',
    pov: 'Write from a specific point of view, sharing opinions and perspectives',
    'case-study':
      'Analyze a specific example with problem, solution, and results',
    'how-to': 'Provide step-by-step instructions with actionable tips',
  };
  return formats[format];
}
```

### 3. Video Generation with Remotion

Automatically render videos from content:

```typescript
// remotion/compositions/ContentVideo.tsx
import { Composition } from 'remotion';
import { VideoContent } from './VideoContent';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={VideoContent}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Title',
          content: [],
          theme: 'modern',
        }}
      />
    </>
  );
};
```

```typescript
// remotion/compositions/VideoContent.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string[];
  theme: 'modern' | 'minimal' | 'vibrant';
}

export const VideoContent: React.FC<ContentVideoProps> = ({
  title,
  content,
  theme,
}) => {
  const frame = useCurrentFrame();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        backgroundColor: theme === 'modern' ? '#1a1a1a' : '#ffffff',
        padding: 60,
      }}
    >
      <h1
        style={{
          fontSize: 72,
          fontWeight: 'bold',
          opacity: titleOpacity,
          color: theme === 'modern' ? '#ffffff' : '#000000',
        }}
      >
        {title}
      </h1>
      {content.map((item, index) => (
        <ContentItem key={index} text={item} index={index} frame={frame} />
      ))}
    </AbsoluteFill>
  );
};

const ContentItem: React.FC<{ text: string; index: number; frame: number }> = ({
  text,
  index,
  frame,
}) => {
  const startFrame = 30 + index * 60;
  const opacity = interpolate(frame, [startFrame, startFrame + 20], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <p
      style={{
        fontSize: 36,
        marginTop: 40,
        opacity,
      }}
    >
      {text}
    </p>
  );
};
```

```typescript
// services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: { title: string; points: string[] }
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      content: content.points,
      theme: 'modern',
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
      title: content.title,
      content: content.points,
      theme: 'modern',
    },
  });

  return outputLocation;
}
```

### 4. Auto-Posting to Facebook

```typescript
// services/social/facebookPoster.ts
import axios from 'axios';

interface FacebookPostOptions {
  message: string;
  videoUrl?: string;
  imageUrl?: string;
  link?: string;
}

export async function postToFacebookPage(
  options: FacebookPostOptions
): Promise<string> {
  const { message, videoUrl, imageUrl, link } = options;
  const pageId = process.env.FACEBOOK_PAGE_ID;
  const accessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN;

  try {
    let endpoint = `https://graph.facebook.com/v18.0/${pageId}/feed`;
    const params: any = {
      message,
      access_token: accessToken,
    };

    if (videoUrl) {
      endpoint = `https://graph.facebook.com/v18.0/${pageId}/videos`;
      params.file_url = videoUrl;
    } else if (imageUrl) {
      endpoint = `https://graph.facebook.com/v18.0/${pageId}/photos`;
      params.url = imageUrl;
    } else if (link) {
      params.link = link;
    }

    const response = await axios.post(endpoint, params);
    return response.data.id;
  } catch (error) {
    console.error('Facebook posting error:', error);
    throw error;
  }
}
```

## Complete Pipeline Example

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNewsSources } from '@/services/research/crawler';
import { generateContent } from '@/services/ai/contentGenerator';
import { renderContentVideo } from '@/services/video/renderer';
import { postToFacebookPage } from '@/services/social/facebookPoster';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone } = await request.json();

    // Step 1: Research
    console.log('Starting research phase...');
    const researchData = await crawlNewsSources(keyword);

    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContent({
      keyword,
      researchData,
      format,
      language,
      tone,
    });

    // Step 3: Extract key points for video
    const contentPoints = extractKeyPoints(content);

    // Step 4: Render Video
    console.log('Rendering video...');
    const videoPath = await renderContentVideo({
      title: keyword,
      points: contentPoints,
    });

    // Step 5: Post to Facebook
    console.log('Posting to Facebook...');
    const postId = await postToFacebookPage({
      message: content.substring(0, 500) + '...',
      videoUrl: `${process.env.NEXT_PUBLIC_APP_URL}${videoPath}`,
    });

    return NextResponse.json({
      success: true,
      content,
      videoPath,
      facebookPostId: postId,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - can be enhanced with AI
  const lines = content.split('\n').filter((line) => line.trim().length > 0);
  return lines.slice(0, 5);
}
```

## Frontend Integration

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [language, setLanguage] = useState<'en' | 'vi'>('en');
  const [tone, setTone] = useState<'expert' | 'friendly' | 'humorous'>('expert');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format, language, tone }),
      });
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <div className="space-y-4 mb-8">
        <input
          type="text"
          placeholder="Enter keyword..."
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          className="w-full p-3 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To</option>
        </select>
        
        <div className="flex gap-4">
          <select
            value={language}
            onChange={(e) => setLanguage(e.target.value as any)}
            className="flex-1 p-3 border rounded"
          >
            <option value="en">English</option>
            <option value="vi">Tiếng Việt</option>
          </select>
          
          <select
            value={tone}
            onChange={(e) => setTone(e.target.value as any)}
            className="flex-1 p-3 border rounded"
          >
            <option value="expert">Expert</option>
            <option value="friendly">Friendly</option>
            <option value="humorous">Humorous</option>
          </select>
        </div>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full p-3 bg-blue-600 text-white rounded disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>
      
      {result && (
        <div className="space-y-4">
          <div className="p-4 border rounded">
            <h2 className="font-bold mb-2">Generated Content:</h2>
            <p className="whitespace-pre-wrap">{result.content}</p>
          </div>
          
          {result.videoPath && (
            <div className="p-4 border rounded">
              <h2 className="font-bold mb-2">Video:</h2>
              <video src={result.videoPath} controls className="w-full" />
            </div>
          )}
          
          {result.facebookPostId && (
            <div className="p-4 border rounded bg-green-50">
              <p>✅ Posted to Facebook: {result.facebookPostId}</p>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Scheduling Content Generation

```typescript
// lib/scheduler.ts
import cron from 'node-cron';
import { crawlNewsSources } from '@/services/research/crawler';
import { generateContent } from '@/services/ai/contentGenerator';

export function scheduleDailyContentGeneration() {
  // Run daily at 9 AM
  cron.schedule('0 9 * * *', async () => {
    const keywords = ['AI trends', 'Marketing automation', 'SaaS growth'];
    
    for (const keyword of keywords) {
      const researchData = await crawlNewsSources(keyword);
      const content = await generateContent({
        keyword,
        researchData,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
      });
      
      // Save to database or queue for review
      await saveToContentQueue(content);
    }
  });
}
```

### Batch Processing

```typescript
// services/batch/processor.ts
export async function processBatchKeywords(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const researchData = await crawlNewsSources(keyword);
      return await generateContent({
        keyword,
        researchData,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
      });
    })
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    content: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/rateLimiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function crawlWithRateLimit(keywords: string[]) {
  return await Promise.all(
    keywords.map((keyword) =>
      limit(() => crawlNewsSources(keyword))
    )
  );
}
```

### Error Handling

```typescript
// lib/errorHandler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering' | 'posting',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function executePipelineWithRetry(
  fn: () => Promise<any>,
  retries = 3
): Promise<any> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise((resolve) => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

### Video Rendering Optimization

```typescript
// Adjust Remotion settings for faster rendering
export const RENDERING_CONFIG = {
  concurrency: 4, // Number of parallel rendering threads
  codec: 'h264',
  quality: 80, // Balance between quality and file size
  crf: 18, // Lower = higher quality, higher file size
};
```

## Best Practices

1. **Always use environment variables** for API keys and secrets
2. **Implement rate limiting** when crawling external sources
3. **Cache research data** to avoid redundant API calls
4. **Queue video rendering** for background processing
5. **Validate content** before auto-posting to social media
6. **Monitor API costs** especially for Claude/OpenAI usage
7. **Use TypeScript types** for better IDE support and error prevention
