---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI
  - generate marketing videos automatically
  - research and write content with Claude
  - create social media content pipeline
  - build automated content workflow
  - generate videos from text with Remotion
  - crawl news and create content
  - automate content research and writing
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with an automated content pipeline that handles research, scriptwriting, and video generation. The system crawls news sources, generates content in multiple formats using Claude/OpenAI, and renders videos using Remotion.

## What It Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for latest news (24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, YouTube Shorts

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

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # News crawling & data extraction
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research Pipeline

```typescript
import { researchTopics } from '@/lib/research/crawler';

// Crawl news sources for a specific keyword
async function fetchLatestNews(keyword: string) {
  const results = await researchTopics({
    keywords: [keyword],
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    limit: 20
  });
  
  return results;
}

// Example usage
const aiNews = await fetchLatestNews('artificial intelligence');
console.log(`Found ${aiNews.length} articles`);
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
  const prompts = {
    'toplist': `Write a top 10 list article about ${topic}`,
    'pov': `Write a point-of-view opinion piece about ${topic}`,
    'case-study': `Write a detailed case study analyzing ${topic}`,
    'how-to': `Write a comprehensive how-to guide about ${topic}`
  };

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `${prompts[format]}. Write in ${language === 'vi' ? 'Vietnamese' : 'English'}. Include data and insights from recent news.`
    }]
  });

  return message.content[0].text;
}

// Example usage
const article = await generateContent('AI trends 2024', 'toplist', 'en');
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, systemPrompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}

// Example usage
const content = await generateWithGPT(
  'Write about the latest AI developments',
  'You are an expert tech journalist writing for a marketing audience'
);
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(
  contentData: {
    title: string;
    points: string[];
    theme: string;
  }
) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: contentData,
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'output',
    `video-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: contentData,
  });

  return outputLocation;
}

// Example usage
const videoPath = await generateVideo({
  title: 'Top 5 AI Tools for Marketers',
  points: ['Tool 1', 'Tool 2', 'Tool 3', 'Tool 4', 'Tool 5'],
  theme: 'modern'
});
```

## Complete Pipeline Example

```typescript
import { researchTopics } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { publishToSocial } from '@/lib/publishing';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topics...');
    const research = await researchTopics({
      keywords: [keyword],
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
      limit: 10
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const englishContent = await generateContent(
      keyword,
      'toplist',
      'en',
      research
    );
    
    const vietnameseContent = await generateContent(
      keyword,
      'toplist',
      'vi',
      research
    );

    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(englishContent);

    // Step 4: Generate Video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      title: `Top ${keyPoints.length} ${keyword} Insights`,
      points: keyPoints,
      theme: 'tech'
    });

    // Step 5: Publish (optional)
    console.log('📤 Publishing...');
    await publishToSocial({
      content: englishContent,
      video: videoPath,
      platforms: ['facebook', 'linkedin', 'twitter']
    });

    return {
      englishContent,
      vietnameseContent,
      videoPath,
      research
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
const result = await runContentPipeline('AI marketing tools');
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const content = await generateContent(
      keyword,
      format || 'toplist',
      language || 'en'
    );

    return NextResponse.json({ content });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopics } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const { keywords, sources, timeRange } = await request.json();

    const results = await researchTopics({
      keywords: keywords || [],
      sources: sources || ['techcrunch', 'a16z'],
      timeRange: timeRange || '24h',
      limit: 20
    });

    return NextResponse.json({ results });
  } catch (error) {
    return NextResponse.json(
      { error: 'Research failed' },
      { status: 500 }
    );
  }
}
```

### Video Generation Endpoint

```typescript
// app/api/video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/remotion';

export async function POST(request: NextRequest) {
  try {
    const { title, points, theme } = await request.json();

    const videoPath = await generateVideo({
      title,
      points,
      theme: theme || 'modern'
    });

    return NextResponse.json({ videoPath });
  } catch (error) {
    return NextResponse.json(
      { error: 'Video generation failed' },
      { status: 500 }
    );
  }
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev
# or
yarn dev

# Server runs on http://localhost:3000
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const content = await generateContent(keyword, 'toplist', 'en');
      return { keyword, content };
    })
  );
  
  return results;
}

