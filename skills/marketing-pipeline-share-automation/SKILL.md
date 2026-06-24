---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation with Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - generate marketing content from research to video automatically
  - create AI content pipeline with Claude and Remotion
  - build automated content workflow with video rendering
  - set up AI marketing automation with auto research
  - generate content and videos from keywords automatically
  - create content pipeline with AI research and rendering
  - automate marketing content from scraping to video
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **marketing-pipeline-share**, an end-to-end AI content automation system that handles research (web scraping), content generation (Claude/OpenAI), and video rendering (Remotion) in a single pipeline. Input a keyword, get researched articles and rendered videos ready for publication.

## What This Project Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) in multiple languages using Claude 3 or OpenAI
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-Platform Export**: Outputs optimized for Reels, TikTok, Shorts

Built with Next.js (TypeScript), it integrates Anthropic Claude, OpenAI, RapidAPI for scraping, and Remotion for video generation.

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

# Copy environment template
cp .env.example .env.local
```

## Environment Configuration

Set up your `.env.local` file with required API keys:

```bash
# AI Providers (use at least one)
ANTHROPIC_API_KEY=
OPENAI_API_KEY=

# Research/Scraping APIs
RAPIDAPI_KEY=
TWITTER_API_KEY=
LINKEDIN_API_KEY=

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=

# Database (if applicable)
DATABASE_URL=

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```typescript
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Auto-scraping endpoints
│   │   ├── generate/      # Content generation
│   │   └── render/        # Video rendering
│   ├── components/        # React components
│   └── page.tsx           # Main interface
├── lib/                   # Core utilities
│   ├── ai/               # AI provider integrations
│   ├── scraper/          # Web scraping logic
│   └── remotion/         # Video templates
├── remotion/             # Remotion video compositions
└── public/               # Static assets
```

## Core API Usage

### 1. Research & Scraping

```typescript
// lib/scraper/sources.ts
import axios from 'axios';

interface ResearchResult {
  source: string;
  title: string;
  content: string;
  url: string;
  publishedAt: Date;
}

export async function scrapeRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const articles = await fetchFromSource(source, keyword);
    results.push(...articles);
  }
  
  return results.sort((a, b) => 
    b.publishedAt.getTime() - a.publishedAt.getTime()
  ).slice(0, 10);
}

async function fetchFromSource(
  source: string, 
  keyword: string
): Promise<ResearchResult[]> {
  const config = {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': `${source}-api.p.rapidapi.com`
    }
  };
  
  const response = await axios.get(
    `https://${source}-api.p.rapidapi.com/search`,
    { ...config, params: { q: keyword, limit: 5 } }
  );
  
  return response.data.articles.map((article: any) => ({
    source,
    title: article.title,
    content: article.description,
    url: article.url,
    publishedAt: new Date(article.publishedAt)
  }));
}
```

### 2. AI Content Generation

```typescript
// lib/ai/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  research: ResearchResult[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(request);
  
  if (provider === 'claude') {
    return generateWithClaude(prompt);
  }
  return generateWithOpenAI(prompt);
}

function buildPrompt(request: ContentRequest): string {
  const researchContext = request.research
    .map(r => `- ${r.title} (${r.source}): ${r.content}`)
    .join('\n');
  
  return `You are a ${request.tone} content creator. 
Generate a ${request.format} article in ${request.language} about "${request.keyword}".

Recent research data (last 24h):
${researchContext}

Requirements:
- Format: ${request.format}
- Language: ${request.language === 'en' ? 'English' : 'Vietnamese'}
- Tone: ${request.tone}
- Include data-backed insights
- 800-1200 words
- Include actionable takeaways`;
}

