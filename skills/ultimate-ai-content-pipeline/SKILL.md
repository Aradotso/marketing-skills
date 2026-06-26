---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - automate content creation from research to video
  - generate blog posts and videos from keywords
  - crawl news sources and create content automatically
  - use AI to write multilingual content with video
  - set up automated content pipeline with Remotion
  - create content workflow from research to publication
  - build AI-powered marketing content system
  - generate social media videos from articles automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill helps you use the Ultimate AI Content Pipeline - a TypeScript-based automated content creation system that researches topics, generates scripts/articles, and produces videos using AI (Claude 3, OpenAI) and Remotion.

## What It Does

The pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual Output**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Multi-Platform Export**: Outputs optimized videos for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion (optional)
REMOTION_LICENSE_KEY=your_remotion_license_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNewsArticles } from '@/lib/crawler/news-crawler';

// Crawl recent news for a topic
async function researchTopic(keyword: string) {
  const articles = await crawlNewsArticles({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 20
  });
  
  return articles;
}

// Example usage
const aiNews = await researchTopic('artificial intelligence');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(topic: string, researchData: any[], format: string) {
  const prompt = `
    Based on the following research data, create a ${format} article about ${topic}.
    Research data: ${JSON.stringify(researchData)}
    
    Requirements:
    - Use data-backed insights
    - Include specific examples and statistics
    - Write in an engaging, professional tone
    - Create both English and Vietnamese versions
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].text;
}
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentVariations(baseContent: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a content marketing expert. Create engaging variations of content for different platforms.'
      },
      {
        role: 'user',
        content: `Create 3 variations of this content for social media:\n\n${baseContent}`
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

async function generateVideo(articleContent: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: {
      title: articleContent.title,
      points: articleContent.keyPoints,
      duration: 30, // seconds
    },
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'output', `video-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps,
  });

  return outputPath;
}
```

## Complete Workflow Example

```typescript
import { crawlNewsArticles } from '@/lib/crawler/news-crawler';
import { generateArticle } from '@/lib/ai/claude-generator';
import { generateVideo } from '@/lib/video/remotion-renderer';
import { publishToSocial } from '@/lib/social/publisher';

async function fullContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('📡 Starting research...');
    const research = await crawlNewsArticles({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
    });

    // Step 2: Generate Content
    console.log('🧠 Generating article...');
    const article = await generateArticle(keyword, research, 'toplist');
    
    // Step 3: Create Video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      title: article.title,
      keyPoints: article.points,
      duration: 30,
    });

    // Step 4: Publish
    console.log('📤 Publishing content...');
    await publishToSocial({
      platforms: ['facebook', 'linkedin', 'twitter'],
      content: article.summary,
      media: videoPath,
    });

    return {
      success: true,
      article,
      video: videoPath,
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
fullContentPipeline('AI automation tools').then(result => {
  console.log('✅ Pipeline completed:', result);
});
```

## Next.js API Routes

### Create Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { fullContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();
    
    const result = await fullContentPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNewsArticles } from '@/lib/crawler/news-crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();
  
  const articles = await crawlNewsArticles({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeRange: timeRange || '24h',
  });
  
  return NextResponse.json({ articles });
}
```

## Content Format Templates

```typescript
// Define content format types
type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';

// Format-specific prompts
const formatPrompts: Record<ContentFormat, string> = {
  'toplist': `Create a listicle with numbered points, each backed by data and examples.`,
  'pov': `Write from a specific perspective or opinion, using first-person narrative.`,
  'case-study': `Analyze a real-world example with problem, solution, and results.`,
  'how-to': `Provide step-by-step instructions with actionable tips.`,
};

async function generateByFormat(
  topic: string,
  format: ContentFormat,
  research: any[]
) {
  const basePrompt = formatPrompts[format];
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: `${basePrompt}\n\nTopic: ${topic}\nResearch: ${JSON.stringify(research)}`
      }
    ],
  });

  return message.content[0].text;
}
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Access the UI at http://localhost:3000

# Run content generation script
npm run generate -- --keyword "AI trends"

# Render videos only
npm run render-videos
```

## Remotion Video Templates

```typescript
// remotion/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ArticleVideo: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div style={{ opacity }}>
        <h1 style={{ color: 'white', fontSize: 48 }}>{title}</h1>
        <ul style={{ color: 'white', fontSize: 24, marginTop: 40 }}>
          {points.map((point, i) => (
            <li key={i} style={{ marginBottom: 20 }}>
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
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
}
```

### Video Rendering Errors

```typescript
// Check Remotion dependencies
if (!process.env.REMOTION_LICENSE_KEY) {
  console.warn('Remotion license key not found. Some features may be limited.');
}

// Ensure FFmpeg is installed
import { ensureFfmpeg } from '@remotion/renderer';
await ensureFfmpeg();
```

### Memory Issues

```typescript
// Process content in batches
async function processBatch(items: any[], batchSize = 5) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => processItem(item))
    );
    results.push(...batchResults);
  }
  
  return results;
}
```

## Best Practices

1. **Always validate API keys** before starting the pipeline
2. **Cache research results** to avoid redundant API calls
3. **Use streaming responses** for real-time content generation feedback
4. **Implement proper error handling** at each pipeline stage
5. **Store generated content** in a database for reuse
6. **Monitor API usage** to stay within rate limits
7. **Test video rendering** locally before production deployment
