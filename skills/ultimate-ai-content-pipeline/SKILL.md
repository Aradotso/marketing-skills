---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up AI content pipeline for marketing
  - create automated video content from text
  - generate marketing content with Claude and OpenAI
  - crawl news and generate content automatically
  - use Remotion to create videos from articles
  - build content automation workflow
  - integrate AI for content research and generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive content automation system that handles the entire content creation workflow: from researching trending topics, generating multilingual content in various formats (toplist, POV, case study, how-to), to automatically rendering videos and infographics using Remotion. Built with Next.js and TypeScript, it integrates Claude 3, OpenAI, and news crawling APIs to create a complete content factory.

## What It Does

- **Auto-Research**: Crawls TechCrunch, a16z, Twitter, LinkedIn for fresh data within 24h
- **AI Content Generation**: Creates content in multiple formats and languages (EN/VN) using Claude/OpenAI
- **Video Rendering**: Automatically converts written content to videos/infographics via Remotion
- **Multi-Platform Export**: Outputs optimized videos for Reels, TikTok, Shorts

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
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# News/Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News crawling modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video compositions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
// src/lib/crawlers/news-crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}

export async function crawlTrendingNews(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<NewsArticle[]> {
  const rapidApi = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    try {
      const results = await rapidApi.searchNews({
        query: keyword,
        source,
        timeframe,
        language: 'en'
      });
      articles.push(...results);
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
    }
  }
  
  return articles;
}

// Extract insights from crawled data
export async function extractInsights(
  articles: NewsArticle[]
): Promise<{ insights: string[]; stats: Record<string, number> }> {
  const content = articles.map(a => a.content).join('\n\n');
  
  // Use AI to analyze and extract key insights
  const insights = await analyzeWithAI(content);
  
  return {
    insights,
    stats: {
      totalArticles: articles.length,
      sources: new Set(articles.map(a => a.source)).size
    }
  };
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: string;
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateContent(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);
    
    // Use Claude for content generation
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  private buildPrompt(request: ContentRequest): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article highlighting top items/tips',
      'pov': 'Write from a specific perspective or opinion piece',
      'case-study': 'Analyze a real-world example with data and outcomes',
      'how-to': 'Step-by-step instructional guide'
    };
    
    return `
You are an expert content writer. Create a ${request.format} article in ${request.language}.

Topic: ${request.keyword}
Tone: ${request.tone}
Format: ${formatInstructions[request.format]}

Research Data:
${request.researchData}

Requirements:
- Use data and insights from the research provided
- Write in ${request.language === 'vi' ? 'Vietnamese' : 'English'}
- Maintain a ${request.tone} tone throughout
- Include specific examples and statistics
- Make it engaging and actionable

Generate the complete article now:
    `.trim();
  }
  
  // Generate content in both languages simultaneously
  async generateBilingual(
    request: Omit<ContentRequest, 'language'>
  ): Promise<{ en: string; vi: string }> {
    const [en, vi] = await Promise.all([
      this.generateContent({ ...request, language: 'en' }),
      this.generateContent({ ...request, language: 'vi' })
    ]);
    
    return { en, vi };
  }
}
```

### 3. Video Rendering with Remotion

```typescript
// src/remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ArticleVideoProps {
  title: string;
  keyPoints: string[];
  stats: Array<{ label: string; value: string }>;
  brandColor: string;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  keyPoints,
  stats,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Section */}
      <div style={{
        opacity: titleOpacity,
        padding: '60px',
        color: '#fff',
        fontSize: '48px',
        fontWeight: 'bold'
      }}>
        {title}
      </div>
      
      {/* Key Points Animation */}
      <div style={{ padding: '60px', paddingTop: '200px' }}>
        {keyPoints.map((point, idx) => {
          const pointOpacity = interpolate(
            frame,
            [60 + idx * 30, 90 + idx * 30],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <div
              key={idx}
              style={{
                opacity: pointOpacity,
                color: brandColor,
                fontSize: '32px',
                marginBottom: '20px'
              }}
            >
              • {point}
            </div>
          );
        })}
      </div>
      
      {/* Stats Section */}
      {frame > 180 && (
        <div style={{
          position: 'absolute',
          bottom: '60px',
          left: '60px',
          right: '60px',
          display: 'flex',
          justifyContent: 'space-around'
        }}>
          {stats.map((stat, idx) => (
            <div key={idx} style={{ textAlign: 'center' }}>
              <div style={{ fontSize: '48px', color: brandColor }}>
                {stat.value}
              </div>
              <div style={{ fontSize: '24px', color: '#fff' }}>
                {stat.label}
              </div>
            </div>
          ))}
        </div>
      )}
    </AbsoluteFill>
  );
};
```

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderArticleVideo(
  articleData: {
    title: string;
    keyPoints: string[];
    stats: Array<{ label: string; value: string }>;
  },
  outputPath: string
): Promise<string> {
  const compositionId = 'ArticleVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: articleData
  });
  
  // Render video
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

## Complete Content Pipeline Workflow

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlTrendingNews, extractInsights } from '@/lib/crawlers/news-crawler';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { renderArticleVideo } from '@/lib/video/renderer';
import path from 'path';

export async function runContentPipeline(keyword: string) {
  console.log(`Starting content pipeline for: ${keyword}`);
  
  // Step 1: Research
  console.log('Step 1: Crawling news...');
  const articles = await crawlTrendingNews(keyword, '24h');
  const { insights, stats } = await extractInsights(articles);
  
  const researchData = `
