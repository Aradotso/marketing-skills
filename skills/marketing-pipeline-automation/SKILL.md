---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline that researches, generates scripts, and produces videos automatically using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate marketing videos from text automatically
  - build content pipeline with Claude and OpenAI
  - create automated marketing workflow
  - set up AI content research and video generation
  - implement automated content publishing system
  - build AI-powered content automation
  - create end-to-end marketing content pipeline
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the **Ultimate AI Content Pipeline** — a complete automation system that handles content research, script generation, and video production using AI (Claude 3, OpenAI) and Remotion for video rendering.

## What This Project Does

The Marketing Pipeline Share is an end-to-end content automation system that:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn
2. **AI Script Generation**: Creates multi-format content (toplist, POV, case studies, how-to) in multiple languages using Claude/OpenAI
3. **Video Rendering**: Automatically converts text content into videos and infographics using Remotion
4. **Multi-Platform Optimization**: Generates content optimized for Reels, TikTok, Shorts, and other platforms

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
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if persisting content)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_CONCURRENCY=4
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping & research
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utilities
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Content Research Module

```typescript
import { ResearchService } from '@/lib/crawler/research-service';

// Initialize research service
const researcher = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Research a topic
async function researchTopic(keyword: string) {
  const insights = await researcher.scanLatestNews({
    keyword,
    timeframe: '24h',
    language: 'en'
  });
  
  return {
    articles: insights.articles,
    trends: insights.trends,
    dataPoints: insights.statistics
  };
}

// Example usage
const aiInsights = await researchTopic('artificial intelligence');
console.log(`Found ${aiInsights.articles.length} relevant articles`);
```

### 2. AI Content Generation

```typescript
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Claude integration for content generation
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateContentWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const prompt = buildPrompt(research, format);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].text;
}

// OpenAI integration as alternative
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateContentWithGPT(
  research: any,
  format: string,
  language: 'en' | 'vi'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator. Generate ${format} content in ${language}.`
      },
      {
        role: 'user',
        content: JSON.stringify(research)
      }
    ],
    temperature: 0.7
  });
  
  return completion.choices[0].message.content;
}
```

### 3. Multi-Language Content Generation

```typescript
interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone: 'professional' | 'friendly' | 'humorous';
}

async function generateMultiLanguageContent(request: ContentRequest) {
  // Step 1: Research
  const research = await researchTopic(request.keyword);
  
  // Step 2: Generate content for each language
  const contents = await Promise.all(
    request.languages.map(async (lang) => {
      const content = await generateContentWithClaude(research, request.format);
      
      // Translate if needed
      if (lang === 'vi') {
        return await translateContent(content, 'en', 'vi');
      }
      
      return content;
    })
  );
  
  return {
    en: contents[0],
    vi: contents[1] || null
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
  style: 'infographic' | 'talking-head' | 'text-animation';
}

async function generateVideo(config: VideoConfig) {
  // Define video dimensions based on format
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const { width, height } = dimensions[config.format];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: config.style,
    inputProps: {
      title: config.title,
      content: config.content
    }
  });
  
  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content
    }
  });
  
  return outputLocation;
}

// Example usage
const videoPath = await generateVideo({
  content: 'AI is transforming marketing...',
  title: 'Top 5 AI Marketing Trends',
  format: 'reels',
  style: 'infographic'
});
```

### 5. Complete Pipeline Integration

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Initialize the complete pipeline
const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  videoRenderer: 'remotion',
  languages: ['en', 'vi']
});

async function runFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate scripts
    console.log('✍️ Generating content...');
    const scripts = await pipeline.generateScripts({
      research,
      formats: ['toplist', 'how-to'],
      tone: 'professional'
    });
    
    // Step 3: Create videos
    console.log('🎬 Rendering videos...');
    const videos = await pipeline.renderVideos(scripts);
    
    // Step 4: Export results
    return {
      keyword,
      research: {
        articlesFound: research.articles.length,
        insights: research.trends
      },
      content: scripts.map(s => ({
        format: s.format,
        language: s.language,
        wordCount: s.content.split(' ').length
      })),
      videos: videos.map(v => ({
        path: v.outputPath,
        duration: v.durationInSeconds,
        format: v.format
      }))
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runFullPipeline('AI marketing automation');
console.log('✅ Pipeline completed:', result);
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run pipeline every day at 6 AM
cron.schedule('0 6 * * *', async () => {
  const keywords = ['AI marketing', 'content automation', 'video marketing'];
  
  for (const keyword of keywords) {
    try {
      await runFullPipeline(keyword);
      console.log(`✅ Completed pipeline for: ${keyword}`);
    } catch (error) {
      console.error(`❌ Failed for ${keyword}:`, error);
    }
  }
});
```

