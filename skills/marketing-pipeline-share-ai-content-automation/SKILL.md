---
name: marketing-pipeline-share-ai-content-automation
description: Automated content pipeline from research to video generation using Claude/OpenAI, web scraping, and Remotion rendering
triggers:
  - how do I set up the AI content pipeline
  - automate content research and video generation
  - configure Claude API for content automation
  - create automated marketing content with AI
  - generate videos from content automatically
  - scrape news sources for content research
  - build AI content pipeline with remotion
  - set up marketing automation with TypeScript
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research (web scraping), content generation (Claude/OpenAI), and video rendering (Remotion). It automates content creation from keyword input to publishable video output.

## What It Does

- **Auto-Research**: Scrapes TechCrunch, a16z, Twitter/X, LinkedIn for recent news (24h window)
- **AI Content Generation**: Creates diverse formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese
- **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
- **Multi-platform Export**: Optimizes videos for Reels, TikTok, Shorts

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Scraping/Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom API endpoints
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
│   │   ├── ai/          # AI provider integrations
│   │   ├── scraper/     # Web scraping modules
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Content Research & Scraping

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';
import { analyzeTrends } from '@/lib/scraper/trend-analyzer';

// Scrape recent news from multiple sources
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await scrapeNews({
    keyword,
    sources,
    timeframe: '24h',
    limit: 50
  });

  // Analyze and extract insights
  const insights = await analyzeTrends(newsData);
  
  return {
    articles: newsData,
    insights,
    keywords: insights.topKeywords,
    stats: insights.statistics
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  researchData: any
) {
  const systemPrompt = `You are an expert content creator. 
  Create a ${format} article about ${topic} in ${language}.
  Use the following research data: ${JSON.stringify(researchData)}`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Create an engaging ${format} article about ${topic}. 
        Include data-backed insights and recent trends.`
      }
    ]
  });

  return message.content[0].text;
}
```

### 3. Alternative: OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  topic: string,
  context: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a marketing content expert.'
      },
      {
        role: 'user',
        content: `Create content about ${topic}. Context: ${context}`
      }
    ],
    temperature: 0.8,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';

async function renderContentVideo(
  content: string,
  title: string,
  format: 'reels' | 'tiktok' | 'shorts'
) {
  // Video dimensions based on format
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content,
      ...dimensions[format]
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${title}-${format}.mp4`,
    inputProps: {
      title,
      content,
    },
  });

  return `out/${title}-${format}.mp4`;
}
```

### 5. Complete Pipeline Example

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/remotion-renderer';

async function runContentPipeline(
  keyword: string,
  config: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    languages: ('en' | 'vi')[];
    videoFormats: ('reels' | 'tiktok' | 'shorts')[];
  }
) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await scrapeNews({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeframe: '24h'
  });

  // Step 2: Generate content for each language
  const contents = await Promise.all(
    config.languages.map(async (lang) => {
      console.log(`✍️ Generating ${lang} content...`);
      const content = await generateContent(
        keyword,
        config.format,
        lang,
        research
      );
      return { lang, content };
    })
  );

  // Step 3: Render videos
  const videos = await Promise.all(
    contents.flatMap(({ lang, content }) =>
      config.videoFormats.map(async (format) => {
        console.log(`🎬 Rendering ${format} video (${lang})...`);
        const videoPath = await renderContentVideo(
          content,
          `${keyword}-${lang}`,
          format
        );
        return { lang, format, videoPath };
      })
    )
  );

  return {
    research,
    contents,
    videos
  };
}

// Usage
const result = await runContentPipeline('AI Marketing Trends', {
  format: 'toplist',
  languages: ['en', 'vi'],
  videoFormats: ['reels', 'tiktok']
});
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNews } from '@/lib/scraper/news-scraper';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();

  try {
    const data = await scrapeNews({ keyword, sources, timeframe });
    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude-generator';

export async function POST(request: NextRequest) {
  const { topic, format, language, research } = await request.json();

  try {
    const content = await generateContent(topic, format, language, research);
    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
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

# Render Remotion video (development)
npm run remotion:dev

# Render Remotion video (production)
npm run remotion:render -- --props='{"title":"My Video"}'

# Type checking
npm run type-check

# Linting
npm run lint
```

## Remotion Video Templates

Create a custom video template:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps, width, height } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div style={{ opacity }}>
        <h1 style={{ color: 'white', fontSize: 80 }}>{title}</h1>
        <p style={{ color: 'white', fontSize: 40, maxWidth: width * 0.8 }}>
          {content}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

Register the composition:

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';

registerRoot(() => {
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
});
```

## Common Patterns

### Rate Limiting & Error Handling

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMinutes: 1
});

async function safeAPICall<T>(
  fn: () => Promise<T>,
  retries = 3
): Promise<T> {
  await limiter.waitForSlot();

  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  
  throw new Error('Max retries exceeded');
}
```

### Content Quality Validation

```typescript
interface ContentQuality {
  wordCount: number;
  readabilityScore: number;
  hasData: boolean;
  hasHeadings: boolean;
}

function validateContent(content: string): ContentQuality {
  const wordCount = content.split(/\s+/).length;
  const hasData = /\d+%|\$\d+|statistics|data shows/i.test(content);
  const hasHeadings = /#+ /m.test(content);

  return {
    wordCount,
    readabilityScore: calculateReadability(content),
    hasData,
    hasHeadings,
  };
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// Implement exponential backoff
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 5
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const waitTime = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Waiting ${waitTime}ms...`);
        await delay(waitTime);
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Scraping Failures

Check for blocked requests or invalid selectors:

```typescript
// Add user agent and retry logic
import axios from 'axios';

const scrapeWithHeaders = async (url: string) => {
  return axios.get(url, {
    headers: {
      'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)',
      'Accept': 'text/html,application/xhtml+xml',
    },
    timeout: 10000,
  });
};
```

### Video Rendering Memory Issues

For large video renders:

```typescript
// Reduce quality or split into chunks
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  quality: 80, // Reduce from default 100
  concurrency: 1, // Limit concurrent frames
  outputLocation: outputPath,
});
```

### TypeScript Configuration

Ensure proper types for AI responses:

```typescript
// types/content.ts
export interface GeneratedContent {
  title: string;
  body: string;
  metadata: {
    format: string;
    language: string;
    wordCount: number;
  };
  sources: string[];
}

// Use with AI generation
const content = await generateContent(...) as GeneratedContent;
```
