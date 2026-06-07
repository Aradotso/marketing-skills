---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scripting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - set up automated marketing content pipeline
  - generate videos from text content automatically
  - crawl news sources for content research
  - create multi-format content with Claude API
  - build automated content workflow with Remotion
  - configure AI content generation pipeline
  - schedule automated content publishing
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Marketing Pipeline Share**, a comprehensive content automation system that handles the entire content lifecycle: from researching trending topics, generating scripts in multiple formats, to rendering videos automatically. Built with TypeScript, Next.js, and powered by Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share is an all-in-one content production pipeline that:

- **Auto-crawls news sources** (TechCrunch, a16z, Twitter/X, LinkedIn) for recent insights
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Creates multilingual content** (English & Vietnamese) with customizable tone
- **Renders videos & infographics** automatically using Remotion
- **Optimizes for platforms** (Reels, TikTok, Shorts) with proper aspect ratios

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
cp .env.example .env.local
```

## Configuration

### Environment Variables

Create a `.env.local` file with the following required variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion (Video Rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### API Provider Setup

The system supports multiple AI providers. Configure in your service files:

```typescript
// lib/ai-config.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4000,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7
  }
}
```

## Core Components

### 1. Content Research Module

Auto-crawl and analyze news sources:

```typescript
// services/research/crawler.ts
import { NewsAPI } from '@/lib/news-api'

export async function crawlRecentNews(keyword: string, hours: number = 24) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin']
  const results = []
  
  for (const source of sources) {
    const articles = await NewsAPI.fetch({
      source,
      keyword,
      timeRange: `${hours}h`,
      language: 'en'
    })
    results.push(...articles)
  }
  
  return analyzeInsights(results)
}

async function analyzeInsights(articles: Article[]) {
  const insights = {
    trends: extractTrends(articles),
    statistics: extractStats(articles),
    quotes: extractQuotes(articles),
    sources: articles.map(a => a.url)
  }
  
  return insights
}
```

### 2. Content Generation with AI

Generate content in multiple formats:

```typescript
// services/content/generator.ts
import Anthropic from '@anthropic-ai/sdk'

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
})

export async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = buildPrompt(topic, format, language, tone)
  
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  })
  
  return {
    content: message.content[0].text,
    metadata: {
      format,
      language,
      tone,
      generatedAt: new Date()
    }
  }
}

function buildPrompt(
  topic: string,
  format: string,
  language: string,
  tone: string
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with 5-10 items',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and conclusions',
    'how-to': 'Create step-by-step tutorial with actionable advice'
  }
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include light humor and engaging storytelling'
  }
  
  return `
Topic: ${topic}
Format: ${formatInstructions[format]}
Tone: ${toneInstructions[tone]}
Language: ${language === 'en' ? 'English' : 'Vietnamese'}

Please write a comprehensive article following these guidelines.
Include statistics, examples, and actionable insights.
  `.trim()
}
```

### 3. Video Generation with Remotion

Render videos from content:

```typescript
// services/video/renderer.ts
import { bundle } from '@remotion/bundler'
import { renderMedia, selectComposition } from '@remotion/renderer'
import path from 'path'

export async function renderContentVideo(
  content: string,
  options: {
    format: 'reels' | 'tiktok' | 'shorts'
    duration: number
  }
) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  )
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      format: options.format
    }
  })
  
  const aspectRatios = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'shorts': { width: 1080, height: 1920 }
  }
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${Date.now()}.mp4`,
    ...aspectRatios[options.format]
  })
}
```

### 4. Complete Pipeline Workflow

Orchestrate the entire content creation process:

```typescript
// services/pipeline/orchestrator.ts
import { crawlRecentNews } from '@/services/research/crawler'
import { generateContent } from '@/services/content/generator'
import { renderContentVideo } from '@/services/video/renderer'

export async function runContentPipeline(config: {
  keyword: string
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
  languages: ('en' | 'vi')[]
  tone: 'expert' | 'friendly' | 'humorous'
  includeVideo: boolean
}) {
  // Step 1: Research
  console.log('🔍 Researching topic...')
  const research = await crawlRecentNews(config.keyword)
  
  // Step 2: Generate content for each language
  console.log('✍️ Generating content...')
  const contents = await Promise.all(
    config.languages.map(lang =>
      generateContent(
        config.keyword,
        config.format,
        lang,
        config.tone
      )
    )
  )
  
  // Step 3: Render videos (if requested)
  if (config.includeVideo) {
    console.log('🎬 Rendering videos...')
    const videos = await Promise.all(
      contents.map(content =>
        renderContentVideo(content.content, {
          format: 'reels',
          duration: 60
        })
      )
    )
    
    return { contents, videos, research }
  }
  
  return { contents, research }
}
```

