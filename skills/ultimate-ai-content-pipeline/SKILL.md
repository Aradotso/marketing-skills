---
name: ultimate-ai-content-pipeline
description: Vietnamese AI content automation system with research crawling, multi-format content generation (Claude/OpenAI), and automated video rendering via Remotion
triggers:
  - "set up the AI content pipeline system"
  - "generate automated content with research crawling"
  - "create video content from articles using Remotion"
  - "configure Claude or OpenAI for content generation"
  - "automate content research and article writing"
  - "render videos from blog posts automatically"
  - "set up bilingual content generation pipeline"
  - "integrate content automation with social media"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive Vietnamese content automation system that handles the entire content creation workflow: from automated research crawling (TechCrunch, a16z, Twitter/X, LinkedIn), to AI-powered article generation in multiple formats (using Claude 3 or OpenAI), to automatic video rendering via Remotion. Built with Next.js and TypeScript.

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
cp .env.example .env.local
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Provider (choose one or both)
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Core Architecture

The pipeline consists of three main stages:

1. **Research Phase**: Automated web crawling and data extraction
2. **Content Generation Phase**: AI-powered article writing with Claude/OpenAI
3. **Media Generation Phase**: Automated video/image rendering with Remotion

## Research & Crawling

### Automated News Crawling

```typescript
// lib/research/crawler.ts
import { fetchTechCrunchNews } from './sources/techcrunch';
import { fetchTwitterTrends } from './sources/twitter';
import { fetchLinkedInPosts } from './sources/linkedin';

interface ResearchData {
  topic: string;
  sources: Array<{
    title: string;
    url: string;
    summary: string;
    publishedAt: Date;
  }>;
  insights: string[];
  dataPoints: Array<{ metric: string; value: string }>;
}

export async function conductResearch(
  keyword: string,
  timeframe: '24h' | '7d' | '30d' = '24h'
): Promise<ResearchData> {
  const [techCrunch, twitter, linkedin] = await Promise.all([
    fetchTechCrunchNews(keyword, timeframe),
    fetchTwitterTrends(keyword),
    fetchLinkedInPosts(keyword, timeframe)
  ]);

  return {
    topic: keyword,
    sources: [...techCrunch, ...twitter, ...linkedin],
    insights: extractInsights([...techCrunch, ...twitter, ...linkedin]),
    dataPoints: extractDataPoints([...techCrunch, ...twitter, ...linkedin])
  };
}
```

### Example: Fetching Research Data

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { conductResearch } from '@/lib/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, timeframe } = req.body;

  try {
    const researchData = await conductResearch(keyword, timeframe);
    return res.status(200).json(researchData);
  } catch (error) {
    console.error('Research failed:', error);
    return res.status(500).json({ error: 'Research failed' });
  }
}
```

## Content Generation with AI

### Claude Integration

```typescript
// lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'vi' | 'en';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchData;
}

export async function generateArticleWithClaude(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Viết dưới dạng danh sách xếp hạng với tiêu đề hấp dẫn',
    'pov': 'Viết dưới góc nhìn chuyên gia với quan điểm rõ ràng',
    'case-study': 'Phân tích case study chi tiết với số liệu cụ thể',
    'how-to': 'Hướng dẫn từng bước chi tiết và dễ hiểu'
  };

  const toneInstructions = {
    'expert': 'Giọng văn chuyên nghiệp, có chiều sâu',
    'friendly': 'Giọng văn thân thiện, dễ tiếp cận',
    'humorous': 'Giọng văn hài hước, vui vẻ'
  };

  return `
Bạn là một content creator chuyên nghiệp. Hãy viết một bài viết về chủ đề: ${request.topic}

Format: ${formatInstructions[request.format]}
Tone: ${toneInstructions[request.tone]}
Ngôn ngữ: ${request.language === 'vi' ? 'Tiếng Việt' : 'English'}

Dữ liệu nghiên cứu:
${JSON.stringify(request.researchData, null, 2)}

Yêu cầu:
- Sử dụng dữ liệu thực từ research
- Trích dẫn nguồn đáng tin cậy
- Đưa ra insights sâu sắc
- Tối ưu SEO với từ khóa tự nhiên
- Độ dài: 1500-2000 từ
`;
}
```

### OpenAI Integration

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export async function generateArticleWithOpenAI(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a professional content creator specializing in marketing and trending topics.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  return completion.choices[0]?.message?.content || '';
}
```

