---
name: marketing-pipeline-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate content from research to video automatically
  - use Claude and OpenAI for content automation
  - create AI-powered content workflow
  - build automated video content from articles
  - integrate Remotion for marketing video generation
  - automate content research and publishing
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Automation (marketing-pipeline-share) is a comprehensive TypeScript-based system that automates the entire content creation workflow: from research and article generation to automatic video rendering. It integrates Claude 3, OpenAI, and Remotion to create a complete content production pipeline.

**Key capabilities:**
- Auto-crawl and analyze real-time data from news sources (TechCrunch, Twitter, LinkedIn)
- Generate articles in multiple formats (Toplist, POV, Case Study, How-to)
- Multi-language support (English & Vietnamese)
- Automatic video and infographic generation using Remotion
- Next.js interface for easy content management

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
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research crawlers
│   │   ├── generators/  # Content generators
│   │   └── remotion/    # Video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/           # Remotion video templates
```

## Core API Usage

### 1. Research Module

Crawl and analyze real-time content from various sources:

```typescript
import { researchContent } from '@/lib/research/crawler';

async function gatherResearch(keyword: string) {
  const research = await researchContent({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 10
  });

  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    statistics: research.statistics
  };
}

// Usage
const data = await gatherResearch('AI marketing automation');
```

### 2. Content Generation with Claude

Generate articles using Claude API:

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(research: any, format: string) {
  const prompt = `
Based on the following research data, create a ${format} article:
${JSON.stringify(research, null, 2)}

Format: ${format}
Language: English and Vietnamese
Tone: Professional yet engaging
Include: Statistics, insights, and actionable takeaways
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].text;
}

