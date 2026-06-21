---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do i use the marketing pipeline automation tool
  - set up automated content generation with AI
  - create automated video content from research
  - use remotion for video generation in content pipeline
  - configure claude and openai for content automation
  - build an ai-powered content workflow
  - automate social media content creation with AI
  - generate videos from blog posts automatically
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**marketing-pipeline-share** is an end-to-end automated content pipeline that transforms keywords into published content and videos. The system:

1. **Researches** trending topics by crawling TechCrunch, a16z, Twitter/X, LinkedIn
2. **Generates** multi-format content (toplist, POV, case study, how-to) in multiple languages
3. **Renders** videos and infographics using Remotion
4. **Publishes** to social platforms automatically

Built with TypeScript, Next.js, Claude 3, OpenAI, and Remotion.

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
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Social Media APIs (optional)
FACEBOOK_PAGE_ACCESS_TOKEN=your_token
LINKEDIN_ACCESS_TOKEN=your_token
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── research/    # Web scraping & data collection
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video templates
│   └── utils/           # Helper functions
├── public/              # Static assets
└── config/              # Configuration files
```

## Key Components

### 1. Research Module (Auto-Scan)

Automatically crawls and analyzes recent content from multiple sources:

```typescript
// src/lib/research/scanner.ts
import { searchNews } from './sources/techcrunch';
import { scrapeTweets } from './sources/twitter';
import { analyzeInsights } from '../ai/analyzer';

interface ResearchResult {
  keyword: string;
  sources: Array<{
    title: string;
    url: string;
    summary: string;
    date: string;
  }>;
  insights: string[];
  trends: string[];
}

export async function performResearch(
  keyword: string
): Promise<ResearchResult> {
  // Gather data from multiple sources
  const [newsArticles, tweets, linkedinPosts] = await Promise.all([
    searchNews(keyword),
    scrapeTweets(keyword),
    searchLinkedIn(keyword)
  ]);

  // Combine and deduplicate
  const allSources = [...newsArticles, ...tweets, ...linkedinPosts];

  // Use AI to extract insights
  const insights = await analyzeInsights(allSources, keyword);

  return {
    keyword,
    sources: allSources,
    insights: insights.key_points,
    trends: insights.trending_topics
  };
}
```

### 2. Content Generation (AI-Powered)

Generate multi-format content using Claude or OpenAI:

```typescript
// src/lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentOptions {
  keyword: string;
  research: ResearchResult;
  format: ContentFormat;
  language: Language;
  tone: Tone;
}

export async function generateContent(
  options: ContentOptions
): Promise<string> {
  const { keyword, research, format, language, tone } = options;

  const prompt = buildPrompt(keyword, research, format, language, tone);

  // Use Claude for longer, nuanced content
  if (format === 'case-study' || format === 'pov') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  // Use OpenAI for structured formats
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content writer creating ${format} content.`,
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(
  keyword: string,
  research: ResearchResult,
  format: ContentFormat,
  language: Language,
  tone: Tone
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article',
    'pov': 'Write from a unique perspective or opinion',
    'case-study': 'Analyze a specific example with data',
    'how-to': 'Provide step-by-step instructions'
  };

  return `
    Keyword: ${keyword}
    Format: ${formatInstructions[format]}
    Language: ${language === 'en' ? 'English' : 'Vietnamese'}
    Tone: ${tone}
    
    Research Data:
    ${research.sources.map(s => `- ${s.title}: ${s.summary}`).join('\n')}
    
    Key Insights:
    ${research.insights.join('\n')}
    
    Write a compelling, SEO-optimized article incorporating the research data and insights.
  `;
}
```

### 3. Video Generation (Remotion)

Automatically render videos from generated content:

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  imageUrls: string[];
  duration: number; // in seconds
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      slides: parseContentToSlides(config.content),
      images: config.imageUrls,
    },
  });

  return outputLocation;
}

function parseContentToSlides(content: string): Array<{
  text: string;
  duration: number;
}> {
  // Split content into digestible slides
  const paragraphs = content.split('\n\n').filter(p => p.trim());
  
  return paragraphs.map(paragraph => ({
    text: paragraph.trim(),
    duration: Math.max(3, paragraph.length / 20), // ~20 chars per second
  }));
}
```

```tsx
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface Slide {
  text: string;
  duration: number;
}

interface ContentVideoProps {
  title: string;
  slides: Slide[];
  images: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  slides,
  images,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  let currentTime = 0;
  const sequences = slides.map((slide, index) => {
    const from = Math.floor(currentTime * fps);
    currentTime += slide.duration;
    
    return (
      <Sequence key={index} from={from} durationInFrames={Math.floor(slide.duration * fps)}>
        <AbsoluteFill style={{ backgroundColor: '#1a1a1a', padding: 60 }}>
          {images[index] && (
            <img
              src={images[index]}
              style={{
                width: '100%',
                height: '50%',
                objectFit: 'cover',
                borderRadius: 16,
              }}
            />
          )}
          <div
            style={{
              color: 'white',
              fontSize: 48,
              fontWeight: 'bold',
              marginTop: 40,
              textAlign: 'center',
            }}
          >
            {slide.text}
          </div>
        </AbsoluteFill>
      </Sequence>
    );
  });