Insights from ${stats.totalArticles} articles across ${stats.sources} sources:
${insights.join('\n')}
  `;
  
  // Step 2: Generate Content
  console.log('Step 2: Generating content...');
  const generator = new ContentGenerator();
  
  const content = await generator.generateBilingual({
    keyword,
    format: 'toplist',
    tone: 'expert',
    researchData
  });
  
  // Step 3: Render Video
  console.log('Step 3: Rendering video...');
  const videoData = {
    title: `Top Trends: ${keyword}`,
    keyPoints: insights.slice(0, 5),
    stats: [
      { label: 'Articles Analyzed', value: stats.totalArticles.toString() },
      { label: 'Sources', value: stats.sources.toString() }
    ]
  };
  
  const outputPath = path.join(
    process.cwd(), 
    'public/videos', 
    `${keyword.replace(/\s+/g, '-')}.mp4`
  );
  
  const videoUrl = await renderArticleVideo(videoData, outputPath);
  
  return {
    content: {
      english: content.en,
      vietnamese: content.vi
    },
    video: videoUrl,
    metadata: {
      keyword,
      articlesAnalyzed: stats.totalArticles,
      sources: stats.sources,
      generatedAt: new Date().toISOString()
    }
  };
}
```

## Next.js API Routes

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Failed to generate:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="mb-6">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI marketing trends)"
          className="w-full p-3 border rounded"
        />
      </div>
      
      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-600 text-white px-6 py-3 rounded disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-8 space-y-6">
          <div>
            <h2 className="text-xl font-bold mb-2">English Content</h2>
            <div className="bg-gray-100 p-4 rounded whitespace-pre-wrap">
              {result.content.english}
            </div>
          </div>
          
          <div>
            <h2 className="text-xl font-bold mb-2">Vietnamese Content</h2>
            <div className="bg-gray-100 p-4 rounded whitespace-pre-wrap">
              {result.content.vietnamese}
            </div>
          </div>
          
          {result.video && (
            <div>
              <h2 className="text-xl font-bold mb-2">Generated Video</h2>
              <video controls className="w-full">
                <source src={result.video} type="video/mp4" />
              </video>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Running the Application

```bash
# Development
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video only (if needed)
npm run remotion:render
```

## Common Patterns

### Custom Content Formats

```typescript
// Add custom format in src/lib/ai/formats/
export const customFormats = {
  'product-review': {
    structure: ['intro', 'features', 'pros-cons', 'verdict'],
    tone: 'balanced',
    minLength: 1000
  },
  'news-summary': {
    structure: ['headline', 'key-facts', 'analysis', 'impact'],
    tone: 'neutral',
    minLength: 500
  }
};
```

### Multi-Platform Video Export

```typescript
// src/lib/video/export-formats.ts
export const platformFormats = {
  'tiktok': { width: 1080, height: 1920, fps: 30 },
  'instagram-reels': { width: 1080, height: 1920, fps: 30 },
  'youtube-shorts': { width: 1080, height: 1920, fps: 30 },
  'linkedin': { width: 1280, height: 720, fps: 30 }
};

export async function renderForPlatform(
  composition: any,
  platform: keyof typeof platformFormats
) {
  const format = platformFormats[platform];
  // Render with platform-specific dimensions
}
```

## Troubleshooting

**API Rate Limits**: Implement request queuing and caching for crawled data
```typescript
// Use rate limiting middleware
import { Ratelimit } from '@upstash/ratelimit';
```

**Video Rendering Memory Issues**: Reduce composition complexity or increase Node memory
```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

**AI Response Quality**: Adjust temperature and max_tokens in AI client configuration
```typescript
messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 8000,
  temperature: 0.7
});
```

**Multilingual Content**: Ensure prompts specify language clearly and provide examples in target language
