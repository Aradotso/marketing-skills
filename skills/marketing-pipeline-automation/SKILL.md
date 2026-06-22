---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation with multi-language support
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline automation
  - generate videos from content automatically
  - create content pipeline with Claude and OpenAI
  - automate research and scriptwriting workflow
  - build AI content generation system
  - use remotion for automated video rendering
  - crawl and analyze content trends automatically
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to transform keywords into multi-format, multi-language content with automated video rendering.

## What It Does

The Marketing Pipeline Automation system provides:

- **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude and OpenAI
- **Multi-Language Support**: Generates content simultaneously in English and Vietnamese with customizable tone
- **Video Rendering**: Automatically converts written content into videos and infographics using Remotion
- **Platform Optimization**: Exports videos in formats optimized for Reels, TikTok, and Shorts

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

# Set up environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following required variables:

```bash
# AI Provider API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video rendering settings
REMOTION_TIMEOUT=300000
REMOTION_QUALITY=high

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── components/       # React/Next.js components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (OpenAI, Claude)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── pages/           # Next.js pages
│   └── remotion/        # Video composition templates
├── public/              # Static assets
└── .env                 # Environment variables
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { crawlTrendingSources } from '@/lib/crawler/research';

async function gatherResearch(keyword: string) {
  const sources = {
    techcrunch: true,
    twitter: true,
    linkedin: true,
    a16z: true
  };
  
  const research = await crawlTrendingSources({
    keyword,
    sources,
    timeframe: '24h',
    limit: 50
  });
  
  return research;
}

// Usage
const data = await gatherResearch('AI marketing tools');
console.log(data.insights, data.trends, data.statistics);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = buildPrompt(research, format, language, tone);
  
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

function buildPrompt(research: any, format: string, language: string, tone: string) {
  const languageInstructions = language === 'vi' 
    ? 'Viết bằng tiếng Việt, tự nhiên và chuyên nghiệp'
    : 'Write in English, natural and professional';
    
  const toneInstructions = {
    expert: 'Use authoritative, data-driven language',
    friendly: 'Use conversational, approachable tone',
    humorous: 'Include witty remarks and engaging humor'
  };
  
  return `
${languageInstructions}. ${toneInstructions[tone]}.

Format: ${format}

Research Data:
${JSON.stringify(research, null, 2)}

Create comprehensive content based on this research. Include:
- Compelling headline
- Data-backed insights
- Practical examples
- Clear structure with sections
- Call-to-action
`;
}
```

