---
name: ultimate-ai-content-pipeline
description: Automated content generation system from research to video using Claude, OpenAI, and Remotion
triggers:
  - how do I generate automated content with AI pipeline
  - set up ultimate content automation system
  - create content from research to video automatically
  - use Claude and OpenAI for content generation
  - automate social media content creation end-to-end
  - generate videos from articles using Remotion
  - build AI content pipeline with research and scripting
  - create automated marketing content workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating scripts in multiple formats (Toplist, POV, Case Study, How-to), to rendering videos automatically. It integrates Claude 3, OpenAI, and Remotion to transform a single keyword into publication-ready content across multiple platforms.

## What This Project Does

This is an end-to-end content automation pipeline that:

- **Auto-researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Generates content** in multiple formats and languages (English/Vietnamese) using Claude or OpenAI
- **Renders videos** and infographics automatically using Remotion
- **Optimizes output** for various social platforms (Reels, TikTok, Shorts)

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

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping API
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Content Settings
DEFAULT_LANGUAGE=vi
DEFAULT_TONE=professional
DEFAULT_FORMAT=toplist

# Remotion Video Settings
REMOTION_CODEC=h264
VIDEO_WIDTH=1080
VIDEO_HEIGHT=1920
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping & research
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key API Patterns

### 1. Research & Content Scraping

```typescript
import { scrapeNewsArticles } from '@/lib/research/scraper';
import { analyzeInsights } from '@/lib/research/analyzer';

async function gatherResearch(keyword: string) {
  // Scrape latest articles from multiple sources
  const articles = await scrapeNewsArticles({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 10
  });

  // Extract and analyze insights
  const insights = await analyzeInsights(articles);
  
  return {
    articles,
    insights,
    trends: insights.trends,
    dataPoints: insights.statistics
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  research: any,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    language: 'en' | 'vi';
    tone: 'expert' | 'friendly' | 'casual';
  }
) {
  const prompt = `Based on this research data:
${JSON.stringify(research, null, 2)}

Create a ${options.format} article in ${options.language} with a ${options.tone} tone.
Include specific data points and actionable insights.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(
  research: any,
  format: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and social media.'
      },
      {
        role: 'user',
        content: `Create a ${format} article based on: ${JSON.stringify(research)}`
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

async function renderContentVideo(
  content: {
    title: string;
    points: string[];
    stats: Array<{ label: string; value: string }>;
  }
) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });

  // Render video
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
    inputProps: content,
  });

  return outputLocation;
}
```

## Complete Content Pipeline Example

```typescript
import { gatherResearch } from '@/lib/research';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/render';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await gatherResearch(keyword);

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContentWithClaude(research, {
      format: 'toplist',
      language: 'vi',
      tone: 'expert'
    });

    // Parse content into video-friendly format
    const videoData = parseContentForVideo(content);

    // Step 3: Render video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo(videoData);

    return {
      success: true,
      content,
      videoPath,
      research: research.insights
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Helper function
function parseContentForVideo(content: string) {
  // Extract title, key points, and statistics
  const lines = content.split('\n');
  const title = lines[0].replace(/^#\s*/, '');
  const points = lines
    .filter(line => line.match(/^\d+\./))
    .map(line => line.replace(/^\d+\.\s*/, ''));
  
  return { title, points, stats: [] };
}
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: Request) {
  const { keyword, format, language } = await request.json();

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }

  try {
    const result = await runContentPipeline(keyword);
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
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

# Render Remotion video (standalone)
npx remotion render src/index.tsx ContentVideo output.mp4
```

## Common Content Formats

### Toplist Format

```typescript
const toplistPrompt = {
  structure: [
    'Engaging headline',
    'Brief introduction',
    '5-10 numbered items with descriptions',
    'Data points for each item',
    'Conclusion with CTA'
  ],
  example: 'Top 10 AI Tools Revolutionizing Marketing in 2024'
};
```

### POV (Point of View) Format

```typescript
const povPrompt = {
  structure: [
    'Personal hook/story',
    'Controversial or unique perspective',
    'Supporting evidence',
    'Counter-arguments addressed',
    'Strong conclusion'
  ],
  example: 'Why Traditional Marketing Is Dead (And What To Do Instead)'
};
```

### Case Study Format

```typescript
const caseStudyPrompt = {
  structure: [
    'Company/situation background',
    'Problem statement',
    'Solution implemented',
    'Results with metrics',
    'Key takeaways'
  ],
  example: 'How Company X Increased Conversions by 300% Using AI'
};
```

## Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{ 
          color: 'white', 
          fontSize: 60,
          textAlign: 'center',
          padding: 40
        }}>
          {title}
        </div>
      </Sequence>

      {points.map((point, index) => (
        <Sequence 
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <div style={{ 
            color: 'white',
            fontSize: 40,
            padding: 40
          }}>
            {index + 1}. {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

```bash
# Install required Remotion dependencies
npm install @remotion/cli @remotion/renderer @remotion/bundler

# If ffmpeg issues occur
npx remotion install
```

### Memory Issues with Large Content

```typescript
// Stream large content generation
async function generateLargeContent(research: any) {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: 'Generate...' }]
  });

  let fullText = '';
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta') {
      fullText += chunk.delta.text;
    }
  }
  
  return fullText;
}
```

### Environment Variable Not Found

```typescript
// Add validation at startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
}

validateEnv();
```
