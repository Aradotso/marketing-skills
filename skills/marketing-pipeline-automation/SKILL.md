---
name: marketing-pipeline-automation
description: AI-powered content pipeline that automates research, script writing, video generation, and social media posting with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate video content from research automatically
  - set up an AI marketing content pipeline
  - create automated blog posts and videos
  - use Claude and OpenAI for content generation
  - build a content automation system with Remotion
  - automate research to video workflow
  - create multi-format AI content pipeline
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based automation system that handles the entire content creation workflow: from research scraping to script generation to video rendering and social media publishing.

## What This Project Does

The Marketing Pipeline automates:
- **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter/X, LinkedIn for recent insights
- **AI Content Generation**: Uses Claude 3 and OpenAI to create content in multiple formats (toplist, POV, case study, how-to)
- **Multi-Language Support**: Generates content in both English and Vietnamese
- **Video Rendering**: Converts written content to video using Remotion
- **Social Media Publishing**: Auto-posts to Facebook pages and other platforms

## Installation

### Prerequisites

```bash
# Required environment
Node.js >= 18
npm or yarn
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Configure environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Social Media
FACEBOOK_ACCESS_TOKEN=your_facebook_token
FACEBOOK_PAGE_ID=your_page_id

# Video Rendering
REMOTION_LAMBDA_REGION=us-east-1
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
DATABASE_URL=your_database_connection_string
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── components/     # React/Next.js UI components
│   ├── lib/
│   │   ├── ai/        # AI integration (Claude, OpenAI)
│   │   ├── research/  # Content scraping modules
│   │   ├── render/    # Remotion video rendering
│   │   └── social/    # Social media posting
│   ├── pages/         # Next.js pages
│   └── remotion/      # Remotion video templates
├── public/
└── config/
```

## Core Features and Usage

### 1. Research & Content Scraping

```typescript
import { researchTopic } from '@/lib/research/scraper';

// Scrape recent news and insights
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });
  
  return research.insights;
}

// Example usage
const insights = await gatherResearch('AI marketing automation');
console.log(insights);
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
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = `Create a ${format} article about "${topic}" in ${language} with a ${tone} tone. 
  Use these research insights: ${JSON.stringify(insights)}`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

// Generate content
const article = await generateContent(
  'AI in Content Marketing',
  'toplist',
  'en',
  'expert'
);
```

