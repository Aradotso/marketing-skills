---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, auto-posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate videos automatically from blog posts
  - crawl news and create social media content
  - build automated marketing content workflow
  - create TikTok and Reels videos from articles
  - automate Facebook page posting with AI
  - research trending topics and generate content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation. The pipeline crawls trending news, generates multi-format content using Claude/OpenAI, and automatically renders videos using Remotion.

## What This Project Does

The Marketing Pipeline Share is an all-in-one content automation system that:

- **Auto-researches** trending topics from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- **Generates content** in multiple formats (Top List, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Writes in multiple languages** (English & Vietnamese) with customizable tone
- **Renders videos** automatically from written content using Remotion
- **Auto-posts** to Facebook pages and social platforms
- **Exports optimized** video formats for Reels, TikTok, and Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ and npm/yarn required
node --version  # Should be 18.x or higher
```

### Clone and Install

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Setup

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_API_KEY=your_twitter_api_key_here

# Social Media Integration
FACEBOOK_PAGE_TOKEN=your_facebook_page_token_here
FACEBOOK_PAGE_ID=your_page_id_here

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

The application will be available at `http://localhost:3000`

## Key Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI providers (Claude, OpenAI)
│   │   ├── crawlers/    # News crawling modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── services/        # API services
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core API Usage

### 1. Research & Crawling

```typescript
// src/lib/crawlers/newsScanner.ts
import { RapidAPI } from '@/lib/api/rapidapi';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}

export async function scanTrendingNews(
  keyword: string,
  hours: number = 24
): Promise<NewsArticle[]> {
  const rapidAPI = new RapidAPI(process.env.RAPIDAPI_KEY!);
  
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    const results = await rapidAPI.searchNews({
      query: keyword,
      source,
      hours,
    });
    
    articles.push(...results);
  }
  
  return articles;
}

// Usage in API route
import { scanTrendingNews } from '@/lib/crawlers/newsScanner';

export async function POST(request: Request) {
  const { keyword } = await request.json();
  
  const articles = await scanTrendingNews(keyword, 24);
  
  return Response.json({ articles, count: articles.length });
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const systemPrompt = `You are an expert content creator specializing in ${request.format} format.
Write in ${request.language === 'vi' ? 'Vietnamese' : 'English'} with a ${request.tone} tone.
Use the provided research data to create original, data-backed insights.`;

  const userPrompt = `Create a ${request.format} article about "${request.keyword}".

Research Data:
${request.researchData}

Requirements:
- Include specific data points and statistics
- Make it engaging and actionable
- Optimize for social media sharing
- Add relevant hashtags`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. Alternative: OpenAI Content Generation

```typescript
// src/lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentOpenAI(
  request: ContentRequest
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} format.`,
      },
      {
        role: 'user',
        content: `Create a ${request.format} article about "${request.keyword}" using this research:\n\n${request.researchData}`,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  outputPath: string;
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  // Get video dimensions based on format
  const dimensions = {
    reels: { width: 1080, height: 1920 }, // 9:16
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  const { width, height } = dimensions[config.format];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  // Render video
  await renderMedia({
    composition: {
      ...composition,
      width,
      height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: config.outputPath,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return config.outputPath;
}
```

### 5. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Sequence } from 'remotion';
import { z } from 'zod';

export const contentVideoSchema = z.object({
  title: z.string(),
  content: z.string(),
});