  return <>{sequences}</>;
};
```

### 4. Publishing Module

Automatically post to social platforms:

```typescript
// src/lib/publish/publisher.ts
interface PublishTarget {
  platform: 'facebook' | 'linkedin' | 'twitter';
  content: string;
  mediaUrl?: string;
}

export async function publishContent(target: PublishTarget): Promise<void> {
  switch (target.platform) {
    case 'facebook':
      await publishToFacebook(target.content, target.mediaUrl);
      break;
    case 'linkedin':
      await publishToLinkedIn(target.content, target.mediaUrl);
      break;
    case 'twitter':
      await publishToTwitter(target.content, target.mediaUrl);
      break;
  }
}

async function publishToFacebook(
  content: string,
  mediaUrl?: string
): Promise<void> {
  const token = process.env.FACEBOOK_PAGE_ACCESS_TOKEN;
  const pageId = process.env.FACEBOOK_PAGE_ID;

  const formData = new FormData();
  formData.append('message', content);
  if (mediaUrl) {
    formData.append('url', mediaUrl);
  }

  await fetch(`https://graph.facebook.com/v18.0/${pageId}/feed`, {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${token}`,
    },
    body: formData,
  });
}
```

## Complete Pipeline Example

```typescript
// src/lib/pipeline/execute.ts
import { performResearch } from '../research/scanner';
import { generateContent } from '../content/generator';
import { renderContentVideo } from '../video/renderer';
import { publishContent } from '../publish/publisher';

export async function executeContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await performResearch(keyword);

  // Step 2: Generate content (multiple languages)
  console.log('✍️ Generating content...');
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      research,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
    }),
    generateContent({
      keyword,
      research,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
    }),
  ]);

  // Step 3: Render video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo({
    title: keyword,
    content: englishContent,
    imageUrls: research.sources.slice(0, 5).map(s => s.imageUrl || ''),
    duration: 60,
  });

  // Step 4: Publish
  console.log('📤 Publishing...');
  await Promise.all([
    publishContent({
      platform: 'facebook',
      content: englishContent,
      mediaUrl: videoPath,
    }),
    publishContent({
      platform: 'linkedin',
      content: englishContent,
      mediaUrl: videoPath,
    }),
  ]);

  console.log('✅ Pipeline complete!');
}
```

## API Routes (Next.js)

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { executeContentPipeline } from '@/lib/pipeline/execute';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    // Execute pipeline in background
    executeContentPipeline(keyword).catch(console.error);

    return NextResponse.json({
      message: 'Pipeline started',
      keyword,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render a specific video composition
npx remotion render src/remotion/index.ts ContentVideo output.mp4
```

## Common Patterns

### Scheduling Content Generation

```typescript
// src/lib/scheduler/cron.ts
import cron from 'node-cron';
import { executeContentPipeline } from '../pipeline/execute';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = await fetchTrendingKeywords();
  
  for (const keyword of trendingKeywords) {
    await executeContentPipeline(keyword);
  }
});
```

### Custom Video Templates

```tsx
// src/remotion/templates/InfographicVideo.tsx
import { AbsoluteFill, interpolate, useCurrentFrame } from 'remotion';

export const InfographicVideo: React.FC<{
  stats: Array<{ label: string; value: number }>;
}> = ({ stats }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#fff' }}>
      {stats.map((stat, i) => {
        const progress = interpolate(
          frame,
          [i * 30, (i + 1) * 30],
          [0, stat.value]
        );

        return (
          <div key={i} style={{ padding: 20 }}>
            <h3>{stat.label}</h3>
            <div style={{ fontSize: 72 }}>{Math.floor(progress)}</div>
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for research APIs
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  keywords.map(keyword => 
    limit(() => performResearch(keyword))
  )
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
  inputProps,
});
```

### Claude/OpenAI Token Limits

```typescript
// Split long content into chunks
function chunkContent(text: string, maxTokens: number = 3000): string[] {
  const chunks: string[] = [];
  const sentences = text.split('. ');
  let currentChunk = '';

  for (const sentence of sentences) {
    if ((currentChunk + sentence).length > maxTokens * 4) {
      chunks.push(currentChunk);
      currentChunk = sentence;
    } else {
      currentChunk += sentence + '. ';
    }
  }

  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

### Environment Variables Not Loading

Ensure `.env.local` is in the project root and Next.js is restarted after changes:

```bash
# Kill all Next.js processes
pkill -f next

# Restart
npm run dev
```

## Best Practices

1. **API Key Security**: Never commit API keys. Use environment variables and `.gitignore`.
2. **Error Handling**: Wrap all API calls in try-catch blocks with proper logging.
3. **Caching**: Cache research results to avoid redundant API calls.
4. **Queue System**: Use a job queue (Bull, BullMQ) for processing multiple pipelines.
5. **Monitoring**: Log pipeline execution metrics for optimization.