### Bilingual Content Generation

```typescript
// lib/ai/bilingual.ts
export async function generateBilingualContent(
  request: Omit<ContentRequest, 'language'>
): Promise<{ vi: string; en: string }> {
  const [vietnameseContent, englishContent] = await Promise.all([
    generateArticleWithClaude({ ...request, language: 'vi' }),
    generateArticleWithClaude({ ...request, language: 'en' })
  ]);

  return {
    vi: vietnameseContent,
    en: englishContent
  };
}
```

## Video Rendering with Remotion

### Video Composition

```typescript
// remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ArticleVideoProps {
  title: string;
  keyPoints: string[];
  dataPoints: Array<{ metric: string; value: string }>;
  brandColor: string;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  keyPoints,
  dataPoints,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          padding: 40
        }}>
          <h1 style={{ 
            color: brandColor, 
            fontSize: 60,
            textAlign: 'center',
            opacity: Math.min(1, frame / 30)
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Key Points Sequence */}
      {keyPoints.map((point, index) => (
        <Sequence 
          key={index}
          from={fps * (3 + index * 4)} 
          durationInFrames={fps * 4}
        >
          <KeyPointSlide point={point} color={brandColor} />
        </Sequence>
      ))}

      {/* Data Visualization Sequence */}
      <Sequence from={fps * (3 + keyPoints.length * 4)} durationInFrames={fps * 5}>
        <DataVisualization dataPoints={dataPoints} color={brandColor} />
      </Sequence>
    </AbsoluteFill>
  );
};
```

### Render Video Programmatically

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderArticleVideo(
  articleData: {
    title: string;
    keyPoints: string[];
    dataPoints: Array<{ metric: string; value: string }>;
  },
  outputPath: string
): Promise<string> {
  const compositionId = 'ArticleVideo';
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: articleData
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: articleData
  });

  return outputPath;
}
```

### API Route for Video Rendering

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderArticleVideo } from '@/lib/video/renderer';
import path from 'path';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { title, keyPoints, dataPoints } = req.body;

  try {
    const outputPath = path.join(
      process.cwd(), 
      'public', 
      'videos', 
      `${Date.now()}.mp4`
    );

    const videoPath = await renderArticleVideo(
      { title, keyPoints, dataPoints },
      outputPath
    );

    return res.status(200).json({ 
      videoUrl: videoPath.replace(path.join(process.cwd(), 'public'), '')
    });
  } catch (error) {
    console.error('Video rendering failed:', error);
    return res.status(500).json({ error: 'Video rendering failed' });
  }
}
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline/orchestrator.ts
import { conductResearch } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/ai/bilingual';
import { renderArticleVideo } from '@/lib/video/renderer';
import { extractKeyPoints, extractDataPoints } from '@/lib/utils/content-parser';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  brandColor?: string;
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const researchData = await conductResearch(config.keyword, '24h');

  // Step 2: Content Generation
  console.log('✍️ Generating bilingual content...');
  const content = await generateBilingualContent({
    topic: config.keyword,
    format: config.format,
    tone: config.tone,
    researchData
  });

  // Step 3: Video Rendering (optional)
  let videoUrl = null;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    const keyPoints = extractKeyPoints(content.vi);
    const dataPoints = researchData.dataPoints;

    videoUrl = await renderArticleVideo(
      {
        title: config.keyword,
        keyPoints,
        dataPoints
      },
      `public/videos/${Date.now()}.mp4`
    );
  }

  return {
    research: researchData,
    content,
    videoUrl,
    metadata: {
      generatedAt: new Date(),
      keyword: config.keyword,
      format: config.format
    }
  };
}
```

## Usage Examples