### Pattern 2: Batch Processing Multiple Topics

```typescript
async function batchGenerateContent(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(topic => runFullPipeline(topic))
  );
  
  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');
  
  return {
    success: successful.length,
    failed: failed.length,
    results: successful.map(r => r.value)
  };
}
```

### Pattern 3: Custom Content Templates

```typescript
interface ContentTemplate {
  name: string;
  structure: string[];
  aiInstructions: string;
}

const templates: Record<string, ContentTemplate> = {
  'trend-analysis': {
    name: 'Trend Analysis',
    structure: ['introduction', 'data-points', 'analysis', 'predictions', 'conclusion'],
    aiInstructions: 'Analyze trends with data-backed insights and future predictions'
  },
  'quick-tips': {
    name: 'Quick Tips',
    structure: ['hook', 'tip-1', 'tip-2', 'tip-3', 'call-to-action'],
    aiInstructions: 'Create actionable tips in concise, engaging format'
  }
};

async function generateFromTemplate(
  templateName: string,
  keyword: string
) {
  const template = templates[templateName];
  const research = await researchTopic(keyword);
  
  const prompt = `
    Create content following this structure: ${template.structure.join(' -> ')}
    Instructions: ${template.aiInstructions}
    Research data: ${JSON.stringify(research)}
  `;
  
  return await generateContentWithClaude(prompt, 'custom');
}
```

## Development Workflow

### Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the UI at http://localhost:3000
```

### Testing Video Generation

```bash
# Preview Remotion compositions
npm run remotion:preview

# Render a specific composition
npm run remotion:render -- --composition=infographic
```

### Building for Production

```bash
# Build the Next.js application
npm run build

# Start production server
npm run start
```

## Troubleshooting

### Issue: AI API Rate Limits

```typescript
import pRetry from 'p-retry';

async function generateWithRetry(prompt: string) {
  return pRetry(
    async () => {
      return await generateContentWithClaude(prompt, 'toplist');
    },
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
}
```

### Issue: Video Rendering Memory Errors

```typescript
// Adjust Remotion concurrency
const config = {
  concurrency: process.env.REMOTION_CONCURRENCY || 2,
  // Lower concurrency if experiencing memory issues
};

await renderMedia({
  ...composition,
  concurrency: config.concurrency
});
```

### Issue: Research Data Quality

```typescript
// Implement content validation
function validateResearch(data: any): boolean {
  return (
    data.articles.length >= 3 &&
    data.trends.length > 0 &&
    data.statistics.length > 0
  );
}

const research = await researchTopic(keyword);
if (!validateResearch(research)) {
  throw new Error('Insufficient research data. Try different keywords.');
}
```

### Issue: Multi-Language Translation Quality

```typescript
// Use dedicated translation with validation
async function translateWithValidation(
  text: string,
  from: string,
  to: string
) {
  const translated = await translateContent(text, from, to);
  
  // Validate translation quality
  const wordCountDiff = Math.abs(
    text.split(' ').length - translated.split(' ').length
  );
  
  if (wordCountDiff > text.split(' ').length * 0.5) {
    console.warn('Translation may be inaccurate - significant length difference');
  }
  
  return translated;
}
```

## Best Practices

1. **Cache Research Data**: Store research results to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue for video generation to prevent memory overload
3. **Monitor API Usage**: Track Claude/OpenAI token usage to manage costs
4. **Version Content**: Keep versions of generated content for A/B testing
5. **Test Before Publishing**: Always preview generated videos before scheduling posts

## Advanced Configuration

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    maxArticles: 20,
    timeframe: '24h'
  },
  ai: {
    provider: 'claude',
    model: 'claude-3-opus-20240229',
    fallback: 'gpt-4-turbo-preview',
    temperature: 0.7,
    maxTokens: 4096
  },
  video: {
    defaultFormat: 'reels',
    fps: 30,
    quality: 'high',
    styles: ['infographic', 'text-animation']
  },
  output: {
    contentDir: './content',
    videoDir: './public/videos',
    saveIntermediate: true
  }
};
```

This skill enables AI coding agents to effectively use the Marketing Pipeline Automation system for end-to-end content creation, from research through video generation.
