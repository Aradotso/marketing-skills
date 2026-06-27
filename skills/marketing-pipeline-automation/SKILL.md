---
name: marketing-pipeline-automation
description: AI-powered content pipeline for automated research, scriptwriting, posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from blog posts automatically
  - scrape news and create content pipeline
  - build AI content automation system
  - create multi-language marketing content
  - auto-generate social media videos
  - set up content research and posting workflow
  - integrate Remotion for video generation
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with an automated content pipeline that handles research, scriptwriting, multilingual content generation, and video creation. The system crawls news sources, generates content in multiple formats using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is a TypeScript-based automation system that:

- **Auto-researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to)
- **Produces bilingual output** (English and Vietnamese)
- **Renders videos** automatically using Remotion
- **Manages posting** workflows for social media platforms

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following variables:

```env
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_LAMBDA_ACCESS_KEY_ID=your_aws_access_key
REMOTION_LAMBDA_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Core Architecture

The pipeline typically consists of:

1. **Research Module** - Crawls and analyzes data sources
2. **Content Generation Module** - Uses AI to create content
3. **Video Rendering Module** - Converts content to video using Remotion
4. **Publishing Module** - Handles distribution

## Key Usage Patterns

### 1. Research and Data Crawling

```typescript
import { NewsResearcher } from './modules/research';

const researcher = new NewsResearcher({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h'
});

// Fetch trending topics
const trendingTopics = await researcher.getTrendingTopics({
  keyword: 'AI marketing',
  limit: 10
});

// Get detailed insights
const insights = await researcher.analyzeTopics(trendingTopics);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Generate a ${format} article about ${topic}. 
                Include data-backed insights and make it engaging.
                Output in both English and Vietnamese.`
    }]
  });

  return message.content;
}

// Generate different formats
const toplist = await generateContent('AI tools', 'toplist');
const caseStudy = await generateContent('Marketing automation', 'case-study');
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateScript(topic: string, language: string = 'en') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in marketing.'
      },
      {
        role: 'user',
        content: `Write a compelling script about ${topic} in ${language}`
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
import { webpackOverride } from './webpack-override';

interface VideoProps {
  title: string;
  content: string[];
  duration: number;
}

async function renderVideo(props: VideoProps) {
  const bundleLocation = await bundle({
    entryPoint: './src/video/index.tsx',
    webpackOverride,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: props,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${props.title}.mp4`,
    inputProps: props,
  });
}

// Render video from content
await renderVideo({
  title: 'AI Marketing Trends',
  content: ['Point 1', 'Point 2', 'Point 3'],
  duration: 30
});
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from './pipeline';

const pipeline = new ContentPipeline({
  openaiKey: process.env.OPENAI_API_KEY,
  claudeKey: process.env.ANTHROPIC_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY,
});

async function runFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await pipeline.research({
      keyword,
      sources: ['techcrunch', 'twitter'],
      depth: 'detailed'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      research,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'professional'
    });

    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const video = await pipeline.renderVideo({
      content,
      aspectRatio: '9:16', // For Reels/TikTok
      style: 'modern'
    });

    // Step 4: Schedule posting
    console.log('📅 Scheduling posts...');
    await pipeline.schedulePost({
      content,
      video,
      platforms: ['facebook', 'instagram', 'tiktok'],
      schedule: new Date(Date.now() + 3600000) // 1 hour from now
    });

    return { content, video };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
await runFullPipeline('AI automation tools 2024');
```

### 6. Multi-Format Content Generation

```typescript
interface ContentFormat {
  type: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
}

async function generateMultiFormat(topic: string, formats: ContentFormat[]) {
  const results = await Promise.all(
    formats.map(async (format) => {
      const prompt = buildPrompt(topic, format);
      
      const content = await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 3000,
        messages: [{ role: 'user', content: prompt }]
      });

      return {
        format,
        content: content.content[0].text,
        timestamp: new Date()
      };
    })
  );

  return results;
}

