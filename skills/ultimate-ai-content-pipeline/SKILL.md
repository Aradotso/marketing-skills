---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline for research, script writing, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation from research to video
  - generate multi-format content with AI
  - create automated marketing content pipeline
  - build content automation with Claude and OpenAI
  - set up auto-research and video generation
  - integrate Remotion for video content automation
  - create AI-powered content workflow
  - automate social media content generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from research and script generation to automated video rendering.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-crawls news sources** (TechCrunch, a16z, Twitter, LinkedIn) for real-time data
- **Generates multi-format content** (Top Lists, POV, Case Studies, How-To guides) using Claude 3 and OpenAI
- **Renders videos automatically** using Remotion for social media platforms
- **Supports multilingual output** (English and Vietnamese)
- **Customizes tone and voice** for different audiences
- **Exports optimized content** for Reels, TikTok, and Shorts

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

## Configuration

Create a `.env` file in the project root:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for content crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if needed)
DATABASE_URL=your_database_url

# Remotion for video rendering
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Content crawling logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { analyzeContent } from '@/lib/ai/content-analyzer';

async function researchTopic(keyword: string) {
  // Crawl recent news from multiple sources
  const newsData = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Analyze and extract insights using AI
  const insights = await analyzeContent({
    data: newsData,
    analysisType: 'deep',
    extractStats: true
  });

  return insights;
}

// Usage
const insights = await researchTopic('AI automation');
console.log(insights.keyPoints);
console.log(insights.statistics);
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/generator/content-generator';

async function createArticle(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format: format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'dual', // 'en' | 'vi' | 'dual'
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    aiProvider: 'claude', // 'claude' | 'openai'
    includeStats: true,
    wordCount: 1500
  });

  return content;
}

// Usage with Claude
const article = await createArticle('Marketing Automation', 'toplist');
console.log(article.english);
console.log(article.vietnamese);
```

### 3. OpenAI Integration

```typescript
import { OpenAI } from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer creating engaging social media content.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### 4. Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
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

// Advanced usage with system prompt
async function generateMarketingContent(topic: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    system: `You are a marketing expert who creates viral content. 
             Always include data-backed insights and actionable takeaways.`,
    messages: [
      {
        role: 'user',
        content: `Create a compelling top 10 list about ${topic} with recent statistics.`
      }
    ]
  });

  return message.content[0].text;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/VideoTemplate';

async function generateVideo(content: any) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      stats: content.statistics
    }
  });

  // Render video
  const outputLocation = `./public/videos/${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.inputProps
  });

  return outputLocation;
}
```

### 6. Complete Pipeline Workflow

```typescript
import { runContentPipeline } from '@/lib/pipeline/main-pipeline';

async function automateContentCreation(keyword: string) {
  const result = await runContentPipeline({
    keyword,
    steps: {
      research: true,
      contentGeneration: true,
      videoRendering: true,
      autoPublish: false // Set to true for auto-posting
    },
    config: {
      contentFormat: 'toplist',
      languages: ['en', 'vi'],
      videoFormats: ['reels', 'tiktok', 'shorts'],
      aiProvider: 'claude'
    }
  });

  return {
    research: result.researchData,
    content: result.generatedContent,
    videos: result.renderedVideos,
    publishUrls: result.publishedUrls
  };
}

// Usage
const output = await automateContentCreation('AI Marketing Tools 2024');
console.log('Content created:', output.content.english);
console.log('Videos generated:', output.videos);
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateBilingualContent(topic: string) {
  const englishContent = await generateContent({
    topic,
    language: 'en',
    aiProvider: 'claude'
  });

  const vietnameseContent = await generateContent({
    topic,
    language: 'vi',
    aiProvider: 'claude',
    context: englishContent // Use English as context for consistency
  });

  return { englishContent, vietnameseContent };
}
```

### Batch Content Processing

```typescript
async function batchGenerateContent(topics: string[]) {
  const results = await Promise.all(
    topics.map(async (topic) => {
      const research = await researchTopic(topic);
      const content = await generateContent({
        topic,
        format: 'toplist',
        insights: research
      });
      const video = await generateVideo(content);
      
      return { topic, content, video };
    })
  );

  return results;
}
```

### Error Handling & Retry Logic

```typescript
async function generateContentWithRetry(
  topic: string,
  maxRetries: number = 3
) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const content = await generateContent({ topic });
      return content;
    } catch (error) {
      console.error(`Attempt ${attempt + 1} failed:`, error);
      
      if (attempt === maxRetries - 1) {
        throw new Error(`Failed after ${maxRetries} attempts`);
      }
      
      // Wait before retrying (exponential backoff)
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, attempt) * 1000)
      );
    }
  }
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion studio for video template editing
npm run remotion

# Render a specific video composition
npm run render -- --composition=ContentVideo --props='{"title":"My Video"}'

# Type checking
npm run type-check

# Lint code
npm run lint
```

## API Routes

The Next.js application exposes these API endpoints:

### POST /api/research
```typescript
// Request
{
  "keyword": "AI automation",
  "sources": ["techcrunch", "twitter"],
  "timeframe": "24h"
}

// Response
{
  "insights": [...],
  "statistics": [...],
  "sources": [...]
}
```

### POST /api/generate
```typescript
// Request
{
  "topic": "Marketing Automation",
  "format": "toplist",
  "language": "dual",
  "tone": "professional"
}

// Response
{
  "english": "...",
  "vietnamese": "...",
  "metadata": {...}
}
```

### POST /api/render-video
```typescript
// Request
{
  "contentId": "abc123",
  "format": "reels",
  "resolution": "1080x1920"
}

// Response
{
  "videoUrl": "https://...",
  "duration": 45,
  "size": "12MB"
}
```

## Troubleshooting

### API Key Issues
```typescript
// Validate API keys on startup
function validateEnvVars() {
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
```

### Rate Limiting
```typescript
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

async function processMultipleTopics(topics: string[]) {
  const results = await Promise.all(
    topics.map(topic => 
      limit(() => generateContent({ topic }))
    )
  );
  
  return results;
}
```

### Video Rendering Memory Issues
```typescript
// Optimize Remotion rendering for large projects
const renderConfig = {
  concurrency: 1, // Reduce concurrent renders
  frameRange: [0, 300], // Limit frame range if needed
  scale: 0.75, // Reduce scale for faster rendering
  verbose: true
};
```

### Content Quality Issues
```typescript
// Add validation layer
function validateContent(content: string): boolean {
  const checks = {
    minLength: content.length > 500,
    hasStats: /\d+%|\d+ (users|people|companies)/.test(content),
    hasStructure: content.includes('#') || content.includes('##'),
    noPlaceholders: !content.includes('[insert') && !content.includes('TODO')
  };
  
  return Object.values(checks).every(check => check);
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement retry logic** for AI API calls
3. **Cache research results** to avoid redundant crawling
4. **Validate generated content** before rendering videos
5. **Use TypeScript types** for all data structures
6. **Monitor API usage** to stay within rate limits
7. **Optimize video rendering** by choosing appropriate settings
8. **Test content quality** with multiple AI providers
