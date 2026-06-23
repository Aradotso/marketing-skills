---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, auto-posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from blog posts automatically
  - build marketing content pipeline
  - scrape news and create content
  - auto-post to social media with AI
  - create multilingual marketing content
  - generate video content with Remotion
  - research and write articles with Claude
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Automation is a comprehensive TypeScript-based system that automates the entire content creation workflow from research to publication. It combines AI models (Claude 3, OpenAI), web scraping for real-time data, automatic content generation in multiple formats and languages, and video rendering via Remotion.

**Key capabilities:**
- Auto-scrape recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- Generate content in multiple formats (listicles, POV, case studies, how-to)
- Multilingual output (English & Vietnamese)
- Automatic video/infographic rendering
- Direct publishing to social platforms

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

Create a `.env.local` file in the project root:

```env
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Data Sources (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Social Media APIs (optional)
FACEBOOK_ACCESS_TOKEN=your_facebook_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion compositions
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── package.json
```

## Core Usage Patterns

### 1. Research & Data Scraping

```typescript
import { researchTopic } from '@/lib/scraper/research';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
    language: 'en'
  });

  return {
    articles: research.articles,
    insights: research.insights,
    dataPoints: research.statistics
  };
}

// Usage
const data = await gatherInsights('AI automation');
console.log(`Found ${data.articles.length} recent articles`);
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
  const prompt = `Create a ${format} article about "${topic}" in ${language}.
Use the following research data: ${JSON.stringify(researchData)}

Requirements:
- ${format === 'toplist' ? 'List format with 5-10 items' : ''}
- ${format === 'pov' ? 'Personal perspective with strong opinions' : ''}
- Include data-backed insights
- ${language === 'vi' ? 'Write in Vietnamese' : 'Write in English'}
- Tone: professional yet engaging`;

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
}

// Generate bilingual content
const enContent = await generateContent('Marketing AI Tools', 'toplist', 'en');
const viContent = await generateContent('Marketing AI Tools', 'toplist', 'vi');
```

### 3. OpenAI Integration (Alternative)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(topic: string, style: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a marketing content expert. Generate ${style} content.`
      },
      {
        role: 'user',
        content: `Write about: ${topic}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion.config';

async function generateVideo(contentData: ContentData) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: contentData.title,
      points: contentData.keyPoints,
      style: contentData.visualStyle
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${contentData.slug}.mp4`,
    inputProps: {
      title: contentData.title,
      points: contentData.keyPoints,
    },
  });

  return `out/${contentData.slug}.mp4`;
}
```

### 5. Full Pipeline Automation

```typescript
import { researchTopic } from '@/lib/scraper/research';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { publishToSocial } from '@/lib/social/publisher';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  autoPublish: boolean;
  platforms: ('facebook' | 'linkedin' | 'twitter')[];
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate content in multiple languages
  console.log('✍️ Generating content...');
  const contents = await Promise.all(
    config.languages.map(lang =>
      generateContent(
        config.keyword,
        config.format,
        lang,
        research
      )
    )
  );

  // Step 3: Generate video (if enabled)
  let videoPath: string | null = null;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await generateVideo({
      title: config.keyword,
      keyPoints: research.insights.slice(0, 5),
      visualStyle: 'modern'
    });
  }

  // Step 4: Auto-publish (if enabled)
  if (config.autoPublish) {
    console.log('📤 Publishing to platforms...');
    for (const platform of config.platforms) {
      await publishToSocial({
        platform,
        content: contents[0], // Use primary language
        media: videoPath ? [videoPath] : [],
        scheduledTime: new Date()
      });
    }
  }

  return {
    research,
    contents,
    videoPath,
    published: config.autoPublish
  };
}

// Example usage
const result = await runContentPipeline({
  keyword: 'AI Marketing Tools 2026',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  autoPublish: true,
  platforms: ['facebook', 'linkedin']
});
```

### 6. Next.js API Routes

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo || false,
      autoPublish: false,
      platforms: []
    });

    return NextResponse.json({
      success: true,
      data: result
    });
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
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npm run remotion

# Type checking
npm run type-check

# Linting
npm run lint
```

## Common Workflows

### Workflow 1: Quick Article Generation

```typescript
// Quick script to generate article
import { generateContent } from '@/lib/ai/claude';

const article = await generateContent(
  'Top 10 Marketing Automation Tools',
  'toplist',
  'en',
  { insights: [], articles: [], statistics: [] }
);

console.log(article);
```

### Workflow 2: Scheduled Content Creation

```typescript
import cron from 'node-cron';

// Run pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Starting daily content generation...');
  
  const topics = ['AI Marketing', 'SEO Trends', 'Social Media Tips'];
  
  for (const topic of topics) {
    await runContentPipeline({
      keyword: topic,
      format: 'pov',
      languages: ['en', 'vi'],
      generateVideo: false,
      autoPublish: true,
      platforms: ['facebook']
    });
  }
});
```

### Workflow 3: Batch Video Generation

```typescript
const contentList = [
  { title: '5 Marketing Mistakes', points: [...] },
  { title: 'How to Use AI in Marketing', points: [...] },
  { title: 'Social Media Strategy 2026', points: [...] }
];

const videos = await Promise.all(
  contentList.map(content => generateVideo(content))
);

console.log(`Generated ${videos.length} videos`);
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries reached');
}

// Usage
const content = await retryWithBackoff(() =>
  generateContent('topic', 'toplist', 'en', research)
);
```

### Video Rendering Issues

```typescript
// Check Remotion license and dependencies
import { getVideoMetadata } from '@remotion/renderer';

async function validateVideoSetup() {
  try {
    // Test basic rendering capability
    const metadata = await getVideoMetadata({
      serveUrl: './src/remotion',
      composition: 'TestComposition'
    });
    console.log('✅ Remotion setup valid');
    return true;
  } catch (error) {
    console.error('❌ Remotion setup error:', error.message);
    return false;
  }
}
```

### Memory Issues with Large Batches

```typescript
// Process content in chunks
async function processBatch<T>(
  items: T[],
  processor: (item: T) => Promise<any>,
  chunkSize = 5
) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(processor)
    );
    results.push(...chunkResults);
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }
  
  return results;
}
```

## Best Practices

1. **Always validate research data** before feeding to AI models
2. **Cache expensive API calls** to reduce costs
3. **Use TypeScript types** for all content configurations
4. **Test video compositions** before batch rendering
5. **Monitor API usage** to stay within rate limits
6. **Store generated content** in a database for reuse
7. **Implement proper error handling** for all external API calls
