---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - create automated content pipeline with AI
  - generate social media content from research automatically
  - build AI content generation workflow
  - automate content research and video creation
  - set up AI-powered marketing content system
  - create multilingual content with Claude and OpenAI
  - build content automation pipeline with video rendering
  - generate blog posts and videos from keywords automatically
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end AI content automation system that transforms keywords into research, scripts, articles, and videos. The pipeline crawls news sources (TechCrunch, Twitter, LinkedIn), generates content in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

## What This Project Does

This TypeScript/Next.js system automates the entire content creation workflow:

1. **Research Phase**: Crawls recent articles from major tech/business sources
2. **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using AI
3. **Multilingual Output**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates videos and infographics from content using Remotion
5. **Multi-platform Export**: Optimized video formats for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Video rendering settings
REMOTION_CONCURRENCY=4
REMOTION_OUTPUT_DIR=./output/videos

# Next.js settings
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Research endpoints
│   │   ├── generate/      # Content generation
│   │   └── render/        # Video rendering
│   └── page.tsx           # Main UI
├── lib/                   # Core libraries
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── crawler/          # News crawling logic
│   ├── content/          # Content formatting
│   └── video/            # Remotion video generation
├── components/           # React components
└── remotion/            # Remotion video templates
```

## Key API Routes

### 1. Research Endpoint

Crawls and analyzes recent news on a keyword:

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNewsSources } from '@/lib/crawler';

export async function POST(req: NextRequest) {
  const { keyword, timeframe = '24h' } = await req.json();
  
  const research = await crawlNewsSources({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe
  });
  
  return NextResponse.json({ research });
}
```

**Usage:**

```typescript
const response = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ 
    keyword: 'AI automation',
    timeframe: '24h'
  })
});

const { research } = await response.json();
```

### 2. Content Generation Endpoint

Generates content using AI based on research:

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/generate';

export async function POST(req: NextRequest) {
  const { 
    research, 
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language, // 'en' | 'vi'
    tone // 'expert' | 'friendly' | 'humorous'
  } = await req.json();
  
  const content = await generateContent({
    research,
    format,
    language,
    tone,
    provider: 'claude' // or 'openai'
  });
  
  return NextResponse.json({ content });
}
```

### 3. Video Rendering Endpoint

Renders video from content:

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/render';

export async function POST(req: NextRequest) {
  const { 
    content, 
    platform, // 'reels' | 'tiktok' | 'shorts'
    template 
  } = await req.json();
  
  const videoUrl = await renderVideo({
    content,
    platform,
    template
  });
  
  return NextResponse.json({ videoUrl });
}
```

## Core Library Usage

### AI Content Generation

```typescript
// lib/ai/generate.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface GenerateOptions {
  research: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  provider: 'claude' | 'openai';
}

export async function generateContent(options: GenerateOptions) {
  const { research, format, language, tone, provider } = options;
  
  const prompt = buildPrompt(research, format, language, tone);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].text;
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content;
  }
}

function buildPrompt(
  research: string, 
  format: string, 
  language: string, 
  tone: string
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with rankings and explanations',
    'pov': 'Write from a personal perspective with opinions and insights',
    'case-study': 'Analyze a real example with data and outcomes',
    'how-to': 'Provide step-by-step instructions to achieve a goal'
  };
  
  return `
You are a professional content writer. Based on the following research data, 
create a ${format} article in ${language} with a ${tone} tone.

${formatInstructions[format]}

Research Data:
${research}

Requirements:
- ${language === 'en' ? 'Write in English' : 'Viết bằng Tiếng Việt'}
- Use data and statistics from the research
- Include relevant examples
- Optimize for SEO
- Length: 1500-2000 words
`;
}
```

### News Crawler Implementation

```typescript
// lib/crawler/sources.ts
import axios from 'axios';

interface CrawlOptions {
  keyword: string;
  sources: string[];
  timeframe: string;
}

export async function crawlNewsSources(options: CrawlOptions) {
  const { keyword, sources, timeframe } = options;
  const results: any[] = [];
  
  // TechCrunch crawler
  if (sources.includes('techcrunch')) {
    const techcrunchData = await crawlTechCrunch(keyword, timeframe);
    results.push(...techcrunchData);
  }
  
  // Twitter/X crawler via RapidAPI
  if (sources.includes('twitter')) {
    const twitterData = await crawlTwitter(keyword, timeframe);
    results.push(...twitterData);
  }
  
  // LinkedIn crawler
  if (sources.includes('linkedin')) {
    const linkedinData = await crawlLinkedIn(keyword, timeframe);
    results.push(...linkedinData);
  }
  
  return analyzeResearch(results);
}

async function crawlTwitter(keyword: string, timeframe: string) {
  const response = await axios.get(
    'https://twitter154.p.rapidapi.com/search/search',
    {
      params: { query: keyword, section: 'top', limit: 20 },
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        'X-RapidAPI-Host': 'twitter154.p.rapidapi.com'
      }
    }
  );
  
  return response.data.results.map((tweet: any) => ({
    source: 'twitter',
    title: tweet.text.substring(0, 100),
    content: tweet.text,
    url: `https://twitter.com/i/status/${tweet.id}`,
    date: tweet.created_at,
    metrics: {
      likes: tweet.favorite_count,
      retweets: tweet.retweet_count
    }
  }));
}

