---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to script to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create content from keyword to video automatically
  - use the marketing pipeline to generate posts
  - automate content creation with AI research
  - build content workflow with Claude and Remotion
  - set up automated video generation from articles
  - configure the content automation pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use the Ultimate AI Content Pipeline - a complete automated content creation system that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** latest news from sources like TechCrunch, a16z, Twitter/X, LinkedIn (last 24 hours)
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports bilingual** output (English & Vietnamese) with customizable tone
- **Renders videos** and infographics automatically using Remotion
- **Optimizes for platforms** like Reels, TikTok, Shorts

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Optional: Database
DATABASE_URL=your_database_connection_string

# Optional: Social Media APIs for posting
FACEBOOK_ACCESS_TOKEN=your_facebook_token
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run remotion:render
```

## Core API & Usage Patterns

### 1. Research & Content Crawling

The pipeline automatically fetches recent content from news sources:

```typescript
import { crawlNewsContent } from './lib/research/crawler';
import { analyzeContent } from './lib/research/analyzer';

async function researchTopic(keyword: string) {
  // Crawl from multiple sources
  const crawledData = await crawlNewsContent({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 50
  });

  // Analyze and extract insights
  const insights = await analyzeContent(crawledData, {
    extractStats: true,
    identifyTrends: true,
    findQuotes: true
  });

  return insights;
}
```

### 2. AI Content Generation with Claude

Generate content in various formats using Claude:

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  insights: any[]
) {
  const prompt = buildContentPrompt(topic, format, insights);

  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
  });

  return message.content[0].text;
}

function buildContentPrompt(topic: string, format: string, insights: any[]) {
  const insightsSummary = insights.map(i => 
    `- ${i.title}: ${i.summary} (Source: ${i.source})`
  ).join('\n');

  return `You are an expert content creator. Create a ${format} article about "${topic}".

Research data from the last 24 hours:
${insightsSummary}

Requirements:
- Use data-backed insights and statistics
- ${format === 'toplist' ? 'Create a numbered list format' : ''}
- ${format === 'pov' ? 'Write from a unique perspective' : ''}
- ${format === 'case-study' ? 'Include real examples and outcomes' : ''}
- ${format === 'how-to' ? 'Provide step-by-step actionable guidance' : ''}
- Engaging, professional tone
- Include relevant quotes if available

Generate the complete article now:`;
}
```

### 3. Bilingual Content Generation

Generate content in both English and Vietnamese:

```typescript
async function generateBilingualContent(
  topic: string,
  format: string,
  insights: any[]
) {
  const englishContent = await generateContent(topic, format, insights);

  // Translate to Vietnamese with cultural adaptation
  const vietnamesePrompt = `Translate the following article to Vietnamese. 
Adapt it culturally for Vietnamese readers while maintaining the core message and data.

Original article:
${englishContent}

Provide natural, engaging Vietnamese translation:`;

  const vietnameseMessage = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: vietnamesePrompt }],
  });

  return {
    english: englishContent,
    vietnamese: vietnameseMessage.content[0].text,
  };
}
```

### 4. Video Generation with Remotion

