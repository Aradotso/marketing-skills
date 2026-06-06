---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "set up automated content pipeline with AI"
  - "create content from research to video automatically"
  - "generate articles and videos from keywords"
  - "automate content creation with Claude and OpenAI"
  - "build AI-powered content generation system"
  - "scrape news and generate content automatically"
  - "create multilingual content with video rendering"
  - "set up end-to-end content automation pipeline"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete content automation system that takes a keyword and produces research-backed articles and videos. It crawls recent news from sources like TechCrunch and Twitter, generates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI, and renders videos using Remotion.

## What It Does

1. **Auto-Research**: Crawls real-time data from news sources and social media (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats and languages (English/Vietnamese)
3. **Video Rendering**: Automatically generates infographics and short-form videos from content
4. **Multi-Platform Ready**: Outputs optimized for Reels, TikTok, Shorts

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
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database for content storage
DATABASE_URL=your_database_connection_string

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app directory
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI integration (Claude, OpenAI)
│   │   ├── research/       # Web scraping & data collection
│   │   ├── content/        # Content generation logic
│   │   └── video/          # Remotion video rendering
│   └── utils/              # Helper functions
├── public/                 # Static assets
├── remotion/              # Remotion video templates
└── package.json
```

## Core API Usage

### 1. Research Module (Data Collection)

```typescript
import { scrapeLatestNews } from '@/lib/research/scraper';
import { analyzeInsights } from '@/lib/research/analyzer';

// Scrape recent news on a topic
async function gatherResearch(keyword: string) {
  const newsData = await scrapeLatestNews({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  const insights = await analyzeInsights(newsData);
  
  return {
    rawData: newsData,
    insights: insights,
    statistics: insights.statistics,
    trends: insights.trends
  };
}
```

### 2. Content Generation with AI

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Using Claude for content generation
async function generateContentWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'casestudy' | 'howto'
) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = `
Based on this research data: ${JSON.stringify(research)}

Create a ${format} article with:
- Engaging headline
- Data-backed insights
- Actionable takeaways
- Both English and Vietnamese versions
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ]
  });

  return message.content;
}

// Using OpenAI alternative
async function generateContentWithOpenAI(
  research: any,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creator with ${tone} tone. Use research data to create engaging articles.`
      },
      {
        role: 'user',
        content: JSON.stringify(research)
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Render video from article content
async function renderContentVideo(
  content: {
    title: string;
    keyPoints: string[];
    statistics: any[];
  },
  outputPath: string
) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      stats: content.statistics,
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      stats: content.statistics,
    },
  });

  return outputPath;
}

// Optimize for different platforms
async function renderForPlatforms(content: any) {
  const platforms = [
    { name: 'reels', width: 1080, height: 1920 },
    { name: 'youtube', width: 1920, height: 1080 },
    { name: 'tiktok', width: 1080, height: 1920 },
  ];

  const outputs = await Promise.all(
    platforms.map(platform =>
      renderContentVideo(
        content,
        `./output/${platform.name}-${Date.now()}.mp4`
      )
    )
  );

  return outputs;
}
```

## Complete Pipeline Example

```typescript
import { gatherResearch } from '@/lib/research/scraper';
import { generateContentWithClaude } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await gatherResearch(keyword);

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContentWithClaude(
      research,
      'toplist'
    );

    // Parse content structure
    const parsedContent = JSON.parse(content[0].text);

    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo(
      {
        title: parsedContent.title,
        keyPoints: parsedContent.keyPoints,
        statistics: research.insights.statistics,
      },
      `./output/${keyword}-${Date.now()}.mp4`
    );

    console.log('✅ Pipeline complete!');
    return {
      article: parsedContent,
      video: videoPath,
      research: research,
    };

  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI content automation')
  .then(result => console.log('Success:', result))
  .catch(err => console.error('Failed:', err));
```

## Next.js API Routes

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    const result = await runContentPipeline(keyword);

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

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateBilingualContent(research: any) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const languages = ['english', 'vietnamese'];
  const contents = {};

  for (const lang of languages) {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: `Create article in ${lang}: ${JSON.stringify(research)}`
      }]
    });

    contents[lang] = message.content[0].text;
  }

  return contents;
}
```

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    const result = await runContentPipeline(keyword);
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Custom Content Formats

```typescript
interface ContentFormat {
  type: 'toplist' | 'pov' | 'casestudy' | 'howto';
  tone: 'expert' | 'friendly' | 'humorous';
  length: 'short' | 'medium' | 'long';
}

async function generateCustomFormat(
  research: any,
  format: ContentFormat
) {
  const prompt = `
Format: ${format.type}
Tone: ${format.tone}
Length: ${format.length}
Research: ${JSON.stringify(research)}

Create optimized content following these specifications.
  `;

  // Use appropriate AI provider
  return await generateContentWithClaude(research, format.type);
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access at http://localhost:3000
```

## Building for Production

```bash
# Build the application
npm run build

# Start production server
npm start
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
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

### Memory Issues with Video Rendering

```typescript
// Process videos in chunks
async function renderVideosInChunks(contents: any[], chunkSize = 3) {
  const chunks = [];
  for (let i = 0; i < contents.length; i += chunkSize) {
    chunks.push(contents.slice(i, i + chunkSize));
  }

  const results = [];
  for (const chunk of chunks) {
    const chunkResults = await Promise.all(
      chunk.map(content => renderContentVideo(content, `./output/${content.id}.mp4`))
    );
    results.push(...chunkResults);
    
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }

  return results;
}
```

### Research Data Quality

```typescript
// Validate and filter research data
function validateResearchData(data: any) {
  return {
    articles: data.articles.filter(a => 
      a.publishedDate && 
      a.content.length > 100 &&
      !a.isSpam
    ),
    insights: data.insights.filter(i => 
      i.confidence > 0.7
    ),
    statistics: data.statistics.filter(s => 
      s.source && s.verified
    )
  };
}
```

## Best Practices

1. **Always validate research data** before passing to AI
2. **Use environment variables** for all API keys
3. **Implement rate limiting** for API calls
4. **Cache research results** to avoid redundant scraping
5. **Monitor AI token usage** to control costs
6. **Test video outputs** across different platforms before bulk rendering
