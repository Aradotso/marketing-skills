---
name: marketing-pipeline-share-ai-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, video generation, and multi-platform publishing
triggers:
  - automate content creation with AI
  - generate video content from text
  - crawl news and create articles automatically
  - build content pipeline with Claude and OpenAI
  - create multilingual marketing content
  - render videos from blog posts
  - automate social media content generation
  - research and generate content from trending topics
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based system automates the entire content creation workflow from research to video generation. It crawls news sources (TechCrunch, Twitter, LinkedIn), generates articles in multiple formats and languages using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes content from major tech news sources (last 24h)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to)
- **Multilingual Support**: Generates content in English and Vietnamese simultaneously
- **Video Generation**: Automatically renders infographics and short videos from text
- **Multi-Platform**: Exports content optimized for Reels, TikTok, Shorts

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

### Environment Setup

Create a `.env.local` file in the project root:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Visit `http://localhost:3000` to access the application.

## Core Modules

### 1. Research/Crawling Module

The research module fetches trending content from multiple sources:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  articles: Article[];
}

interface Article {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
  source: string;
}

export async function crawlTechCrunch(keyword: string): Promise<Article[]> {
  const response = await axios.get('https://techcrunch.com/wp-json/wp/v2/posts', {
    params: {
      search: keyword,
      per_page: 10,
      _embed: true
    }
  });
  
  return response.data.map((post: any) => ({
    title: post.title.rendered,
    url: post.link,
    publishedAt: post.date,
    content: post.excerpt.rendered,
    source: 'TechCrunch'
  }));
}

export async function crawlTwitter(keyword: string): Promise<Article[]> {
  const options = {
    method: 'GET',
    url: 'https://twitter-api45.p.rapidapi.com/search.php',
    params: { query: keyword, search_type: 'Latest' },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'twitter-api45.p.rapidapi.com'
    }
  };

  const response = await axios.request(options);
  return response.data.timeline.map((tweet: any) => ({
    title: tweet.text.substring(0, 100),
    url: `https://twitter.com/user/status/${tweet.tweet_id}`,
    publishedAt: tweet.created_at,
    content: tweet.text,
    source: 'Twitter'
  }));
}

export async function aggregateResearch(keyword: string): Promise<NewsSource[]> {
  const [techCrunch, twitter] = await Promise.all([
    crawlTechCrunch(keyword),
    crawlTwitter(keyword)
  ]);

  return [
    { name: 'TechCrunch', url: 'https://techcrunch.com', articles: techCrunch },
    { name: 'Twitter', url: 'https://twitter.com', articles: twitter }
  ];
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

interface GeneratedContent {
  title: string;
  content: string;
  summary: string;
  tags: string[];
}

export async function generateWithClaude(
  options: ContentOptions
): Promise<GeneratedContent> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(options);

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

  const response = message.content[0].text;
  return parseAIResponse(response);
}

export async function generateWithOpenAI(
  options: ContentOptions
): Promise<GeneratedContent> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const prompt = buildPrompt(options);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and tech content.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  const response = completion.choices[0].message.content || '';
  return parseAIResponse(response);
}

function buildPrompt(options: ContentOptions): string {
  const formatInstructions = {
    'toplist': 'Create a top 10 list format',
    'pov': 'Write from a personal point of view perspective',
    'case-study': 'Structure as a detailed case study with data',
    'how-to': 'Write as a step-by-step tutorial'
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative tone',
    'friendly': 'Use conversational, approachable tone',
    'humorous': 'Use light humor and engaging tone'
  };

  return `
Create a ${options.language === 'vi' ? 'Vietnamese' : 'English'} article about "${options.keyword}".

Format: ${formatInstructions[options.format]}
Tone: ${toneInstructions[options.tone]}

Research Data:
${options.researchData}

Requirements:
- Title: Compelling and SEO-optimized
- Content: 1500-2000 words, well-structured with headings
- Include data and insights from the research
- Add 5-7 relevant tags
- Provide a 2-sentence summary

Return JSON format:
{
  "title": "...",
  "content": "...",
  "summary": "...",
  "tags": ["tag1", "tag2", ...]
}
`;
}

