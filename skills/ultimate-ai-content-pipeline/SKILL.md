---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up AI content pipeline for marketing
  - generate videos from blog posts automatically
  - research and write articles using Claude API
  - crawl news and create content automatically
  - build automated content workflow with Remotion
  - create multilingual marketing content with AI
  - automate social media video generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based system automates the entire content creation workflow: from researching trending topics across news sources (TechCrunch, Twitter, LinkedIn) to generating multi-format articles in multiple languages, and finally rendering videos for social media platforms using Remotion.

## What It Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** recent news and insights from major tech/business publications
- **Generates** articles in multiple formats (listicles, POV, case studies, how-tos)
- **Translates** content into English and Vietnamese simultaneously
- **Renders** videos and infographics automatically using Remotion
- **Optimizes** video output for Reels, TikTok, and YouTube Shorts

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
# AI Model APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database and Storage
DATABASE_URL=your_database_url
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key APIs and Usage

### 1. Content Research and Crawling

```typescript
import { NewsScanner } from '@/lib/crawler/scanner';

// Scan recent news for a topic
async function researchTopic(keyword: string) {
  const scanner = new NewsScanner({
    apiKey: process.env.RAPIDAPI_KEY!,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  const insights = await scanner.scan(keyword);
  
  return {
    articles: insights.articles,
    trends: insights.trends,
    stats: insights.statistics
  };
}

// Usage
const research = await researchTopic('AI automation');
console.log(`Found ${research.articles.length} relevant articles`);
```

### 2. AI Content Generation

```typescript
import { Anthropic } from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

// Using Claude for content generation
async function generateArticle(
  topic: string, 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!
  });

  const prompt = `
    Write a ${format} article about: ${topic}
    Language: ${language}
    Tone: Professional yet engaging
    Include: Data-backed insights, actionable tips
    Length: 1500-2000 words
  `;

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

// Using OpenAI as alternative
async function generateWithOpenAI(topic: string) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer for marketing and tech.'
      },
      {
        role: 'user',
        content: `Create a comprehensive article about: ${topic}`
      }
    ],
    temperature: 0.7
  });

  return completion.choices[0].message.content;
}
```

### 3. Multi-Language Content Generation

```typescript
interface ContentConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

async function generateMultilingualContent(config: ContentConfig) {
  const englishContent = await generateArticle(
    config.topic,
    config.format,
    'en'
  );

  const vietnameseContent = await generateArticle(
    config.topic,
    config.format,
    'vi'
  );

  return {
    en: {
      title: extractTitle(englishContent),
      content: englishContent,
      metadata: generateMetadata(englishContent, 'en')
    },
    vi: {
      title: extractTitle(vietnameseContent),
      content: vietnameseContent,
      metadata: generateMetadata(vietnameseContent, 'vi')
    }
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion.config';

interface VideoConfig {
  title: string;
  content: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(config: VideoConfig) {
  // Determine dimensions based on format
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const { width, height } = dimensions[config.format];

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      slides: config.content,
      backgroundColor: '#000000',
      textColor: '#ffffff'
    }
  });

  // Render video
  const outputLocation = `./output/${Date.now()}-${config.format}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps
  });

  return outputLocation;
}

// Usage
const video = await generateVideo({
  title: '5 AI Tools to Boost Your Productivity',
  content: [
    'Tool 1: ChatGPT for content creation',
    'Tool 2: Midjourney for image generation',
    'Tool 3: Notion AI for note-taking',
    'Tool 4: Jasper for copywriting',
    'Tool 5: Synthesia for video creation'
  ],
  duration: 30,
  format: 'reels'
});
```

### 5. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    formats: ['toplist', 'how-to'],
    generateVideo: true
  });

  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await pipeline.research(keyword);

    // Step 2: Generate content
    console.log('✍️ Generating articles...');
    const articles = await pipeline.generateContent({
      research,
      formats: ['toplist'],
      tone: 'professional'
    });

    // Step 3: Create videos
    console.log('🎬 Rendering videos...');
    const videos = await pipeline.generateVideos({
      articles,
      platforms: ['reels', 'tiktok', 'shorts']
    });

    // Step 4: Export results
    console.log('💾 Exporting content...');
    await pipeline.export({
      articles,
      videos,
      outputDir: './output'
    });

    return {
      success: true,
      articles: articles.length,
      videos: videos.length
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runContentPipeline('AI automation tools 2026')
  .then(result => console.log('✅ Pipeline complete:', result))
  .catch(error => console.error('❌ Pipeline failed:', error));
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run content generation pipeline
npm run generate -- --topic "your topic" --format toplist

# Render videos only
npm run render-video -- --input ./content/article.json

# Run research crawler
npm run research -- --keyword "AI trends" --sources techcrunch,twitter
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run content pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = await getTrendingTopics();
  
  for (const topic of topics) {
    await runContentPipeline(topic);
  }
});
```