## API Routes (Next.js)

### Create Content Endpoint

```typescript
// app/api/content/create/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { runContentPipeline } from '@/services/pipeline/orchestrator'

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en', 'vi'],
      tone: body.tone || 'friendly',
      includeVideo: body.includeVideo || false
    })
    
    return NextResponse.json({
      success: true,
      data: result
    })
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 })
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { crawlRecentNews } from '@/services/research/crawler'

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  const keyword = searchParams.get('keyword')
  const hours = parseInt(searchParams.get('hours') || '24')
  
  if (!keyword) {
    return NextResponse.json({
      error: 'Keyword is required'
    }, { status: 400 })
  }
  
  const insights = await crawlRecentNews(keyword, hours)
  
  return NextResponse.json({
    success: true,
    data: insights
  })
}
```

## Common Usage Patterns

### Pattern 1: Quick Content Generation

```typescript
// Quick single content generation
import { generateContent } from '@/services/content/generator'

const article = await generateContent(
  'AI Marketing Automation',
  'toplist',
  'en',
  'expert'
)

console.log(article.content)
```

### Pattern 2: Batch Multi-Language Content

```typescript
// Generate content in multiple languages simultaneously
const languages = ['en', 'vi'] as const

const multiLangContent = await Promise.all(
  languages.map(lang =>
    generateContent('AI Trends 2024', 'pov', lang, 'friendly')
  )
)

const [englishContent, vietnameseContent] = multiLangContent
```

### Pattern 3: Research-First Workflow

```typescript
// Research before writing
const research = await crawlRecentNews('ChatGPT Enterprise', 24)

const enrichedContent = await generateContent(
  `ChatGPT Enterprise - ${research.trends.join(', ')}`,
  'case-study',
  'en',
  'expert'
)
```

### Pattern 4: Full Automation Pipeline

```typescript
// Complete automation from research to video
const pipeline = await runContentPipeline({
  keyword: 'AI Video Generation',
  format: 'how-to',
  languages: ['en', 'vi'],
  tone: 'friendly',
  includeVideo: true
})

// Access results
console.log('Articles:', pipeline.contents.length)
console.log('Videos:', pipeline.videos?.length)
console.log('Research insights:', pipeline.research.trends)
```

## CLI Commands

```bash
# Development
npm run dev          # Start Next.js dev server
npm run build        # Build for production
npm run start        # Start production server

# Remotion (Video rendering)
npm run remotion     # Open Remotion studio
npm run render       # Render specific video composition

# Type checking
npm run type-check   # Run TypeScript compiler

# Linting
npm run lint         # Run ESLint
```

## Troubleshooting

### API Rate Limits

If hitting rate limits with AI providers:

```typescript
// lib/rate-limiter.ts
import pLimit from 'p-limit'

const limit = pLimit(2) // Max 2 concurrent requests

export async function generateWithRateLimit(topics: string[]) {
  return Promise.all(
    topics.map(topic =>
      limit(() => generateContent(topic, 'toplist', 'en', 'expert'))
    )
  )
}
```

### Video Rendering Memory Issues

For large video renders:

```typescript
// Increase Node memory
// package.json
{
  "scripts": {
    "render": "NODE_OPTIONS=--max-old-space-size=4096 remotion render"
  }
}
```

### Crawling Failures

Handle network errors gracefully:

```typescript
async function safeCrawl(keyword: string) {
  try {
    return await crawlRecentNews(keyword)
  } catch (error) {
    console.error('Crawl failed:', error)
    // Fallback to cached data or manual input
    return {
      trends: [],
      statistics: [],
      quotes: [],
      sources: []
    }
  }
}
```

### Content Quality Issues

Improve AI output with better prompts:

```typescript
// Add context and constraints
const improvedPrompt = `
Context: ${research.trends.join(', ')}
Statistics: ${research.statistics.join(', ')}

Write a ${format} article about ${topic}.
Requirements:
- Include at least 3 data points
- Reference recent news (last 24h)
- Maintain ${tone} tone
- Target length: 800-1200 words
`
```

## Best Practices

1. **Always validate API keys** before running pipelines
2. **Cache research results** to avoid redundant API calls
3. **Use TypeScript strict mode** for type safety
4. **Implement retry logic** for external API calls
5. **Monitor token usage** to control costs
6. **Test video renders locally** before batch processing
7. **Version control prompts** for reproducibility

This skill enables comprehensive content automation workflows while maintaining flexibility for customization and scaling.