Transform content into video format:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateContentVideo(
  content: {
    title: string;
    keyPoints: string[];
    stats: Array<{ label: string; value: string }>;
  },
  outputPath: string
) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      stats: content.stats,
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      stats: content.stats,
    },
  });

  return outputPath;
}
```

### 5. Complete Pipeline Workflow

End-to-end content generation pipeline:

```typescript
import { crawlNewsContent, analyzeContent } from './lib/research';
import { generateBilingualContent } from './lib/ai/claude';
import { generateContentVideo } from './lib/video/remotion';
import { postToFacebook } from './lib/social/facebook';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log(`[1/5] Researching: ${keyword}...`);
    const crawledData = await crawlNewsContent({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
    });

    const insights = await analyzeContent(crawledData);

    // Step 2: Generate content
    console.log('[2/5] Generating content...');
    const content = await generateBilingualContent(
      keyword,
      'toplist',
      insights
    );

    // Step 3: Extract video-ready data
    console.log('[3/5] Preparing video data...');
    const videoData = extractVideoData(content.english);

    // Step 4: Render video
    console.log('[4/5] Rendering video...');
    const videoPath = await generateContentVideo(
      videoData,
      `./output/${keyword}-video.mp4`
    );

    // Step 5: Auto-post (optional)
    console.log('[5/5] Publishing...');
    await postToFacebook({
      message: content.vietnamese,
      videoPath,
    });

    return {
      content,
      videoPath,
      insights,
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function extractVideoData(content: string) {
  // Parse content to extract title, key points, stats
  const lines = content.split('\n');
  const title = lines[0].replace(/^#\s*/, '');
  
  const keyPoints = lines
    .filter(line => line.match(/^\d+\./))
    .map(line => line.replace(/^\d+\.\s*/, ''));

  const stats = lines
    .filter(line => line.match(/\d+%|\$[\d,]+/))
    .map(line => {
      const match = line.match(/(.+?):\s*(.+)/);
      return match ? { label: match[1], value: match[2] } : null;
    })
    .filter(Boolean);

  return { title, keyPoints, stats };
}
```

## Configuration Patterns

### Custom Content Formats

```typescript
// lib/config/contentFormats.ts
export const contentFormats = {
  toplist: {
    minItems: 5,
    maxItems: 10,
    includeStats: true,
    includeImages: true,
  },
  pov: {
    perspective: 'expert' | 'beginner' | 'skeptic',
    tone: 'professional' | 'casual' | 'humorous',
  },
  'case-study': {
    includeProblem: true,
    includeSolution: true,
    includeResults: true,
    minExamples: 2,
  },
  'how-to': {
    minSteps: 3,
    maxSteps: 12,
    includeVisuals: true,
    difficultyLevel: 'beginner' | 'intermediate' | 'advanced',
  },
};
```

### Video Templates Configuration

```typescript
// remotion/config.ts
export const videoConfig = {
  platforms: {
    reels: {
      width: 1080,
      height: 1920,
      fps: 30,
      durationFrames: 900, // 30 seconds
    },
    tiktok: {
      width: 1080,
      height: 1920,
      fps: 30,
      durationFrames: 1800, // 60 seconds
    },
    youtube_shorts: {
      width: 1080,
      height: 1920,
      fps: 30,
      durationFrames: 1800,
    },
  },
};
```

## Common Workflows

### Daily Trend Content Generation

```typescript
async function generateDailyTrendContent() {
  const trendingTopics = await getTrendingTopics();

  for (const topic of trendingTopics) {
    await runContentPipeline(topic);
    await delay(5000); // Rate limiting
  }
}

async function getTrendingTopics(): Promise<string[]> {
  // Fetch from Twitter/X trends API
  const response = await fetch(
    'https://api.twitter.com/2/trends/place.json?id=1',
    {
      headers: {
        Authorization: `Bearer ${process.env.TWITTER_BEARER_TOKEN}`,
      },
    }
  );

  const data = await response.json();
  return data[0].trends.slice(0, 5).map(t => t.name);
}
```

### Batch Content Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runContentPipeline(keyword).catch(err => ({
        keyword,
        error: err.message,
      }))
    )
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  console.log(`Completed: ${successful.length}/${keywords.length}`);
  console.log(`Failed: ${failed.length}`);

  return { successful, failed };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function callWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429) {
        const waitTime = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Waiting ${waitTime}ms...`);
        await delay(waitTime);
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Failures

```typescript
// Add error handling and cleanup
async function safeRenderVideo(config: any) {
  let bundleLocation: string | null = null;

  try {
    bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
    });

    return await renderMedia({
      ...config,
      serveUrl: bundleLocation,
    });
  } catch (error) {
    console.error('Video rendering failed:', error);
    throw error;
  } finally {
    // Cleanup bundle
    if (bundleLocation && fs.existsSync(bundleLocation)) {
      fs.rmSync(bundleLocation, { recursive: true });
    }
  }
}
```

### Content Quality Issues

```typescript
// Validate generated content
function validateContent(content: string): boolean {
  const checks = {
    minLength: content.length > 500,
    hasHeading: /^#+\s+.+/m.test(content),
    hasMultipleParagraphs: content.split('\n\n').length > 3,
    noPlaceholders: !content.includes('[INSERT'),
  };

  const passed = Object.values(checks).every(Boolean);

  if (!passed) {
    console.warn('Content validation failed:', checks);
  }

  return passed;
}
```

## Advanced Usage

### Custom AI Prompts

```typescript
// lib/prompts/templates.ts
export const promptTemplates = {
  linkedin: (topic: string, insights: string) => `
    Create a LinkedIn post about ${topic}.
    Format: Hook → Value → Call-to-action
    Max length: 1300 characters
    Include: ${insights}
  `,
  
  twitter: (topic: string) => `
    Create a Twitter thread (5-7 tweets) about ${topic}.
    First tweet: Eye-catching hook
    Middle tweets: Value-packed insights
    Last tweet: Call to action
  `,
};
```

### Multi-Platform Export

```typescript
async function exportToAllPlatforms(content: any) {
  const platforms = ['reels', 'tiktok', 'youtube_shorts'];

  const videos = await Promise.all(
    platforms.map(platform =>
      generateContentVideo(
        content,
        `./output/${platform}-${Date.now()}.mp4`,
        videoConfig.platforms[platform]
      )
    )
  );

  return videos;
}
```

This skill provides comprehensive guidance for AI agents to effectively use the Ultimate AI Content Pipeline for automated content creation, research, and video generation.