### Basic Content Generation

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          tone: 'friendly',
          generateVideo: true
        })
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Pipeline failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">
        AI Content Pipeline
      </h1>

      <div className="flex gap-4 mb-8">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Nhập từ khóa..."
          className="flex-1 px-4 py-2 border rounded"
        />
        <button
          onClick={handleGenerate}
          disabled={loading}
          className="px-6 py-2 bg-blue-600 text-white rounded"
        >
          {loading ? 'Đang xử lý...' : 'Tạo Content'}
        </button>
      </div>

      {result && (
        <div className="space-y-6">
          <div>
            <h2 className="text-2xl font-bold mb-4">Nội dung Tiếng Việt</h2>
            <div className="prose max-w-none">
              {result.content.vi}
            </div>
          </div>

          {result.videoUrl && (
            <div>
              <h2 className="text-2xl font-bold mb-4">Video</h2>
              <video src={result.videoUrl} controls className="w-full" />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

### CLI Usage (if applicable)

```bash
# Generate content with research
npm run pipeline -- --keyword "AI trends 2024" --format toplist --video

# Research only
npm run research -- --keyword "machine learning" --timeframe 7d

# Render video from existing content
npm run render -- --input content.json --output video.mp4
```

## Common Patterns

### Custom Research Sources

```typescript
// lib/research/sources/custom-source.ts
export async function fetchCustomSource(keyword: string) {
  const response = await fetch(`https://api.example.com/search?q=${keyword}`, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!
    }
  });

  const data = await response.json();
  
  return data.results.map((item: any) => ({
    title: item.title,
    url: item.link,
    summary: item.snippet,
    publishedAt: new Date(item.date)
  }));
}
```

### Content Scheduling

```typescript
// lib/scheduler/content-scheduler.ts
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function scheduleContentGeneration(
  keywords: string[],
  schedule: 'daily' | 'weekly'
) {
  const interval = schedule === 'daily' ? 24 * 60 * 60 * 1000 : 7 * 24 * 60 * 60 * 1000;

  for (const keyword of keywords) {
    setTimeout(async () => {
      const result = await runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'friendly',
        generateVideo: true
      });

      // Auto-post to social media or save to CMS
      await publishContent(result);
    }, interval);
  }
}
```

## Troubleshooting

### Common Issues

**API Rate Limits**:
```typescript
// lib/utils/rate-limiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  retries = 3
): Promise<T> {
  try {
    return await fn();
  } catch (error: any) {
    if (error.status === 429 && retries > 0) {
      await new Promise(resolve => setTimeout(resolve, 5000));
      return withRateLimit(fn, retries - 1);
    }
    throw error;
  }
}
```

**Video Rendering Memory Issues**:
```typescript
// Reduce video quality for large batches
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: compositionId,
  inputProps: {
    ...articleData,
    quality: 'medium' // instead of 'high'
  }
});
```

**Research Data Empty**:
- Check API keys are valid and have credits
- Verify network connectivity to research sources
- Ensure keyword is not too niche (try broader terms)

**Claude/OpenAI Timeouts**:
```typescript
// Implement timeout handling
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 30000);

try {
  const response = await anthropic.messages.create({
    // ... config
  }, { signal: controller.signal });
} finally {
  clearTimeout(timeoutId);
}
```

## Performance Optimization

### Parallel Processing

```typescript
// Process multiple keywords simultaneously
const results = await Promise.allSettled(
  keywords.map(keyword => 
    runContentPipeline({ 
      keyword, 
      format: 'toplist', 
      tone: 'friendly',
      generateVideo: false 
    })
  )
);
```

### Caching Research Data

```typescript
// lib/cache/research-cache.ts
const cache = new Map<string, { data: ResearchData; timestamp: number }>();
const CACHE_TTL = 3600000; // 1 hour

export function getCachedResearch(keyword: string): ResearchData | null {
  const cached = cache.get(keyword);
  if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
    return cached.data;
  }
  return null;
}

export function setCachedResearch(keyword: string, data: ResearchData) {
  cache.set(keyword, { data, timestamp: Date.now() });
}
```