export const ContentVideo: React.FC<z.infer<typeof contentVideoSchema>> = ({
  title,
  content,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  const contentPoints = content.split('\n').filter(Boolean);

  return (
    <AbsoluteFill style={{ backgroundColor: '#0f172a' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1 style={{ 
            color: 'white', 
            fontSize: 72, 
            textAlign: 'center',
            padding: '0 60px',
            fontWeight: 'bold'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {contentPoints.map((point, index) => (
        <Sequence 
          key={index} 
          from={60 + index * 90} 
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: '0 80px',
            }}
          >
            <div style={{
              color: 'white',
              fontSize: 48,
              lineHeight: 1.5,
              textAlign: 'center',
              backgroundColor: 'rgba(0,0,0,0.5)',
              padding: '40px',
              borderRadius: '20px',
            }}>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 6. Facebook Auto-Posting

```typescript
// src/lib/social/facebook.ts
interface FacebookPost {
  message: string;
  videoUrl?: string;
  imageUrl?: string;
  link?: string;
}

export async function postToFacebookPage(
  post: FacebookPost
): Promise<string> {
  const pageToken = process.env.FACEBOOK_PAGE_TOKEN!;
  const pageId = process.env.FACEBOOK_PAGE_ID!;

  let endpoint = `https://graph.facebook.com/v18.0/${pageId}`;
  const formData = new FormData();
  formData.append('access_token', pageToken);
  formData.append('message', post.message);

  if (post.videoUrl) {
    endpoint += '/videos';
    formData.append('file_url', post.videoUrl);
  } else if (post.imageUrl) {
    endpoint += '/photos';
    formData.append('url', post.imageUrl);
  } else {
    endpoint += '/feed';
    if (post.link) {
      formData.append('link', post.link);
    }
  }

  const response = await fetch(endpoint, {
    method: 'POST',
    body: formData,
  });

  const result = await response.json();
  return result.id || result.post_id;
}
```

## Complete Workflow Example

```typescript
// src/app/api/pipeline/route.ts
import { scanTrendingNews } from '@/lib/crawlers/newsScanner';
import { generateContent } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/renderer';
import { postToFacebookPage } from '@/lib/social/facebook';

export async function POST(request: Request) {
  const { keyword, format, language, platform } = await request.json();

  try {
    // Step 1: Research
    console.log('🔍 Scanning trending news...');
    const articles = await scanTrendingNews(keyword, 24);
    const researchData = articles
      .map(a => `${a.title}\n${a.content}`)
      .join('\n\n---\n\n');

    // Step 2: Generate Content
    console.log('✍️ Generating content with AI...');
    const content = await generateContent({
      keyword,
      format,
      language,
      tone: 'expert',
      researchData,
    });

    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: keyword,
      content,
      format: platform,
      outputPath: `./public/videos/${Date.now()}.mp4`,
    });

    // Step 4: Auto-post
    console.log('📤 Posting to Facebook...');
    const postId = await postToFacebookPage({
      message: content,
      videoUrl: `${process.env.NEXT_PUBLIC_APP_URL}${videoPath}`,
    });

    return Response.json({
      success: true,
      postId,
      videoPath,
      articleCount: articles.length,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return Response.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
// Generate both English and Vietnamese versions
async function generateBilingualContent(keyword: string, researchData: string) {
  const [enContent, viContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData,
    }),
    generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData,
    }),
  ]);

  return { en: enContent, vi: viContent };
}
```

### Batch Video Rendering

```typescript
async function renderMultiplePlatforms(title: string, content: string) {
  const platforms: Array<'reels' | 'tiktok' | 'shorts'> = [
    'reels',
    'tiktok',
    'shorts',
  ];

  const videos = await Promise.all(
    platforms.map(platform =>
      renderContentVideo({
        title,
        content,
        format: platform,
        outputPath: `./public/videos/${platform}-${Date.now()}.mp4`,
      })
    )
  );

  return videos;
}
```

### Scheduled Content Pipeline

```typescript
// Use with cron or scheduled API calls
import { CronJob } from 'cron';

const dailyContentJob = new CronJob('0 9 * * *', async () => {
  const keywords = ['AI trends', 'Tech news', 'Marketing tips'];
  
  for (const keyword of keywords) {
    await fetch('http://localhost:3000/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        language: 'vi',
        platform: 'reels',
      }),
    });
  }
});

dailyContentJob.start();
```

## Configuration

### Customize AI Prompts

```typescript
// src/config/prompts.ts
export const CONTENT_PROMPTS = {
  toplist: {
    system: 'You create engaging top 10 lists with actionable insights',
    template: 'Create a top ${count} list about ${keyword}',
  },
  pov: {
    system: 'You write thought-provoking opinion pieces',
    template: 'Share a unique perspective on ${keyword}',
  },
  'case-study': {
    system: 'You analyze real-world examples in depth',
    template: 'Analyze a case study about ${keyword}',
  },
  'how-to': {
    system: 'You write clear, step-by-step tutorials',
    template: 'Create a how-to guide for ${keyword}',
  },
};
```

### Video Template Customization

```typescript
// remotion/config.ts
export const VIDEO_CONFIG = {
  fps: 30,
  duration: 30, // seconds
  fonts: {
    title: 'Inter-Bold',
    body: 'Inter-Regular',
  },
  colors: {
    background: '#0f172a',
    text: '#ffffff',
    accent: '#3b82f6',
  },
};
```

## Troubleshooting

### API Rate Limiting

```typescript
// Add retry logic with exponential backoff
async function generateContentWithRetry(
  request: ContentRequest,
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Remotion Rendering Issues

```bash
# Install required dependencies for video rendering
npm install --save-dev @remotion/bundler @remotion/renderer

# For headless server environments
sudo apt-get install -y chromium-browser

# Set Chromium path
export PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
```

### Memory Issues with Large Videos

```typescript
// Render videos in chunks
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo',
  inputProps,
});

// Render shorter segments
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  concurrency: 1, // Reduce concurrency
  enforceAudioTrack: false, // Disable if no audio
});
```

### Environment Variable Issues

```typescript
// Validate environment variables on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
    'FACEBOOK_PAGE_TOKEN',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}

validateEnv();
```

This skill enables agents to help developers build complete automated content pipelines combining AI research, content generation, video rendering, and social media posting.