function parseAIResponse(response: string): GeneratedContent {
  // Extract JSON from response
  const jsonMatch = response.match(/\{[\s\S]*\}/);
  if (jsonMatch) {
    return JSON.parse(jsonMatch[0]);
  }
  
  // Fallback parsing if not JSON
  return {
    title: 'Generated Article',
    content: response,
    summary: response.substring(0, 200),
    tags: []
  };
}
```

### 3. Video Generation with Remotion

Render videos from generated content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  style: 'minimal' | 'colorful' | 'corporate';
  duration: number; // in seconds
  platform: 'reels' | 'tiktok' | 'shorts';
}

const PLATFORM_SPECS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const specs = PLATFORM_SPECS[config.platform];
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style
    },
    overwrite: true,
  });

  return outputPath;
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  style: 'minimal' | 'colorful' | 'corporate';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  style
}) => {
  const frame = useCurrentFrame();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const contentOpacity = interpolate(frame, [30, 60], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const styles = {
    minimal: { bg: '#ffffff', text: '#000000' },
    colorful: { bg: '#4F46E5', text: '#ffffff' },
    corporate: { bg: '#1F2937', text: '#F3F4F6' }
  };

  const colors = styles[style];

  return (
    <AbsoluteFill style={{ backgroundColor: colors.bg }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 80,
          }}
        >
          <h1
            style={{
              fontSize: 72,
              fontWeight: 'bold',
              color: colors.text,
              textAlign: 'center',
              opacity: titleOpacity,
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      <Sequence from={90} durationInFrames={210}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 80,
          }}
        >
          <p
            style={{
              fontSize: 36,
              color: colors.text,
              textAlign: 'center',
              lineHeight: 1.6,
              opacity: contentOpacity,
            }}
          >
            {content.substring(0, 300)}...
          </p>
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { aggregateResearch } from '@/lib/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword } = req.body;

  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }

  try {
    const research = await aggregateResearch(keyword);
    res.status(200).json({ success: true, data: research });
  } catch (error) {
    console.error('Research error:', error);
    res.status(500).json({ error: 'Failed to fetch research data' });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateWithClaude, generateWithOpenAI } from '@/lib/ai/content-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, language, tone, researchData, provider } = req.body;

  try {
    const options = {
      keyword,
      format: format || 'how-to',
      language: language || 'en',
      tone: tone || 'expert',
      researchData: JSON.stringify(researchData)
    };

    const content = provider === 'openai'
      ? await generateWithOpenAI(options)
      : await generateWithClaude(options);

    res.status(200).json({ success: true, content });
  } catch (error) {
    console.error('Generation error:', error);
    res.status(500).json({ error: 'Failed to generate content' });
  }
}
```

### Video Rendering Endpoint

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderContentVideo } from '@/lib/video/renderer';
import path from 'path';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { title, content, style, platform } = req.body;

  try {
    const outputPath = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);
    
    await renderContentVideo(
      {
        title,
        content,
        style: style || 'minimal',
        duration: 10,
        platform: platform || 'reels'
      },
      outputPath
    );

    const videoUrl = `/videos/${path.basename(outputPath)}`;
    res.status(200).json({ success: true, videoUrl });
  } catch (error) {
    console.error('Render error:', error);
    res.status(500).json({ error: 'Failed to render video' });
  }
}
```

## Complete Workflow Example

```typescript
// lib/pipeline/complete-workflow.ts
import { aggregateResearch } from '@/lib/research/crawler';
import { generateWithClaude } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineResult {
  article: {
    title: string;
    content: string;
    summary: string;
    tags: string[];
  };
  video: {
    url: string;
    path: string;
  };
  research: any[];
}

