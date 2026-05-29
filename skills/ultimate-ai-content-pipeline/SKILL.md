---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline from research to script generation to video creation using Claude, OpenAI, and Remotion
triggers:
  - how do I use the AI content pipeline to generate content
  - set up automated content research and video generation
  - create AI-powered marketing content with this pipeline
  - generate videos from text using Remotion integration
  - automate content creation from research to video
  - configure Claude and OpenAI for content generation
  - use the marketing pipeline to create social media content
  - build automated content workflow with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content creation pipeline: from researching trending topics across TechCrunch, a16z, Twitter/X, and LinkedIn, to generating scripts with AI (Claude/OpenAI), and finally rendering videos with Remotion. This system automates up to 90% of the content creation workflow.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major news sources (last 24h)
- **AI Content Generation**: Creates multiple content formats (toplists, POVs, case studies, how-tos) in multiple languages
- **Video Rendering**: Automatically generates infographics and short-form videos optimized for Reels, TikTok, and Shorts
- **Multi-Platform**: Integrates with OpenAI, Anthropic Claude, and RapidAPI for flexible content generation

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

```env
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here
RAPIDAPI_KEY=your_rapidapi_key_here

# Content Sources
TECHCRUNCH_API_KEY=your_key_here
TWITTER_BEARER_TOKEN=your_token_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── lib/              # Core utilities
│   │   ├── ai/           # AI integrations (Claude, OpenAI)
│   │   ├── crawlers/     # Content crawlers
│   │   ├── video/        # Remotion video generation
│   │   └── utils/        # Helper functions
│   ├── components/       # React components
│   └── types/            # TypeScript definitions
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Key Usage Patterns

### 1. Content Research & Crawling

```typescript
import { researchTopic } from '@/lib/crawlers/research';

async function gatherContent(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 10
  });

  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    dataPoints: research.dataPoints
  };
}

// Usage
const content = await gatherContent('AI automation');
console.log(`Found ${content.articles.length} articles`);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(topic, format, language);

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

function buildPrompt(topic: string, format: string, language: string): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with rankings and explanations',
    'pov': 'Write a personal perspective piece with strong opinions',
    'case-study': 'Analyze a real-world example with data and outcomes',
    'how-to': 'Create a step-by-step tutorial with actionable instructions'
  };

  return `Topic: ${topic}
Format: ${formatInstructions[format]}
Language: ${language === 'vi' ? 'Vietnamese' : 'English'}
Tone: Professional yet engaging
Include: Data points, insights, and actionable takeaways`;
}
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithGPT(
  researchData: any,
  format: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content creator specializing in ${format} articles.`
      },
      {
        role: 'user',
        content: `Create content based on this research:\n${JSON.stringify(researchData)}`
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
import path from 'path';

async function generateVideo(
  contentScript: string,
  outputPath: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const { width, height } = dimensions[platform];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      script: contentScript,
      platform
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      script: contentScript,
      platform
    },
  });

  return outputPath;
}

// Usage
await generateVideo(
  'Your content script here',
  './output/video.mp4',
  'reels'
);
```

### 5. Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/crawlers/research';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h'
    });

    // Step 2: Generate content (both languages)
    console.log('✍️ Generating content...');
    const [contentEN, contentVI] = await Promise.all([
      generateContentWithClaude(keyword, 'toplist', 'en'),
      generateContentWithClaude(keyword, 'toplist', 'vi')
    ]);

    // Step 3: Generate videos for multiple platforms
    console.log('🎬 Rendering videos...');
    const videos = await Promise.all([
      generateVideo(contentEN, `./output/${keyword}-reels.mp4`, 'reels'),
      generateVideo(contentEN, `./output/${keyword}-tiktok.mp4`, 'tiktok'),
      generateVideo(contentVI, `./output/${keyword}-shorts-vi.mp4`, 'shorts')
    ]);

    return {
      research,
      content: { en: contentEN, vi: contentVI },
      videos,
      status: 'success'
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runContentPipeline('AI marketing automation').then(result => {
  console.log('✅ Pipeline completed:', result.videos);
});
```

### 6. Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages, platforms } = await request.json();

    // Validate input
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    // Run pipeline
    const result = await runContentPipeline(keyword);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('API error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Development Workflow

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run production server
npm start

# Render Remotion video locally
npx remotion render remotion/index.ts ContentVideo output.mp4

# Preview Remotion composition
npx remotion preview remotion/index.ts
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(fn: Function, maxRetries = 3) {
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

### Remotion Memory Issues

```typescript
// Reduce frame rate or resolution for large videos
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo',
  inputProps: {
    script: contentScript,
    fps: 30, // Reduce from 60 to 30
    scale: 0.8 // Render at 80% resolution
  },
});
```

### Content Quality Issues

```typescript
// Add content validation
function validateContent(content: string): boolean {
  const checks = {
    minLength: content.length > 500,
    hasHeadings: /#{1,3}\s/.test(content),
    hasData: /\d+%|\$\d+|\d+x/.test(content),
    notEmpty: content.trim().length > 0
  };

  return Object.values(checks).every(Boolean);
}

// Use in pipeline
const content = await generateContentWithClaude(keyword, format, language);
if (!validateContent(content)) {
  throw new Error('Generated content does not meet quality standards');
}
```

### Environment Variable Issues

```typescript
// Validate environment variables at startup
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

// Call at app initialization
validateEnv();
```

## Best Practices

1. **Batch Processing**: Process multiple keywords in parallel for efficiency
2. **Caching**: Cache research results to avoid redundant API calls
3. **Error Handling**: Always wrap AI calls in try-catch with proper logging
4. **Content Review**: Implement a review step before auto-publishing
5. **Rate Limiting**: Respect API rate limits with proper throttling
6. **Version Control**: Track generated content versions for A/B testing

## Common Patterns

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run pipeline daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  for (const topic of trendingTopics) {
    await runContentPipeline(topic);
  }
});
```

### Multi-Language Content

```typescript
const languages = ['en', 'vi'];
const contents = await Promise.all(
  languages.map(lang => 
    generateContentWithClaude(keyword, format, lang)
  )
);
```

### Platform-Specific Videos

```typescript
const platforms = ['reels', 'tiktok', 'shorts'] as const;
const videos = await Promise.all(
  platforms.map(platform =>
    generateVideo(content, `./output/${platform}.mp4`, platform)
  )
);
```