### Pattern 2: Batch Processing

```typescript
async function batchGenerateContent(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(topic => runContentPipeline(topic))
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  return { successful: successful.length, failed: failed.length };
}
```

### Pattern 3: Custom Video Templates

```typescript
// remotion/templates/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  content: string[];
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <h1 style={{ opacity: Math.min(1, frame / 30) }}>
        {title}
      </h1>
      {content.map((text, i) => (
        <p key={i} style={{ 
          opacity: frame > (i + 1) * 60 ? 1 : 0 
        }}>
          {text}
        </p>
      ))}
    </AbsoluteFill>
  );
};
```

## Configuration Options

### AI Provider Configuration

```typescript
// lib/config/ai.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7
  }
};
```

### Crawler Configuration

```typescript
// lib/config/crawler.ts
export const CRAWLER_CONFIG = {
  sources: {
    techcrunch: {
      enabled: true,
      priority: 'high',
      rateLimit: 10 // requests per minute
    },
    twitter: {
      enabled: true,
      priority: 'medium',
      rateLimit: 15
    },
    linkedin: {
      enabled: true,
      priority: 'medium',
      rateLimit: 10
    }
  },
  timeRange: '24h',
  maxArticles: 50
};
```

### Video Configuration

```typescript
// remotion.config.ts
export const VIDEO_CONFIG = {
  fps: 30,
  dimensions: {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
    landscape: { width: 1920, height: 1080 }
  },
  codec: 'h264',
  quality: 'high'
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(
  fn: () => Promise<any>,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Memory Issues

```typescript
// Render videos in chunks for large content
async function renderLargeVideo(content: string[]) {
  const chunkSize = 10;
  const chunks = [];
  
  for (let i = 0; i < content.length; i += chunkSize) {
    const chunk = content.slice(i, i + chunkSize);
    const video = await generateVideo({
      title: `Part ${i / chunkSize + 1}`,
      content: chunk,
      duration: 30,
      format: 'reels'
    });
    chunks.push(video);
  }
  
  return chunks;
}
```

### Content Quality Issues

```typescript
// Validate generated content before proceeding
function validateContent(content: string): boolean {
  const minLength = 1000;
  const hasHeadings = /#{1,3}\s+.+/g.test(content);
  const hasParagraphs = content.split('\n\n').length > 3;
  
  return (
    content.length >= minLength &&
    hasHeadings &&
    hasParagraphs
  );
}

async function generateValidatedContent(topic: string) {
  let attempts = 0;
  const maxAttempts = 3;
  
  while (attempts < maxAttempts) {
    const content = await generateArticle(topic, 'toplist', 'en');
    
    if (validateContent(content)) {
      return content;
    }
    
    attempts++;
    console.log(`Content validation failed. Retry ${attempts}/${maxAttempts}`);
  }
  
  throw new Error('Failed to generate valid content');
}
```

### Missing API Keys

```typescript
// Validate environment variables on startup
function validateEnv() {
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

validateEnv();
```

## Best Practices

1. **Always validate API keys** before running the pipeline
2. **Implement rate limiting** to avoid hitting API quotas
3. **Use environment variables** for all sensitive configuration
4. **Cache research results** to avoid redundant API calls
5. **Monitor token usage** to control costs
6. **Validate generated content** before video rendering
7. **Use TypeScript types** for better code safety
8. **Handle errors gracefully** with proper retry logic