// Usage
const article = await generateArticle(researchData, 'case-study');
```

### 3. Content Generation with OpenAI

Alternative using OpenAI:

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(research: any, options: {
  format: string;
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
}) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content writer. Generate ${options.format} content in ${options.language} with a ${options.tone} tone.`
      },
      {
        role: 'user',
        content: `Create content based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

Create videos from article content:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(articleData: {
  title: string;
  keyPoints: string[];
  statistics: any[];
}) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: articleData,
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'public/videos', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: articleData,
  });

  return outputPath;
}

// Usage
const videoPath = await generateVideo({
  title: 'Top 5 AI Marketing Tools',
  keyPoints: ['Tool 1', 'Tool 2', 'Tool 3'],
  statistics: [{ label: 'ROI', value: '300%' }]
});
```

## Complete Pipeline Example

End-to-end content creation workflow:

```typescript
import { researchContent } from '@/lib/research/crawler';
import { generateArticle } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/remotion/render';
import { publishContent } from '@/lib/publishing/auto-post';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching content...');
    const research = await researchContent({
      keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h',
      maxResults: 15
    });

    // Step 2: Generate Article (English)
    console.log('✍️ Generating English article...');
    const articleEN = await generateArticle(research, {
      format: 'toplist',
      language: 'en',
      tone: 'professional'
    });

    // Step 3: Generate Article (Vietnamese)
    console.log('✍️ Generating Vietnamese article...');
    const articleVI = await generateArticle(research, {
      format: 'toplist',
      language: 'vi',
      tone: 'professional'
    });

    // Step 4: Extract key points for video
    const keyPoints = extractKeyPoints(articleEN);
    
    // Step 5: Generate Video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      title: keyword,
      keyPoints: keyPoints.slice(0, 5),
      statistics: research.statistics
    });

    // Step 6: Publish (optional)
    console.log('📤 Publishing content...');
    await publishContent({
      articles: [
        { language: 'en', content: articleEN },
        { language: 'vi', content: articleVI }
      ],
      video: videoPath,
      platforms: ['facebook', 'linkedin']
    });

    return {
      success: true,
      articles: { en: articleEN, vi: articleVI },
      video: videoPath
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runContentPipeline('AI content automation 2024');
```

## Content Format Types

Define article formats:

```typescript
type ContentFormat = 
  | 'toplist'      // Top 10 lists, rankings
  | 'pov'          // Point of view, opinion pieces
  | 'case-study'   // In-depth case studies
  | 'how-to'       // Step-by-step guides
  | 'news-digest'  // News roundups
  | 'comparison';  // Tool/product comparisons

interface ArticleConfig {
  format: ContentFormat;
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous' | 'authoritative';
  length: 'short' | 'medium' | 'long'; // 500/1500/3000 words
  includeStats: boolean;
  includeCTA: boolean;
}

const config: ArticleConfig = {
  format: 'toplist',
  language: 'en',
  tone: 'professional',
  length: 'medium',
  includeStats: true,
  includeCTA: true
};
```

## Remotion Video Templates

Example Remotion composition:

```typescript
// remotion/ArticleVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ArticleVideo: React.FC<{
  title: string;
  keyPoints: string[];
  statistics: Array<{ label: string; value: string }>;
}> = ({ title, keyPoints, statistics }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{
          justifyContent: 'center',
          alignItems: 'center',
          color: 'white',
          fontSize: 60,
          fontWeight: 'bold',
          padding: 40
        }}>
          {title}
        </AbsoluteFill>
      </Sequence>

      {/* Key Points */}
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={90 + index * 120}
          durationInFrames={120}
        >
          <AbsoluteFill style={{
            justifyContent: 'center',
            alignItems: 'center',
            color: 'white',
            fontSize: 40,
            padding: 60
          }}>
            <div>{index + 1}. {point}</div>
          </AbsoluteFill>
        </Sequence>
      ))}

      {/* Statistics */}
      <Sequence from={90 + keyPoints.length * 120} durationInFrames={150}>
        <AbsoluteFill style={{
          justifyContent: 'center',
          alignItems: 'center',
          color: 'white'
        }}>
          {statistics.map((stat, i) => (
            <div key={i} style={{ fontSize: 35, margin: 20 }}>
              {stat.label}: <strong>{stat.value}</strong>
            </div>
          ))}
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## Running the Application

### Development Mode

```bash
# Start Next.js development server
npm run dev

# Access at http://localhost:3000
```

### Production Build

```bash
# Build for production
npm run build

# Start production server
npm start
```

### Remotion Studio (for video editing)

```bash
# Open Remotion Studio
npm run remotion

# Render specific composition
npm run remotion render ArticleVideo output.mp4
```

## Common Workflows

### Workflow 1: Daily Content Automation

```typescript
import cron from 'node-cron';

// Run daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  const keywords = [
    'AI marketing trends',
    'social media automation',
    'content marketing tools'
  ];

  for (const keyword of keywords) {
    await runContentPipeline(keyword);
  }
});
```

### Workflow 2: Multi-Platform Publishing

```typescript
async function publishToMultiplePlatforms(content: {
  title: string;
  body: string;
  video?: string;
}) {
  const platforms = [
    { name: 'facebook', handler: publishToFacebook },
    { name: 'linkedin', handler: publishToLinkedIn },
    { name: 'twitter', handler: publishToTwitter }
  ];

  const results = await Promise.allSettled(
    platforms.map(platform => 
      platform.handler({
        title: content.title,
        body: content.body,
        media: content.video
      })
    )
  );

  return results;
}
```

### Workflow 3: Batch Video Generation

```typescript
async function batchGenerateVideos(articles: any[]) {
  const videos = [];

  for (const article of articles) {
    const videoPath = await generateVideo({
      title: article.title,
      keyPoints: article.keyPoints,
      statistics: article.statistics
    });
    
    videos.push({
      articleId: article.id,
      videoPath,
      platform: determineBestPlatform(article)
    });
  }

  return videos;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

async function generateMultipleArticles(topics: string[]) {
  const results = await Promise.all(
    topics.map(topic => 
      limit(() => generateArticle(topic))
    )
  );
  return results;
}
```

### Memory Issues with Video Rendering

```typescript
// Use chunked rendering for long videos
async function renderLargeVideo(segments: any[]) {
  const chunks = [];
  
  for (let i = 0; i < segments.length; i += 5) {
    const chunk = segments.slice(i, i + 5);
    const chunkPath = await renderMedia({
      composition: chunk,
      // ... config
    });
    chunks.push(chunkPath);
  }
  
  // Merge chunks
  return mergeVideoChunks(chunks);
}
```

### Claude/OpenAI Timeout Errors

```typescript
// Add retry logic
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateArticle(prompt);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

### Research Crawler Failures

```typescript
// Graceful degradation
async function robustResearch(keyword: string) {
  const sources = ['techcrunch', 'twitter', 'linkedin'];
  const results = [];

  for (const source of sources) {
    try {
      const data = await crawlSource(source, keyword);
      results.push(data);
    } catch (error) {
      console.warn(`Failed to crawl ${source}:`, error);
      // Continue with other sources
    }
  }

  if (results.length === 0) {
    throw new Error('All research sources failed');
  }

  return mergeResearchResults(results);
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement caching** for research data to reduce API calls
3. **Add error boundaries** around AI generation calls
4. **Monitor API usage** to stay within rate limits
5. **Version control video templates** separately from code
6. **Test prompts extensively** before automating
7. **Use TypeScript types** for content schemas
8. **Implement logging** for pipeline debugging
