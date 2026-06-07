---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, Facebook posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate AI-powered marketing content automatically
  - create video content from text with Remotion
  - build automated content pipeline with Claude
  - scrape news and generate social media posts
  - set up AI content workflow for Facebook
  - auto-generate multilingual marketing content
  - create content automation system with OpenAI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end automated content creation pipeline that handles research, scriptwriting, Facebook posting, and video generation. It crawls news sources, uses AI (Claude/OpenAI) to generate content in multiple formats and languages, and renders videos using Remotion.

## What It Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Research Phase**: Automatically crawls news from TechCrunch, a16z, X (Twitter), LinkedIn
2. **Content Generation**: Uses Claude 3 or OpenAI to create content in various formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Converts text content to videos using Remotion
5. **Auto-posting**: Publishes content to Facebook pages automatically

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

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# News/Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Facebook Integration
FACEBOOK_PAGE_ACCESS_TOKEN=your_facebook_token
FACEBOOK_PAGE_ID=your_page_id

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# App Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/
│   ├── ai/               # AI service integrations
│   ├── crawlers/         # News/content crawlers
│   ├── video/            # Remotion video generation
│   └── utils/            # Helper functions
├── remotion/             # Remotion video templates
├── public/               # Static assets
└── types/                # TypeScript type definitions
```

## Key API Usage

### 1. Research & Content Crawling

```typescript
import { crawlNews } from '@/lib/crawlers/news-crawler';

// Crawl latest news by keyword
async function fetchLatestNews(keyword: string) {
  const newsData = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    limit: 10
  });
  
  return newsData;
}