async function generateWithClaude(prompt: string): Promise<string> {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const message = await client.messages.create({
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

async function generateWithOpenAI(prompt: string): Promise<string> {
  const client = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });
  
  const completion = await client.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: prompt }],
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content || '';
}
```

### 3. Video Rendering with Remotion

```typescript
// lib/remotion/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reel' | 'tiktok' | 'short'; // 9:16 vertical
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: parseContentForVideo(config.content),
      format: config.format
    }
  });
  
  const outputPath = path.join(
    process.cwd(), 
    'public/videos', 
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps
  });
  
  return outputPath;
}

function parseContentForVideo(content: string): string[] {
  // Extract key points for video slides
  const lines = content.split('\n');
  const keyPoints: string[] = [];
  
  lines.forEach(line => {
    if (line.match(/^[-•]\s/) || line.match(/^\d+\./)) {
      keyPoints.push(line.replace(/^[-•\d.]\s*/, ''));
    }
  });
  
  return keyPoints.slice(0, 5); // Max 5 slides
}
```

```tsx
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface Props {
  title: string;
  content: string[];
  format: 'reel' | 'tiktok' | 'short';
}

export const ContentVideo: React.FC<Props> = ({ title, content, format }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <TitleSlide title={title} />
      </Sequence>
      
      {content.map((point, index) => (
        <Sequence 
          key={index}
          from={90 + (index * 90)} 
          durationInFrames={90}
        >
          <ContentSlide text={point} index={index + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## API Routes

### Generate Full Pipeline

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeRecentNews } from '@/lib/scraper/sources';
import { generateContent } from '@/lib/ai/generator';
import { renderContentVideo } from '@/lib/remotion/render';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, generateVideo } = await request.json();
    
    // Step 1: Research
    const research = await scrapeRecentNews(keyword);
    
    // Step 2: Generate Content
    const content = await generateContent({
      keyword,
      research,
      format,
      language,
      tone
    });
    
    // Step 3: Render Video (optional)
    let videoUrl = null;
    if (generateVideo) {
      const videoPath = await renderContentVideo({
        content,
        title: keyword,
        format: 'reel'
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        research: research.map(r => ({ title: r.title, url: r.url }))
      }
    });
    
  } catch (error) {
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { scrapeRecentNews } from '@/lib/scraper/sources';
import { generateContent } from '@/lib/ai/generator';

const keywords = ['AI automation', 'content marketing', 'video creation'];
const formats: ContentFormat[] = ['toplist', 'how-to', 'case-study'];

async function batchGenerate() {
  for (const keyword of keywords) {
    const research = await scrapeRecentNews(keyword);
    
    for (const format of formats) {
      const content = await generateContent({
        keyword,
        research,
        format,
        language: 'en',
        tone: 'expert'
      });
      
      // Save to database or file
      await saveContent(keyword, format, content);
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }
}
```

### Custom Video Templates

```tsx
// remotion/compositions.ts
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={540}
        fps={30}
        width={1080}
        height={1920} // 9:16 for vertical video
        defaultProps={{
          title: 'Sample Title',
          content: [],
          format: 'reel'
        }}
      />
    </>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      if (!this.processing) {
        this.process();
      }
    });
  }
  
  private async process() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1 req/sec
    }
    
    this.processing = false;
  }
}
```

### Video Rendering Memory Issues

```typescript
// Reduce composition complexity or render in chunks
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: compositionId,
  inputProps: {
    ...props,
    quality: 'medium' // Lower quality for large batches
  }
});

await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-dev-shm-usage']
  }
});
```

### Scraping Failures

```typescript
// Implement retry logic with exponential backoff
async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Open browser
# Navigate to http://localhost:3000

# Test API endpoint
curl -X POST http://localhost:3000/api/pipeline \
  -H "Content-Type: application/json" \
  -d '{
    "keyword": "AI automation",
    "format": "toplist",
    "language": "en",
    "tone": "expert",
    "generateVideo": true
  }'
```

## Production Deployment

```bash
# Build for production
npm run build

# Start production server
npm run start

# Or deploy to Vercel
vercel --prod
```

For detailed setup instructions, refer to `HUONG_DAN_CAI_DAT.md` in the project root.
