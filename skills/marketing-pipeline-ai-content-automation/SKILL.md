---
name: marketing-pipeline-ai-content-automation
description: Ultimate AI content pipeline for automated research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from text with Remotion
  - crawl news and create AI content
  - build AI content automation workflow
  - create automated video content from research
  - use Claude for content generation pipeline
  - automate social media content with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI-powered content automation system that transforms keywords into complete content packages including research, scripts, and rendered videos. It combines web crawling for fresh data, AI content generation (Claude 3, OpenAI), and automated video rendering (Remotion) to create a complete marketing content factory.

**Key capabilities:**
- Auto-crawl recent news from TechCrunch, a16z, Twitter, LinkedIn
- Generate multi-format content (Toplist, POV, Case Study, How-to)
- Bilingual output (English/Vietnamese) with customizable tone
- Auto-render videos and infographics from content
- Next.js frontend for easy content management

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript types
├── remotion/            # Remotion video compositions
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Content Research & Crawling

```typescript
import { crawlRecentNews } from '@/lib/crawler';

interface NewsSource {
  url: string;
  selector: string;
  source: string;
}

const sources: NewsSource[] = [
  { url: 'https://techcrunch.com', selector: '.post-block', source: 'TechCrunch' },
  { url: 'https://a16z.com/blog', selector: 'article', source: 'a16z' }
];

async function gatherResearch(keyword: string, timeframe: '24h' | '7d' = '24h') {
  const results = await crawlRecentNews({
    keyword,
    sources,
    timeframe,
    maxResults: 50
  });

  return results.map(item => ({
    title: item.title,
    content: item.content,
    url: item.url,
    publishedAt: item.publishedAt,
    source: item.source,
    insights: item.extractedInsights
  }));
}
```

### 2. AI Content Generation

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