### 3. Alternative: OpenAI Content Generation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(topic: string, research: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and technology.'
      },
      {
        role: 'user',
        content: `Write a comprehensive article about ${topic} using this research: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  content: string,
  outputPath: string
) {
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      branding: {
        logo: '/logo.png',
        colors: {
          primary: '#007bff',
          secondary: '#6c757d'
        }
      }
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.props,
  });

  return outputPath;
}

// Usage
const videoPath = await renderContentVideo(parsedArticle, './output/video.mp4');
```

### 5. Remotion Video Template Example

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  branding: any;
}> = ({ title, points, branding }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: 'white' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          fontSize: 60,
          fontWeight: 'bold',
          color: branding.colors.primary
        }}>
          {title}
        </AbsoluteFill>
      </Sequence>

      {/* Content Points */}
      {points.map((point, index) => (
        <Sequence 
          key={index}
          from={fps * (3 + index * 4)} 
          durationInFrames={fps * 4}
        >
          <AbsoluteFill style={{
            padding: 100,
            fontSize: 40,
            lineHeight: 1.5
          }}>
            <div>• {point}</div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 6. Social Media Publishing

```typescript
import axios from 'axios';

async function publishToFacebook(
  content: string,
  videoUrl?: string,
  imageUrl?: string
) {
  const pageId = process.env.FACEBOOK_PAGE_ID;
  const accessToken = process.env.FACEBOOK_ACCESS_TOKEN;

  const endpoint = videoUrl 
    ? `https://graph.facebook.com/${pageId}/videos`
    : `https://graph.facebook.com/${pageId}/feed`;

  const payload = {
    message: content,
    access_token: accessToken,
    ...(videoUrl && { file_url: videoUrl }),
    ...(imageUrl && !videoUrl && { link: imageUrl })
  };

  const response = await axios.post(endpoint, payload);
  return response.data;
}

// Schedule post
async function schedulePost(content: string, publishTime: Date) {
  const scheduledTime = Math.floor(publishTime.getTime() / 1000);
  
  return await publishToFacebook(content, undefined, undefined, {
    scheduled_publish_time: scheduledTime,
    published: false
  });
}
```

### 7. Complete Pipeline Example

```typescript
// Complete workflow from research to publishing
async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('Gathering research...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeframe: '24h'
    });

    // Step 2: Generate content
    console.log('Generating content...');
    const englishContent = await generateContent(
      keyword,
      'toplist',
      'en',
      'expert'
    );
    
    const vietnameseContent = await generateContent(
      keyword,
      'toplist',
      'vi',
      'friendly'
    );

    // Step 3: Render video
    console.log('Rendering video...');
    const parsedContent = parseArticleForVideo(englishContent);
    const videoPath = await renderContentVideo(
      parsedContent,
      `./output/${keyword}-video.mp4`
    );

    // Step 4: Publish
    console.log('Publishing to social media...');
    const facebookPost = await publishToFacebook(
      vietnameseContent,
      videoPath
    );

    return {
      success: true,
      contentEn: englishContent,
      contentVi: vietnameseContent,
      video: videoPath,
      socialPost: facebookPost
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
const result = await runContentPipeline('AI content automation 2024');
```

## Common Patterns

### Multi-Format Content Generation

```typescript
const formats = ['toplist', 'pov', 'case-study', 'how-to'] as const;

async function generateMultipleFormats(topic: string, research: any) {
  const contents = await Promise.all(
    formats.map(format => 
      generateContent(topic, format, 'en', 'expert')
    )
  );

  return formats.reduce((acc, format, idx) => {
    acc[format] = contents[idx];
    return acc;
  }, {} as Record<string, string>);
}
```

### Batch Video Rendering

```typescript
async function batchRenderVideos(articles: Array<any>) {
  const renders = articles.map((article, idx) => 
    renderContentVideo(article, `./output/video-${idx}.mp4`)
  );

  return await Promise.all(renders);
}
```

### Content Variation Testing

```typescript
async function generateVariations(topic: string) {
  const tones = ['expert', 'friendly', 'humorous'] as const;
  
  return await Promise.all(
    tones.map(tone => generateContent(topic, 'toplist', 'en', tone))
  );
}
```

## CLI Commands

```bash
# Development
npm run dev          # Start Next.js dev server
npm run build        # Build for production
npm run start        # Start production server

# Remotion
npm run remotion     # Open Remotion studio
npm run render       # Render video from CLI

# Database
npm run db:migrate   # Run database migrations
npm run db:seed      # Seed initial data
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function generateWithRateLimit(topics: string[]) {
  return await Promise.all(
    topics.map(topic => 
      limit(() => generateContent(topic, 'toplist', 'en', 'expert'))
    )
  );
}
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrency for video rendering
import { renderMedia } from '@remotion/renderer';

const renderQueue = pLimit(1); // One render at a time

async function safeRenderVideo(composition: any, output: string) {
  return renderQueue(() => renderMedia({
    composition,
    outputLocation: output,
    concurrency: 1,
    everyNthFrame: 2, // Render every 2nd frame for preview
  }));
}
```

### Claude API Timeout

```typescript
// Add retry logic
async function generateWithRetry(
  topic: string,
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(topic, 'toplist', 'en', 'expert');
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  throw new Error('Max retries reached');
}
```

### Facebook Publishing Errors

```typescript
// Validate content before publishing
function validateFacebookPost(content: string): boolean {
  if (content.length > 63206) {
    console.error('Content exceeds Facebook character limit');
    return false;
  }
  return true;
}

async function safePublish(content: string) {
  if (!validateFacebookPost(content)) {
    content = content.substring(0, 63000) + '...';
  }
  return await publishToFacebook(content);
}
```

## Best Practices

1. **Always use environment variables** for API keys and secrets
2. **Implement error handling** at each pipeline stage
3. **Cache research results** to avoid redundant API calls
4. **Test video templates** in Remotion Studio before batch rendering
5. **Monitor API usage** to stay within rate limits
6. **Version control your prompts** for consistent content generation
7. **Use TypeScript strict mode** for better type safety