function analyzeResearch(results: any[]) {
  // Extract key insights, trends, and statistics
  const insights = {
    totalArticles: results.length,
    sources: [...new Set(results.map(r => r.source))],
    keyTopics: extractKeyTopics(results),
    trendingData: extractTrendingData(results),
    topArticles: results
      .sort((a, b) => (b.metrics?.likes || 0) - (a.metrics?.likes || 0))
      .slice(0, 10)
  };
  
  return insights;
}
```

### Remotion Video Rendering

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderOptions {
  content: any;
  platform: 'reels' | 'tiktok' | 'shorts';
  template: string;
}

const PLATFORM_SPECS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

export async function renderVideo(options: RenderOptions) {
  const { content, platform, template } = options;
  const specs = PLATFORM_SPECS[platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: template,
    inputProps: { content, ...specs }
  });
  
  // Render video
  const outputPath = path.join(
    process.env.REMOTION_OUTPUT_DIR || './output',
    `${Date.now()}-${platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: { content, ...specs }
  });
  
  return outputPath;
}
```

### Remotion Video Component

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  content: {
    title: string;
    points: string[];
    stats: Array<{ label: string; value: string }>;
  };
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title sequence (0-90 frames / 3 seconds) */}
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60
          }}
        >
          <h1
            style={{
              fontSize: 80,
              color: 'white',
              textAlign: 'center',
              fontWeight: 'bold',
              opacity: Math.min(1, frame / 30)
            }}
          >
            {content.title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Content points */}
      {content.points.map((point, index) => (
        <Sequence
          key={index}
          from={90 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60
            }}
          >
            <div
              style={{
                fontSize: 50,
                color: 'white',
                textAlign: 'center',
                opacity: Math.min(1, (frame - (90 + index * 90)) / 20)
              }}
            >
              {index + 1}. {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
      
      {/* Stats overlay */}
      <Sequence
        from={90 + content.points.length * 90}
        durationInFrames={120}
      >
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            flexDirection: 'column',
            gap: 30
          }}
        >
          {content.stats.map((stat, i) => (
            <div
              key={i}
              style={{
                fontSize: 40,
                color: '#00ff88',
                textAlign: 'center'
              }}
            >
              <strong>{stat.value}</strong> {stat.label}
            </div>
          ))}
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

```typescript
// Example: Full content pipeline from keyword to video
async function runContentPipeline(keyword: string) {
  // Step 1: Research
  const researchRes = await fetch('/api/research', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ keyword, timeframe: '24h' })
  });
  const { research } = await researchRes.json();
  
  // Step 2: Generate English content
  const contentEnRes = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      research: JSON.stringify(research),
      format: 'toplist',
      language: 'en',
      tone: 'expert'
    })
  });
  const { content: contentEn } = await contentEnRes.json();
  
  // Step 3: Generate Vietnamese content
  const contentViRes = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      research: JSON.stringify(research),
      format: 'toplist',
      language: 'vi',
      tone: 'friendly'
    })
  });
  const { content: contentVi } = await contentViRes.json();
  
  // Step 4: Render videos for multiple platforms
  const platforms = ['reels', 'tiktok', 'shorts'];
  const videos = await Promise.all(
    platforms.map(platform =>
      fetch('/api/render', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content: contentEn,
          platform,
          template: 'ContentVideo'
        })
      }).then(res => res.json())
    )
  );
  
  return {
    research,
    articles: { english: contentEn, vietnamese: contentVi },
    videos
  };
}
```

## Common Patterns

### Custom Content Format

```typescript
// lib/ai/formats/custom-format.ts
export function buildCustomPrompt(research: string, requirements: any) {
  return `
Create content with the following structure:
1. Hook (attention-grabbing opening)
2. Problem statement (based on research)
3. Solution breakdown (3-5 key points)
4. Data/Evidence (from research)
5. Call-to-action

Research: ${research}
Requirements: ${JSON.stringify(requirements)}
`;
}
```

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await crawlNewsSources({
        keyword,
        sources: ['techcrunch', 'twitter'],
        timeframe: '48h'
      });
      
      const content = await generateContent({
        research: JSON.stringify(research),
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        provider: 'claude'
      });
      
      return { keyword, research, content };
    })
  );
  
  return results;
}
```

### Video Template Customization

```typescript
// remotion/compositions/BrandedVideo.tsx
export const BrandedVideo: React.FC<{
  content: any;
  branding: { logo: string; colors: string[] };
}> = ({ content, branding }) => {
  return (
    <AbsoluteFill
      style={{
        background: `linear-gradient(135deg, ${branding.colors.join(', ')})`
      }}
    >
      <img
        src={branding.logo}
        style={{ position: 'absolute', top: 20, right: 20, width: 100 }}
      />
      {/* Content rendering */}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  
  constructor(private maxConcurrent: number = 3, private delay: number = 1000) {}
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }
    
    this.running++;
    try {
      return await fn();
    } finally {
      this.running--;
    }
  }
}

// Usage
const limiter = new RateLimiter(3, 1000);
const results = await Promise.all(
  keywords.map(k => limiter.add(() => crawlNewsSources({ keyword: k })))
);
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrency in .env.local
REMOTION_CONCURRENCY=2

// Or in code
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  concurrency: 2, // Lower concurrency for memory constraints
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-dev-shm-usage']
  }
});
```

### Claude API Timeout

```typescript
const message = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  timeout: 120000, // 2 minutes
  messages: [{ role: 'user', content: prompt }]
}).catch(error => {
  if (error.message.includes('timeout')) {
    // Retry with shorter content or different model
    return retryWithFallback(prompt);
  }
  throw error;
});
```

### Missing Research Data

```typescript
async function crawlWithFallback(keyword: string) {
  let research = await crawlNewsSources({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  // If insufficient data, expand timeframe
  if (research.totalArticles < 5) {
    research = await crawlNewsSources({
      keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '7d'
    });
  }
  
  return research;
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

# Run video renderer standalone
npm run remotion:render

# Type checking
npm run type-check

# Linting
npm run lint
```

This skill enables AI agents to help developers implement automated content pipelines that combine web scraping, AI generation, and video rendering for marketing workflows.