function buildPrompt(topic: string, format: ContentFormat): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list with detailed explanations',
    'pov': 'Write from a personal perspective with insights',
    'case-study': 'Analyze real examples with data and outcomes',
    'how-to': 'Provide step-by-step instructions'
  };

  return `
    Topic: ${topic}
    Format: ${formatInstructions[format.type]}
    Language: ${format.language}
    Tone: ${format.tone}
    
    Include data-backed insights and real-world examples.
  `;
}
```

### 7. Remotion Video Component Example

```tsx
// src/video/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  backgroundColor?: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  backgroundColor = '#1a1a1a'
}) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor }}>
      {/* Title sequence */}
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity: titleOpacity 
        }}>
          <h1 style={{ fontSize: 60, color: 'white', textAlign: 'center' }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Content points */}
      {points.map((point, index) => (
        <Sequence 
          key={index} 
          from={90 + index * 60} 
          durationInFrames={60}
        >
          <ContentPoint text={point} index={index + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const ContentPoint: React.FC<{ text: string; index: number }> = ({ text, index }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15], [0, 1]);
  const scale = interpolate(frame, [0, 15], [0.8, 1]);

  return (
    <AbsoluteFill style={{ 
      justifyContent: 'center', 
      alignItems: 'center',
      opacity,
      transform: `scale(${scale})`
    }}>
      <div style={{ fontSize: 40, color: 'white', maxWidth: '80%' }}>
        <span style={{ color: '#00ff88', fontWeight: 'bold' }}>
          {index}.
        </span>{' '}
        {text}
      </div>
    </AbsoluteFill>
  );
};
```

## CLI Commands

If the project includes CLI tools:

```bash
# Run research on a topic
npm run research -- --keyword "AI marketing" --sources techcrunch,twitter

# Generate content
npm run generate -- --topic "AI automation" --format toplist --lang en,vi

# Render video
npm run render -- --input content.json --output video.mp4 --ratio 9:16

# Run full pipeline
npm run pipeline -- --keyword "marketing trends" --auto-post
```

## API Integration Patterns

### Rate Limiting and Error Handling

```typescript
import { retry } from './utils/retry';

async function safeApiCall<T>(
  apiCall: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  return retry(async () => {
    try {
      return await apiCall();
    } catch (error: any) {
      if (error.status === 429) {
        console.log('Rate limited, waiting...');
        await new Promise(resolve => setTimeout(resolve, 60000));
        throw error; // Retry
      }
      if (error.status >= 500) {
        console.log('Server error, retrying...');
        throw error;
      }
      // Don't retry client errors
      throw new Error(`API Error: ${error.message}`);
    }
  }, maxRetries);
}

// Usage
const content = await safeApiCall(() => 
  anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2000,
    messages: [{ role: 'user', content: 'Generate content...' }]
  })
);
```

## Troubleshooting

### Common Issues

**API Rate Limits:**
```typescript
// Implement exponential backoff
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

async function withBackoff<T>(fn: () => Promise<T>, retries = 5): Promise<T> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === retries - 1) throw error;
      const waitTime = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1}/${retries} after ${waitTime}ms`);
      await delay(waitTime);
    }
  }
  throw new Error('Max retries exceeded');
}
```

**Remotion Rendering Timeouts:**
```typescript
// Increase timeout for complex videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: 'out/video.mp4',
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    headless: true,
  },
});
```

**Memory Issues with Large Content:**
```typescript
// Process content in batches
async function processBatch<T>(
  items: T[],
  batchSize: number,
  processor: (item: T) => Promise<any>
) {
  const results = [];
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(batch.map(processor));
    results.push(...batchResults);
    // Clear memory between batches
    if (global.gc) global.gc();
  }
  return results;
}
```

**Missing Environment Variables:**
```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

## Development Workflow

```bash
# Start development server
npm run dev

# Run tests
npm test

# Build for production
npm run build

# Type checking
npm run type-check

# Lint code
npm run lint
```

## Best Practices

1. **Always validate API responses** before processing
2. **Implement proper error boundaries** for each pipeline stage
3. **Cache research results** to avoid redundant API calls
4. **Use queues** for video rendering to manage system resources
5. **Log pipeline execution** for debugging and analytics
6. **Version your prompts** for consistent content quality
