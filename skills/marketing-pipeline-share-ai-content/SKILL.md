---
name: marketing-pipeline-share-ai-content
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - create automated marketing content with this pipeline
  - generate video content from text automatically
  - research and write blog posts with AI
  - configure the content automation system
  - use remotion to render marketing videos
  - crawl news sources for content research
  - build an AI content creation workflow
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content pipeline that automates research, script writing, and video generation for marketing content. It combines web scraping, AI writing (Claude/OpenAI), and video rendering (Remotion) into a single automated workflow.

## What It Does

The Marketing Pipeline Share system automates the entire content creation process:

1. **Auto-Research**: Crawls news sources (TechCrunch, Twitter, LinkedIn) for latest trends and data
2. **AI Writing**: Generates multilingual content (English/Vietnamese) in various formats using Claude 3 or OpenAI
3. **Video Generation**: Automatically renders videos and infographics using Remotion
4. **Multi-Format Output**: Creates content optimized for blogs, Reels, TikTok, and YouTube Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Setup

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Core Architecture

The pipeline follows this flow:

```
User Input (Keyword) → Research Module → AI Writing Engine → Content Formatter → Video Renderer → Output
```

## Key Modules & Usage

### 1. Research Module (Web Scraping)

The research module crawls sources for fresh data:

```typescript
// src/services/research.ts
import { RapidAPI } from '@/lib/rapidapi';

interface ResearchOptions {
  keyword: string;
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

export async function conductResearch(options: ResearchOptions) {
  const { keyword, sources, timeRange } = options;
  
  const results = await Promise.all(
    sources.map(source => 
      RapidAPI.search({
        query: keyword,
        source,
        timeRange
      })
    )
  );
  
  return {
    articles: results.flatMap(r => r.articles),
    insights: extractInsights(results),
    stats: compileStatistics(results)
  };
}

// Usage
const research = await conductResearch({
  keyword: 'AI marketing trends',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeRange: '24h'
});
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// src/services/ai-writer.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentOptions {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

export async function generateContent(options: ContentOptions, provider: 'claude' | 'openai' = 'claude') {
  const prompt = buildPrompt(options);
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt
        }
      ],
    });
    
    return message.content[0].text;
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content writer specializing in marketing.'
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      max_tokens: 4096,
    });
    
    return completion.choices[0].message.content;
  }
}

function buildPrompt(options: ContentOptions): string {
  const { topic, format, tone, language, researchData } = options;
  
  return `
Write a ${format} article about "${topic}" in ${language === 'en' ? 'English' : 'Vietnamese'}.
Tone: ${tone}

Use the following research data:
${JSON.stringify(researchData, null, 2)}

Include:
- Catchy headline
- Introduction hook
- Main content with data-backed insights
- Conclusion with CTA
- SEO-optimized structure
`;
}
```

### 3. Video Rendering with Remotion

Create videos from content:

```typescript
// src/remotion/compositions.tsx
import { Composition } from 'remotion';
import { VideoTemplate } from './VideoTemplate';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={VideoTemplate}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920} // Vertical for Reels/TikTok
        defaultProps={{
          title: '',
          highlights: [],
          bgColor: '#1a1a1a',
        }}
      />
    </>
  );
};
```

```typescript
// src/remotion/VideoTemplate.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface VideoProps {
  title: string;
  highlights: string[];
  bgColor: string;
}

export const VideoTemplate: React.FC<VideoProps> = ({ title, highlights, bgColor }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        padding: 60, 
        color: 'white',
        fontSize: 48,
        fontWeight: 'bold',
        opacity: titleOpacity 
      }}>
        {title}
      </div>
      
      {highlights.map((highlight, index) => {
        const startFrame = 60 + (index * 60);
        const opacity = interpolate(
          frame,
          [startFrame, startFrame + 30],
          [0, 1],
          { extrapolateRight: 'clamp' }
        );
        
        return (
          <div
            key={index}
            style={{
              padding: 60,
              marginTop: 100 + (index * 80),
              color: 'white',
              fontSize: 32,
              opacity,
            }}
          >
            • {highlight}
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```

```typescript
// src/services/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(content: {
  title: string;
  highlights: string[];
}) {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });
  
  const outputPath = path.join(process.cwd(), 'public/videos', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content,
  });
  
  return outputPath;
}
```

### 4. Complete Pipeline Example