// Example usage
const aiNews = await fetchLatestNews('artificial intelligence');
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
  language: 'en' | 'vi'
) {
  const prompt = `Create a ${format} article about ${topic} in ${language}. 
Include recent data and insights. Make it engaging and SEO-optimized.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].text;
}

// Example usage
const article = await generateContent(
  'AI in Marketing 2026',
  'toplist',
  'en'
);
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  researchData: any[],
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} marketing content writer. 
Create engaging content based on the research provided.`
      },
      {
        role: 'user',
        content: `Research data: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(
  contentData: {
    title: string;
    highlights: string[];
    duration: number;
  }
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: contentData,
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}

// Example usage
const videoPath = await generateVideo({
  title: 'Top 5 AI Marketing Trends',
  highlights: [
    'Personalization at scale',
    'Predictive analytics',
    'Automated content creation'
  ],
  duration: 30
});
```

### 5. Facebook Auto-Posting

```typescript
import axios from 'axios';

async function postToFacebook(
  message: string,
  imageUrl?: string,
  videoUrl?: string
) {
  const pageAccessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN;
  const pageId = process.env.FACEBOOK_PAGE_ID;
  
  const endpoint = `https://graph.facebook.com/v18.0/${pageId}/feed`;

  const postData: any = {
    message,
    access_token: pageAccessToken,
  };

  if (imageUrl) {
    postData.link = imageUrl;
  }

  if (videoUrl) {
    // For video, use different endpoint
    const videoEndpoint = `https://graph.facebook.com/v18.0/${pageId}/videos`;
    const videoData = {
      file_url: videoUrl,
      description: message,
      access_token: pageAccessToken,
    };
    
    const response = await axios.post(videoEndpoint, videoData);
    return response.data;
  }

  const response = await axios.post(endpoint, postData);
  return response.data;
}
```

## Complete Pipeline Example

```typescript
import { crawlNews } from '@/lib/crawlers/news-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/renderer';
import { postToFacebook } from '@/lib/social/facebook';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching latest news...');
    const newsData = await crawlNews({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h',
      limit: 5
    });

    // Step 2: Generate content
    console.log('✍️ Generating content with AI...');
    const content = await generateContent({
      topic: keyword,
      researchData: newsData,
      format: 'toplist',
      language: 'en',
      tone: 'expert'
    });

    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      title: content.title,
      highlights: content.keyPoints,
      duration: 30
    });

    // Step 4: Post to Facebook
    console.log('📱 Posting to Facebook...');
    const fbPost = await postToFacebook(
      content.socialCaption,
      undefined,
      videoPath
    );

    console.log('✅ Pipeline completed!', fbPost);
    
    return {
      content,
      videoPath,
      fbPostId: fbPost.id
    };
  } catch (error) {
    console.error('❌ Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runContentPipeline('AI Marketing Automation').then(result => {
  console.log('Result:', result);
});
```

## API Routes (Next.js)

### Create Content Endpoint

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone } = body;

    const content = await generateContent({
      topic: keyword,
      format,
      language,
      tone
    });

    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Generation Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { title, highlights, duration } = body;

    const videoPath = await generateVideo({
      title,
      highlights,
      duration
    });

    return NextResponse.json({ 
      success: true, 
      videoUrl: `/videos/${path.basename(videoPath)}` 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Configuration Patterns

### Content Format Templates

```typescript
// lib/config/content-templates.ts
export const contentTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10,
  },
  pov: {
    structure: ['hook', 'opinion', 'evidence', 'callToAction'],
    tone: 'personal',
  },
  caseStudy: {
    structure: ['problem', 'solution', 'results', 'lessons'],
    includeData: true,
  },
  howTo: {
    structure: ['intro', 'steps', 'tips', 'conclusion'],
    stepByStep: true,
  }
};
```

### Multi-language Support

```typescript
// lib/config/languages.ts
export const languageConfig = {
  en: {
    name: 'English',
    toneVariants: ['professional', 'casual', 'witty'],
  },
  vi: {
    name: 'Tiếng Việt',
    toneVariants: ['chuyên nghiệp', 'thân thiện', 'hài hước'],
  }
};
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the application
# http://localhost:3000
```

## Common Workflows

### Daily Content Automation

```typescript
// scripts/daily-automation.ts
import cron from 'node-cron';
import { runContentPipeline } from '@/lib/pipeline';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = [
    'AI Marketing',
    'Social Media Trends',
    'Content Strategy'
  ];

  for (const topic of topics) {
    await runContentPipeline(topic);
  }
});
```

### Batch Video Generation

```typescript
async function batchGenerateVideos(contents: any[]) {
  const videos = await Promise.all(
    contents.map(content => 
      generateVideo({
        title: content.title,
        highlights: content.highlights,
        duration: 30
      })
    )
  );

  return videos;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pRetry from 'p-retry';

export async function withRetry<T>(
  fn: () => Promise<T>,
  options = {}
): Promise<T> {
  return pRetry(fn, {
    retries: 3,
    factor: 2,
    minTimeout: 1000,
    ...options,
    onFailedAttempt: error => {
      console.log(
        `Attempt ${error.attemptNumber} failed. Retrying...`
      );
    },
  });
}

// Usage
const content = await withRetry(() => 
  generateContent({ topic: 'AI', format: 'toplist' })
);
```

### Video Rendering Issues

If Remotion videos fail to render:

```bash
# Install required dependencies
npm install @remotion/renderer @remotion/bundler

# Ensure ffmpeg is installed
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows
# Download from https://ffmpeg.org/download.html
```

### Facebook API Errors

```typescript
// lib/utils/facebook-error-handler.ts
export function handleFacebookError(error: any) {
  if (error.response?.data?.error) {
    const fbError = error.response.data.error;
    
    switch (fbError.code) {
      case 190:
        throw new Error('Invalid access token. Please refresh.');
      case 368:
        throw new Error('Rate limit exceeded. Try again later.');
      default:
        throw new Error(`Facebook API Error: ${fbError.message}`);
    }
  }
  
  throw error;
}
```

### Memory Issues with Large Videos

```typescript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** for external API calls
3. **Cache research data** to avoid repeated crawls
4. **Queue video rendering** for better resource management
5. **Log all pipeline steps** for debugging
6. **Validate content** before posting to social media
7. **Use webhooks** for long-running processes