// Generate content for multiple topics
const batch = await generateBatchContent([
  'AI marketing',
  'content automation',
  'video marketing'
]);
```

### Pattern 2: Content Scheduling

```typescript
import { schedule } from '@/lib/scheduler';

async function scheduleContentGeneration(
  keyword: string,
  publishDate: Date
) {
  const job = await schedule({
    taskType: 'generateAndPublish',
    data: { keyword },
    runAt: publishDate
  });

  return job.id;
}

// Schedule for tomorrow at 9 AM
const tomorrow = new Date();
tomorrow.setDate(tomorrow.getDate() + 1);
tomorrow.setHours(9, 0, 0, 0);

await scheduleContentGeneration('AI trends', tomorrow);
```

### Pattern 3: Multi-language Pipeline

```typescript
async function generateMultilingualContent(topic: string) {
  const languages = ['en', 'vi'] as const;
  
  const content = await Promise.all(
    languages.map(async (lang) => {
      const text = await generateContent(topic, 'toplist', lang);
      return { language: lang, content: text };
    })
  );

  return Object.fromEntries(
    content.map(({ language, content }) => [language, content])
  );
}

// Get content in both languages
const multilingual = await generateMultilingualContent('AI tools');
console.log(multilingual.en); // English version
console.log(multilingual.vi); // Vietnamese version
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = baseDelay * Math.pow(2, i);
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries reached');
}

// Usage
const content = await retryWithBackoff(() =>
  generateContent('AI trends', 'toplist', 'en')
);
```

### Issue: Video Rendering Fails

```typescript
// Add error handling and fallback
async function safeGenerateVideo(data: any) {
  try {
    return await generateVideo(data);
  } catch (error) {
    console.error('Video generation failed:', error);
    
    // Fallback: generate static image instead
    return await generateStaticImage(data);
  }
}
```

### Issue: Research Returns No Results

```typescript
// Implement fallback sources
async function robustResearch(keyword: string) {
  const primarySources = ['techcrunch', 'a16z'];
  const fallbackSources = ['hackernews', 'reddit'];
  
  let results = await researchTopics({
    keywords: [keyword],
    sources: primarySources,
    timeRange: '24h'
  });
  
  if (results.length === 0) {
    console.log('No results from primary sources, trying fallback...');
    results = await researchTopics({
      keywords: [keyword],
      sources: fallbackSources,
      timeRange: '48h' // Expand time range
    });
  }
  
  return results;
}
```

### Issue: Memory Issues with Large Video Renders

```typescript
// Process videos in chunks
async function renderVideoInChunks(segments: any[]) {
  const CHUNK_SIZE = 5;
  const results = [];
  
  for (let i = 0; i < segments.length; i += CHUNK_SIZE) {
    const chunk = segments.slice(i, i + CHUNK_SIZE);
    const videos = await Promise.all(
      chunk.map(segment => generateVideo(segment))
    );
    results.push(...videos);
    
    // Allow garbage collection between chunks
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  return results;
}
```

## Best Practices

1. **Always validate API keys** before running the pipeline
2. **Cache research results** to avoid redundant API calls
3. **Use rate limiting** when working with external APIs
4. **Store generated content** in a database for reuse
5. **Monitor video rendering** as it's resource-intensive
6. **Implement error notifications** for production pipelines
7. **Use queue systems** (Bull, BullMQ) for long-running tasks

## Additional Resources

- [Anthropic Claude Documentation](https://docs.anthropic.com/)
- [OpenAI API Documentation](https://platform.openai.com/docs/)
- [Remotion Documentation](https://www.remotion.dev/docs/)
- [Next.js API Routes](https://nextjs.org/docs/api-routes/introduction)