### 3. OpenAI Integration Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(research: any, config: ContentConfig) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${config.format} format. 
                  Write in ${config.language} with a ${config.tone} tone.`
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

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateContentVideo(
  content: string,
  title: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content,
      ...dimensions[platform]
    },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${title}-${platform}.mp4`,
    inputProps: {
      title,
      content,
    },
  });
  
  return `out/${title}-${platform}.mp4`;
}
```

### 5. Complete Pipeline Workflow

```typescript
import { crawlTrendingSources } from '@/lib/crawler/research';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { generateContentVideo } from '@/lib/video/render';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching:', keyword);
    const research = await crawlTrendingSources({
      keyword,
      sources: {
        techcrunch: true,
        twitter: true,
        linkedin: true,
        a16z: true
      },
      timeframe: '24h',
      limit: 50
    });
    
    // Step 2: Generate content in multiple languages
    console.log('✍️ Generating content...');
    const contentEN = await generateContentWithClaude(
      research,
      'toplist',
      'en',
      'expert'
    );
    
    const contentVI = await generateContentWithClaude(
      research,
      'toplist',
      'vi',
      'friendly'
    );
    
    // Step 3: Generate videos for different platforms
    console.log('🎬 Rendering videos...');
    const platforms: ('reels' | 'tiktok' | 'shorts')[] = ['reels', 'tiktok', 'shorts'];
    
    const videos = await Promise.all(
      platforms.map(platform => 
        generateContentVideo(contentEN, keyword, platform)
      )
    );
    
    return {
      research,
      content: {
        english: contentEN,
        vietnamese: contentVI
      },
      videos
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runContentPipeline('AI automation tools 2026')
  .then(result => console.log('✅ Pipeline complete:', result))
  .catch(err => console.error('❌ Pipeline failed:', err));
```

## Next.js API Routes

### Create Content Endpoint

```typescript
// pages/api/content/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, languages, platforms } = req.body;
  
  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }
  
  try {
    const result = await runContentPipeline(keyword);
    res.status(200).json(result);
  } catch (error) {
    console.error('Generation error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

### Research Endpoint

```typescript
// pages/api/research/crawl.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { crawlTrendingSources } from '@/lib/crawler/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, sources, timeframe } = req.query;
  
  try {
    const research = await crawlTrendingSources({
      keyword: keyword as string,
      sources: JSON.parse(sources as string),
      timeframe: timeframe as string || '24h',
      limit: 50
    });
    
    res.status(200).json(research);
  } catch (error) {
    res.status(500).json({ error: 'Research failed' });
  }
}
```

## CLI Commands

If the project includes CLI tools:

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Generate content from CLI
npm run generate -- --keyword "AI tools" --format toplist --lang en

# Render video only
npm run render -- --input content.json --platform reels

# Run research only
npm run research -- --keyword "marketing automation" --sources all
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLanguageContent(research: any, config: any) {
  const languages: ('en' | 'vi')[] = ['en', 'vi'];
  
  const content = await Promise.all(
    languages.map(async (lang) => ({
      language: lang,
      text: await generateContentWithClaude(
        research,
        config.format,
        lang,
        config.tone
      )
    }))
  );
  
  return Object.fromEntries(
    content.map(c => [c.language, c.text])
  );
}
```

### Batch Video Generation

```typescript
async function batchGenerateVideos(
  contents: Array<{ title: string; content: string }>,
  platforms: string[]
) {
  const jobs = contents.flatMap(content =>
    platforms.map(platform => ({
      content,
      platform: platform as 'reels' | 'tiktok' | 'shorts'
    }))
  );
  
  const results = await Promise.allSettled(
    jobs.map(job =>
      generateContentVideo(job.content.content, job.content.title, job.platform)
    )
  );
  
  return results.map((result, i) => ({
    job: jobs[i],
    status: result.status,
    output: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'Marketing automation', 'Content tools'];
  
  for (const keyword of keywords) {
    try {
      await runContentPipeline(keyword);
      console.log(`✅ Completed: ${keyword}`);
    } catch (error) {
      console.error(`❌ Failed: ${keyword}`, error);
    }
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      
      if (error.status === 429) {
        const waitTime = delay * Math.pow(2, i);
        console.log(`Rate limited. Waiting ${waitTime}ms...`);
        await new Promise(resolve => setTimeout(resolve, waitTime));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => 
  generateContentWithClaude(research, 'toplist', 'en', 'expert')
);
```

### Video Rendering Timeout

```typescript
// Increase timeout in remotion config
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo',
  inputProps,
  timeoutInMilliseconds: parseInt(process.env.REMOTION_TIMEOUT || '300000'),
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
async function processLargeContent(items: any[], chunkSize = 5) {
  const chunks = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    chunks.push(items.slice(i, i + chunkSize));
  }
  
  const results = [];
  for (const chunk of chunks) {
    const chunkResults = await Promise.all(
      chunk.map(item => runContentPipeline(item.keyword))
    );
    results.push(...chunkResults);
    
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  return results;
}
```

### API Key Validation

```typescript
function validateEnvironment() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at startup
validateEnvironment();
```

## Best Practices

1. **Always use environment variables** for API keys and sensitive data
2. **Implement error handling** with retries for external API calls
3. **Cache research data** to avoid redundant crawling within the same day
4. **Queue video rendering jobs** to prevent memory overflow
5. **Monitor API usage** to stay within rate limits and budget
6. **Version control content templates** for consistency across generations
7. **Test with small batches** before scaling to production workloads