export async function runCompletePipeline(
  keyword: string,
  options?: {
    format?: 'toplist' | 'pov' | 'case-study' | 'how-to';
    language?: 'en' | 'vi';
    tone?: 'expert' | 'friendly' | 'humorous';
    videoStyle?: 'minimal' | 'colorful' | 'corporate';
    platform?: 'reels' | 'tiktok' | 'shorts';
  }
): Promise<PipelineResult> {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await aggregateResearch(keyword);
  
  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const article = await generateWithClaude({
    keyword,
    format: options?.format || 'how-to',
    language: options?.language || 'en',
    tone: options?.tone || 'expert',
    researchData: JSON.stringify(research)
  });

  // Step 3: Render video
  console.log('🎬 Rendering video...');
  const videoPath = `public/videos/${Date.now()}.mp4`;
  await renderContentVideo(
    {
      title: article.title,
      content: article.summary,
      style: options?.videoStyle || 'minimal',
      duration: 10,
      platform: options?.platform || 'reels'
    },
    videoPath
  );

  console.log('✅ Pipeline complete!');

  return {
    article,
    video: {
      url: `/videos/${videoPath.split('/').pop()}`,
      path: videoPath
    },
    research
  };
}
```

## Usage in Frontend

```typescript
// components/ContentPipeline.tsx
import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const runPipeline = async () => {
    setLoading(true);
    try {
      // Step 1: Research
      const researchRes = await fetch('/api/research', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword })
      });
      const { data: research } = await researchRes.json();

      // Step 2: Generate content
      const contentRes = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'how-to',
          language: 'en',
          tone: 'expert',
          researchData: research,
          provider: 'claude'
        })
      });
      const { content } = await contentRes.json();

      // Step 3: Render video
      const videoRes = await fetch('/api/render-video', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          title: content.title,
          content: content.summary,
          style: 'minimal',
          platform: 'reels'
        })
      });
      const { videoUrl } = await videoRes.json();

      setResult({ content, videoUrl, research });
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="mb-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
      </div>

      <button
        onClick={runPipeline}
        disabled={loading || !keyword}
        className="bg-blue-500 text-white px-6 py-3 rounded disabled:opacity-50"
      >
        {loading ? 'Processing...' : 'Run Pipeline'}
      </button>

      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">{result.content.title}</h2>
          <p className="mb-4">{result.content.summary}</p>
          
          <video src={result.videoUrl} controls className="w-full max-w-md" />
          
          <div className="mt-4">
            <h3 className="font-bold">Tags:</h3>
            <div className="flex gap-2 mt-2">
              {result.content.tags.map((tag: string) => (
                <span key={tag} className="bg-gray-200 px-3 py-1 rounded">
                  {tag}
                </span>
              ))}
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
```

## Configuration

### Remotion Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(2);
Config.setCodec('h264');
```

### Webpack Config for Next.js

```javascript
// next.config.js
module.exports = {
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.resolve.fallback = {
        ...config.resolve.fallback,
        fs: false,
        net: false,
        tls: false,
      };
    }
    return config;
  },
  images: {
    domains: ['techcrunch.com', 'twitter.com'],
  },
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await aggregateResearch(keyword);
      const content = await generateWithClaude({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        researchData: JSON.stringify(research)
      });
      return { keyword, content };
    })
  );
  
  return results;
}
```

### Scheduled Content Pipeline

```typescript
// Using node-cron for scheduling
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI marketing', 'content automation', 'video marketing'];
  for (const keyword of keywords) {
    await runCompletePipeline(keyword);
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import Bottleneck from 'bottleneck';

const limiter = new Bottleneck({
  minTime: 1000, // 1 request per second
  maxConcurrent: 1
});

export const rateLimitedCrawl = limiter.wrap(crawlTechCrunch);
```

### Video Rendering Memory Issues

Increase Node.js memory limit:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Claude API Errors

Always handle rate limits and token limits:

```typescript
try {
  const content = await generateWithClaude(options);
} catch (error: any) {
  if (error.status === 429) {
    // Rate limited - retry with exponential backoff
    await new Promise(resolve => setTimeout(resolve, 5000));
    return generateWithClaude(options);
  }
  throw error;
}
```

### Missing Environment Variables

```typescript
// lib/config/validate-env.ts
export function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}
```

## Performance Tips

1. **Cache research data** to avoid redundant API calls
2. **Use streaming responses** for large content generation
3. **Render videos asynchronously** and notify users when complete
4. **Implement job queues** (Bull, BullMQ) for heavy processing
5. **Use CDN** for serving generated videos

## Resources

- [Remotion Documentation](https://www.remotion.dev/docs/)
- [Anthropic Claude API](https://docs.anthropic.com/)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [Next.js API Routes](https://nextjs.org/docs/api-routes/introduction)