```typescript
// src/pages/api/pipeline.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { conductResearch } from '@/services/research';
import { generateContent } from '@/services/ai-writer';
import { renderContentVideo } from '@/services/video-renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, language } = req.body;
  
  try {
    // Step 1: Research
    const researchData = await conductResearch({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h',
      language,
    });
    
    // Step 2: Generate Content
    const content = await generateContent({
      topic: keyword,
      format,
      tone: 'expert',
      language,
      researchData,
    });
    
    // Step 3: Extract video highlights
    const highlights = extractHighlights(content);
    
    // Step 4: Render Video
    const videoPath = await renderContentVideo({
      title: keyword,
      highlights,
    });
    
    res.status(200).json({
      success: true,
      content,
      videoUrl: videoPath.replace(process.cwd() + '/public', ''),
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ error: 'Pipeline failed' });
  }
}

function extractHighlights(content: string): string[] {
  // Extract key points from content
  const lines = content.split('\n');
  return lines
    .filter(line => line.trim().startsWith('-') || line.trim().startsWith('•'))
    .map(line => line.replace(/^[-•]\s*/, '').trim())
    .slice(0, 5);
}
```

### 5. Frontend Integration

```typescript
// src/pages/index.tsx
import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language: 'vi',
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
    <div style={{ padding: '40px', maxWidth: '800px', margin: '0 auto' }}>
      <h1>AI Content Pipeline</h1>
      
      <input
        type="text"
        placeholder="Enter keyword..."
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        style={{ width: '100%', padding: '12px', marginBottom: '12px' }}
      />
      
      <select
        value={format}
        onChange={(e) => setFormat(e.target.value as any)}
        style={{ width: '100%', padding: '12px', marginBottom: '12px' }}
      >
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-to Guide</option>
      </select>
      
      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        style={{ width: '100%', padding: '12px', cursor: 'pointer' }}
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div style={{ marginTop: '24px' }}>
          <h2>Generated Content</h2>
          <pre style={{ whiteSpace: 'pre-wrap' }}>{result.content}</pre>
          
          {result.videoUrl && (
            <div>
              <h3>Video</h3>
              <video src={result.videoUrl} controls style={{ width: '100%' }} />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Custom AI Prompts

Create custom prompt templates:

```typescript
// src/config/prompts.ts
export const PROMPT_TEMPLATES = {
  toplist: {
    en: `Create a comprehensive top list article about {topic}. Include numbered items with detailed explanations.`,
    vi: `Tạo bài viết dạng top list về {topic}. Bao gồm các mục được đánh số với giải thích chi tiết.`,
  },
  pov: {
    en: `Write a thought-provoking POV article about {topic} from an expert perspective.`,
    vi: `Viết bài phân tích quan điểm về {topic} từ góc nhìn chuyên gia.`,
  },
  // Add more templates
};
```

### Video Styling

Customize video templates:

```typescript
// src/config/video-styles.ts
export const VIDEO_THEMES = {
  dark: {
    bgColor: '#1a1a1a',
    textColor: '#ffffff',
    accentColor: '#ff6b6b',
  },
  light: {
    bgColor: '#ffffff',
    textColor: '#1a1a1a',
    accentColor: '#4dabf7',
  },
  brand: {
    bgColor: '#2d3748',
    textColor: '#e2e8f0',
    accentColor: '#48bb78',
  },
};
```

## Common Workflows

### Batch Content Generation

```typescript
// src/scripts/batch-generate.ts
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    const research = await conductResearch({
      keyword,
      sources: ['techcrunch'],
      timeRange: '24h',
    });
    
    const content = await generateContent({
      topic: keyword,
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData: research,
    });
    
    results.push({ keyword, content });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Scheduled Research

```typescript
// src/services/scheduler.ts
import cron from 'node-cron';

// Run research daily at 8 AM
cron.schedule('0 8 * * *', async () => {
  const trendingTopics = await getTrendingTopics();
  
  for (const topic of trendingTopics) {
    await conductResearch({
      keyword: topic,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h',
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Reduce quality for large batches
const renderOptions = {
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  inputProps: content,
  imageFormat: 'jpeg',
  jpegQuality: 80, // Reduce from default 100
  scale: 0.8, // Reduce resolution
};
```

### Environment Variable Loading

```typescript
// src/lib/config.ts
function requireEnv(key: string): string {
  const value = process.env[key];
  if (!value) {
    throw new Error(`Missing required environment variable: ${key}`);
  }
  return value;
}

export const config = {
  anthropic: requireEnv('ANTHROPIC_API_KEY'),
  openai: requireEnv('OPENAI_API_KEY'),
  rapidapi: requireEnv('RAPIDAPI_KEY'),
};
```

## Build & Deploy

```bash
# Build for production
npm run build

# Start production server
npm run start

# Render videos in production
npm run remotion:render
```

This AI content pipeline automates the entire marketing content creation process, from research through video generation, enabling teams to scale content production dramatically while maintaining quality and relevance.
