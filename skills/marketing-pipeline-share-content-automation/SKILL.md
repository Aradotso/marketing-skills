---
name: marketing-pipeline-share-content-automation
description: Automated AI content pipeline for research, scripting, posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up automated marketing content pipeline
  - generate videos from content automatically
  - create content from research to video with AI
  - automate content workflow with Claude and OpenAI
  - build AI-powered content generation system
  - crawl news and generate content automatically
  - use Remotion to render content videos
---

# Marketing Pipeline Share - AI Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**marketing-pipeline-share** is a complete AI-powered content automation pipeline built with TypeScript/Next.js. It handles the entire content creation workflow:

1. **Research**: Auto-crawl news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Create articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language**: Generate parallel English and Vietnamese versions
4. **Video Rendering**: Auto-generate infographics and short-form videos using Remotion
5. **Publishing**: Schedule and auto-post to social platforms

This system transforms a single keyword into research-backed, multi-format content with video assets.

## Installation

### Prerequisites

```bash
node >= 18.x
npm or yarn
```

### Clone and Setup

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Social Media APIs (for auto-posting)
FACEBOOK_PAGE_TOKEN=your_facebook_token
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_TOKEN=your_linkedin_token

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Development Server

```bash
npm run dev
# or
yarn dev
```

Visit `http://localhost:3000` to access the UI.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Research & Content Crawling

**Auto-scan news sources:**

```typescript
// src/lib/crawler/news-scanner.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsItem {
  title: string;
  url: string;
  source: string;
  publishedAt: string;
  content: string;
}

export async function scanLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter', 'linkedin']
): Promise<NewsItem[]> {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const newsPromises = sources.map(source => 
    rapidAPI.searchNews({
      query: keyword,
      source,
      timeRange: '24h'
    })
  );
  
  const results = await Promise.all(newsPromises);
  return results.flat();
}
```

**Usage in your workflow:**

```typescript
import { scanLatestNews } from '@/lib/crawler/news-scanner';

const keyword = "AI automation";
const newsItems = await scanLatestNews(keyword);

console.log(`Found ${newsItems.length} recent articles`);
```

### 2. AI Content Generation

**Generate content using Claude or OpenAI:**

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ToneOfVoice = 'expert' | 'friendly' | 'humorous';
export type Language = 'en' | 'vi';

interface ContentGenerationParams {
  keyword: string;
  researchData: string[];
  format: ContentFormat;
  tone: ToneOfVoice;
  language: Language;
  aiProvider: 'claude' | 'openai';
}

export async function generateContent(
  params: ContentGenerationParams
): Promise<string> {
  const prompt = buildPrompt(params);
  
  if (params.aiProvider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
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
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096,
    });
    
    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(params: ContentGenerationParams): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article highlighting the top items',
    'pov': 'Write from a unique perspective or opinion angle',
    'case-study': 'Analyze a specific example with data and outcomes',
    'how-to': 'Provide step-by-step instructional content'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry jargon',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include light humor and engaging storytelling'
  };
  
  return `
You are a ${params.tone} content writer creating a ${params.format} article in ${params.language === 'en' ? 'English' : 'Vietnamese'}.

Topic: ${params.keyword}

Research Data:
${params.researchData.join('\n\n')}

Instructions:
- ${formatInstructions[params.format]}
- ${toneInstructions[params.tone]}
- Use data and insights from the research provided
- Make it engaging and actionable
- Include relevant statistics and examples
${params.language === 'vi' ? '- Write entirely in Vietnamese with natural flow' : ''}

Generate the complete article now:
  `.trim();
}
```

**Complete workflow example:**

```typescript
import { scanLatestNews } from '@/lib/crawler/news-scanner';
import { generateContent } from '@/lib/ai/content-generator';

async function createContentPipeline(keyword: string) {
  // Step 1: Research
  const newsItems = await scanLatestNews(keyword);
  const researchData = newsItems.map(item => 
    `${item.title}\n${item.content}`
  );
  
  // Step 2: Generate English version
  const englishContent = await generateContent({
    keyword,
    researchData,
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    aiProvider: 'claude'
  });
  
  // Step 3: Generate Vietnamese version
  const vietnameseContent = await generateContent({
    keyword,
    researchData,
    format: 'toplist',
    tone: 'expert',
    language: 'vi',
    aiProvider: 'claude'
  });
  
  return {
    english: englishContent,
    vietnamese: vietnameseContent,
    sourceCount: newsItems.length
  };
}
```

### 3. Video Generation with Remotion

**Define video composition:**

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  bgColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  bgColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{
        padding: 60,
        fontFamily: 'Arial',
        color: 'white',
        opacity
      }}>
        <h1 style={{ fontSize: 72, marginBottom: 40 }}>
          {title}
        </h1>
        <ul style={{ fontSize: 36, lineHeight: 1.8 }}>
          {keyPoints.map((point, idx) => (
            <li key={idx} style={{
              opacity: frame > fps * (idx + 1) ? 1 : 0,
              transition: 'opacity 0.5s'
            }}>
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

**Render video programmatically:**

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderParams {
  title: string;
  keyPoints: string[];
  outputPath: string;
  format?: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
}

export async function renderContentVideo(
  params: VideoRenderParams
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition details
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: params.title,
      keyPoints: params.keyPoints,
      bgColor: '#1a1a1a'
    },
  });
  
  // Render video
  const outputLocation = path.resolve(params.outputPath);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: params.title,
      keyPoints: params.keyPoints,
      bgColor: '#1a1a1a'
    },
  });
  
  return outputLocation;
}
```