async function generateContent(config: ContentConfig) {
  const systemPrompt = `You are an expert content creator specializing in ${config.format} articles. 
  Write in ${config.language === 'vi' ? 'Vietnamese' : 'English'} with a ${config.tone} tone.`;

  const userPrompt = `Create a ${config.format} article about "${config.keyword}" using this research data:
  ${JSON.stringify(config.researchData, null, 2)}
  
  Include:
  - Compelling headline
  - Data-backed insights
  - Actionable takeaways
  - SEO-optimized structure`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ],
    system: systemPrompt
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateContentOpenAI(config: ContentConfig) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `Expert ${config.format} content writer. Language: ${config.language}. Tone: ${config.tone}.`
      },
      {
        role: 'user',
        content: `Write about "${config.keyword}" using this research: ${JSON.stringify(config.researchData)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  branding: {
    logo: string;
    primaryColor: string;
  };
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, keyPoints, branding }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity: Math.min(1, frame / 30)
        }}>
          <h1 style={{ color: branding.primaryColor, fontSize: 60 }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {keyPoints.map((point, index) => (
        <Sequence key={index} from={fps * (2 + index * 3)} durationInFrames={fps * 3}>
          <AbsoluteFill style={{ justifyContent: 'center', padding: 60 }}>
            <p style={{ color: '#fff', fontSize: 40 }}>{point}</p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderConfig {
  content: {
    title: string;
    keyPoints: string[];
  };
  format: '16:9' | '9:16' | '1:1'; // Landscape, Reels/Shorts, Square
  outputPath: string;
}

async function renderContentVideo(config: RenderConfig) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.content.title,
      keyPoints: config.content.keyPoints,
      branding: {
        logo: '/logo.png',
        primaryColor: '#FF6B35'
      }
    }
  });

  const dimensions = {
    '16:9': { width: 1920, height: 1080 },
    '9:16': { width: 1080, height: 1920 },
    '1:1': { width: 1080, height: 1080 }
  }[config.format];

  await renderMedia({
    composition: {
      ...composition,
      width: dimensions.width,
      height: dimensions.height
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: config.outputPath,
    inputProps: composition.defaultProps
  });

  return config.outputPath;
}
```

### 5. Complete Pipeline Integration

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, generateVideo } = await req.json();

    // Step 1: Gather research
    const researchData = await gatherResearch(keyword, '24h');

    // Step 2: Generate content
    const content = await generateContent({
      keyword,
      format,
      language,
      tone: 'expert',
      researchData
    });

    // Parse content to extract key points
    const keyPoints = extractKeyPoints(content);

    let videoUrl = null;
    if (generateVideo) {
      // Step 3: Render video
      const outputPath = `./public/videos/${Date.now()}.mp4`;
      await renderContentVideo({
        content: {
          title: keyword,
          keyPoints: keyPoints.slice(0, 5)
        },
        format: '16:9',
        outputPath
      });
      videoUrl = outputPath.replace('./public', '');
    }

    return NextResponse.json({
      success: true,
      data: {
        content,
        keyPoints,
        videoUrl,
        researchSources: researchData.length
      }
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction logic - enhance based on content structure
  const lines = content.split('\n').filter(line => 
    line.trim().startsWith('-') || 
    line.trim().startsWith('•') ||
    line.match(/^\d+\./)
  );
  return lines.map(line => line.replace(/^[-•\d.]\s*/, '').trim());
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run production server
npm start

# Render Remotion video locally (preview)
npx remotion preview remotion/index.ts

# Render specific composition
npx remotion render remotion/index.ts ContentVideo output.mp4

# Type checking
npm run type-check

# Linting
npm run lint
```

## Common Workflows

### Workflow 1: Research to Article

```typescript
async function researchToArticle(topic: string) {
  // 1. Crawl news
  const news = await gatherResearch(topic, '24h');
  
  // 2. Generate English version
  const enContent = await generateContent({
    keyword: topic,
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    researchData: news
  });
  
  // 3. Generate Vietnamese version
  const viContent = await generateContent({
    keyword: topic,
    format: 'toplist',
    language: 'vi',
    tone: 'friendly',
    researchData: news
  });
  
  return { enContent, viContent, sources: news.length };
}
```

### Workflow 2: Article to Multi-Platform Videos

```typescript
async function articleToVideos(content: string, title: string) {
  const keyPoints = extractKeyPoints(content);
  
  const formats: Array<'16:9' | '9:16' | '1:1'> = ['16:9', '9:16', '1:1'];
  const videos: Record<string, string> = {};
  
  for (const format of formats) {
    const outputPath = `./public/videos/${format}-${Date.now()}.mp4`;
    await renderContentVideo({
      content: { title, keyPoints: keyPoints.slice(0, 5) },
      format,
      outputPath
    });
    videos[format] = outputPath.replace('./public', '');
  }
  
  return videos; // { '16:9': '/videos/...', '9:16': '/videos/...', '1:1': '/videos/...' }
}
```

## Troubleshooting

### Issue: Claude API Rate Limits

```typescript
// Implement retry with exponential backoff
async function generateContentWithRetry(config: ContentConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(config);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Issue: Remotion Rendering Fails

Check Node.js version (requires 18+) and ensure FFmpeg is installed:

```bash
# Install FFmpeg (macOS)
brew install ffmpeg

# Install FFmpeg (Ubuntu)
sudo apt-get install ffmpeg

# Verify installation
ffmpeg -version
```

### Issue: Crawling Blocked

```typescript
// Add user agent and delay between requests
import { chromium } from 'playwright';

async function crawlWithBrowser(url: string) {
  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext({
    userAgent: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
  });
  const page = await context.newPage();
  
  await page.goto(url, { waitUntil: 'networkidle' });
  const content = await page.content();
  
  await browser.close();
  return content;
}
```

### Issue: Out of Memory During Video Render

```typescript
// Reduce video quality or split into smaller segments
await renderMedia({
  // ... other options
  scale: 0.75, // Reduce resolution
  concurrency: 1, // Reduce parallel processing
  enforceAudioTrack: false // Skip audio if not needed
});
```

## Best Practices

1. **Rate Limiting**: Implement request queues for API calls
2. **Caching**: Cache research results for 24h to avoid redundant crawls
3. **Error Handling**: Always wrap AI calls in try-catch with fallbacks
4. **Content Validation**: Verify generated content for brand safety before publishing
5. **Video Optimization**: Compress videos before serving (use `ffmpeg` post-processing)
6. **Environment Separation**: Use different API keys for dev/staging/production