**Usage:**

```typescript
import { renderContentVideo } from '@/lib/video/renderer';

const videoPath = await renderContentVideo({
  title: "Top 5 AI Automation Tools",
  keyPoints: [
    "Claude API for content generation",
    "Remotion for video automation",
    "RapidAPI for data collection",
    "Next.js for workflow management",
    "TypeScript for type safety"
  ],
  outputPath: './output/video.mp4',
  format: 'reels'
});

console.log(`Video rendered: ${videoPath}`);
```

### 4. Auto-Publishing to Social Media

**Facebook Page posting:**

```typescript
// src/lib/social/facebook.ts
import axios from 'axios';

interface FacebookPostParams {
  message: string;
  pageId: string;
  videoPath?: string;
}

export async function postToFacebookPage(
  params: FacebookPostParams
): Promise<string> {
  const accessToken = process.env.FACEBOOK_PAGE_TOKEN!;
  
  if (params.videoPath) {
    // Post video
    const formData = new FormData();
    formData.append('access_token', accessToken);
    formData.append('description', params.message);
    formData.append('source', fs.createReadStream(params.videoPath));
    
    const response = await axios.post(
      `https://graph.facebook.com/v18.0/${params.pageId}/videos`,
      formData,
      { headers: formData.getHeaders() }
    );
    
    return response.data.id;
  } else {
    // Post text
    const response = await axios.post(
      `https://graph.facebook.com/v18.0/${params.pageId}/feed`,
      {
        message: params.message,
        access_token: accessToken
      }
    );
    
    return response.data.id;
  }
}
```

## Complete End-to-End Pipeline

**Full automation workflow:**

```typescript
// src/workflows/full-pipeline.ts
import { scanLatestNews } from '@/lib/crawler/news-scanner';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { postToFacebookPage } from '@/lib/social/facebook';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: ToneOfVoice;
  autoPost: boolean;
  generateVideo: boolean;
  facebookPageId?: string;
}

export async function runFullPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for: ${config.keyword}`);
  
  // 1. Research
  console.log('Step 1: Scanning news...');
  const news = await scanLatestNews(config.keyword);
  const research = news.map(n => `${n.title}\n${n.content}`);
  
  // 2. Generate content
  console.log('Step 2: Generating content...');
  const content = await generateContent({
    keyword: config.keyword,
    researchData: research,
    format: config.format,
    tone: config.tone,
    language: 'en',
    aiProvider: 'claude'
  });
  
  // 3. Extract key points for video
  let videoPath: string | undefined;
  if (config.generateVideo) {
    console.log('Step 3: Rendering video...');
    
    // Use AI to extract key points
    const keyPoints = await extractKeyPoints(content);
    
    videoPath = await renderContentVideo({
      title: config.keyword,
      keyPoints,
      outputPath: `./output/${Date.now()}.mp4`,
      format: 'reels'
    });
  }
  
  // 4. Auto-post
  if (config.autoPost && config.facebookPageId) {
    console.log('Step 4: Publishing to Facebook...');
    
    const postId = await postToFacebookPage({
      message: content.substring(0, 500) + '...',
      pageId: config.facebookPageId,
      videoPath
    });
    
    console.log(`Published: ${postId}`);
  }
  
  return {
    content,
    videoPath,
    sourcesUsed: news.length
  };
}

// Helper to extract key points using AI
async function extractKeyPoints(content: string): Promise<string[]> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 1024,
    messages: [{
      role: 'user',
      content: `Extract 5 key bullet points from this content:\n\n${content}`
    }]
  });
  
  const response = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
    
  return response.split('\n').filter(line => line.trim().startsWith('-'));
}
```

**Run the pipeline:**

```typescript
const result = await runFullPipeline({
  keyword: "AI content automation trends 2024",
  format: 'toplist',
  tone: 'expert',
  generateVideo: true,
  autoPost: true,
  facebookPageId: 'your_page_id'
});
```

## API Routes (Next.js)

**Create API endpoint for pipeline:**

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runFullPipeline } from '@/workflows/full-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runFullPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo ?? false,
      autoPost: body.autoPost ?? false,
      facebookPageId: body.facebookPageId
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

**Call from client:**

```typescript
const response = await fetch('/api/pipeline', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI automation',
    format: 'how-to',
    tone: 'friendly',
    generateVideo: true,
    autoPost: false
  })
});

const result = await response.json();
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting and retry logic
async function callWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

If Remotion fails with memory errors:

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Missing Environment Variables

Add validation at startup:

```typescript
// src/lib/config/validate-env.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnvironment() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use queue systems** (Bull, BullMQ) for video rendering jobs
3. **Implement content review** before auto-posting
4. **Monitor API usage** to stay within quotas
5. **Version your prompts** for reproducible results
6. **Test video templates** before production rendering
