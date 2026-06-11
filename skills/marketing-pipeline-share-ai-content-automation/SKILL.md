---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, script generation, and video creation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up AI content pipeline for social media
  - generate videos from written content automatically
  - create multi-language content with Claude and OpenAI
  - build automated marketing content workflow
  - research and generate social media posts with AI
  - use remotion for automatic video generation
  - crawl news sources for content ideas
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research, content generation, and video rendering. It crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for trending topics, generates multi-format content in multiple languages using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls real-time data from major tech news sources and social platforms
- **AI Content Generation**: Creates articles in various formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese simultaneously
- **Video Rendering**: Converts written content into video format using Remotion
- **Platform Optimization**: Exports videos in aspect ratios suitable for Reels, TikTok, Shorts

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

Create a `.env` file with the following variables:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video preview
npm run remotion:preview

# Render video
npm run remotion:render
```

## Key API Usage Patterns

### 1. Content Research Service

```typescript
import { researchContent } from '@/services/research';

async function fetchLatestNews(keyword: string) {
  const researchData = await researchContent({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 10
  });
  
  return researchData;
}

// Example usage
const insights = await fetchLatestNews('AI automation');
console.log(insights.articles); // Array of crawled articles
console.log(insights.trends); // Extracted trends and insights
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
  const systemPrompt = `You are an expert content creator specializing in ${format} articles. 
  Write in ${language === 'en' ? 'English' : 'Vietnamese'}.`;
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Create a ${format} article about: ${topic}`
      }
    ]
  });
  
  return message.content[0].text;
}

// Example: Generate bilingual content
const enContent = await generateContent('AI Content Marketing', 'toplist', 'en');
const viContent = await generateContent('AI Content Marketing', 'toplist', 'vi');
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, tone: 'expert' | 'friendly' | 'humorous') {
  const toneInstructions = {
    expert: 'Use professional, authoritative language with data-backed insights.',
    friendly: 'Write in a conversational, approachable style.',
    humorous: 'Include wit and engaging storytelling elements.'
  };
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creator. ${toneInstructions[tone]}`
      },
      {
        role: 'user',
        content: prompt
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
import path from 'path';

async function renderContentVideo(
  content: {
    title: string;
    points: string[];
    bgColor: string;
  },
  outputPath: string
) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content,
  });
  
  return outputPath;
}

// Example usage
await renderContentVideo({
  title: '5 AI Content Trends in 2024',
  points: [
    'AI-powered research automation',
    'Multi-language content generation',
    'Automated video creation',
    'Real-time trend analysis',
    'Cross-platform optimization'
  ],
  bgColor: '#1a1a2e'
}, './output/video.mp4');
```

### 5. Remotion Component Example

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  bgColor: string;
}> = ({ title, points, bgColor }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        padding: 60, 
        fontFamily: 'Arial, sans-serif',
        color: 'white' 
      }}>
        <h1 style={{ 
          opacity: titleOpacity,
          fontSize: 72,
          marginBottom: 40
        }}>
          {title}
        </h1>
        
        {points.map((point, index) => {
          const pointOpacity = interpolate(
            frame,
            [30 + (index * 20), 50 + (index * 20)],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <div 
              key={index}
              style={{
                opacity: pointOpacity,
                fontSize: 36,
                marginBottom: 20,
                transform: `translateX(${interpolate(
                  frame,
                  [30 + (index * 20), 50 + (index * 20)],
                  [-50, 0],
                  { extrapolateRight: 'clamp' }
                )}px)`
              }}
            >
              {index + 1}. {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

## Common Workflow Pattern

```typescript
// Complete content pipeline
async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchContent({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h'
    });
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const contentPrompt = `Based on this research: ${JSON.stringify(research.insights)}
    Create a toplist article about ${keyword}`;
    
    const content = await generateContent(contentPrompt, 'toplist', 'en');
    
    // Step 3: Extract key points for video
    const videoData = {
      title: `Top 5 Insights: ${keyword}`,
      points: research.insights.slice(0, 5),
      bgColor: '#0f0f23'
    };
    
    // Step 4: Render video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo(
      videoData,
      `./output/${keyword.replace(/\s+/g, '-')}.mp4`
    );
    
    console.log('✅ Pipeline complete!');
    return {
      content,
      videoPath,
      research
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
runContentPipeline('AI Marketing Automation')
  .then(result => console.log('Success:', result))
  .catch(err => console.error('Failed:', err));
```

## Next.js API Route Example

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/ai/content-generator';
import { researchContent } from '@/lib/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, language } = req.body;
  
  try {
    // Research phase
    const research = await researchContent({ keyword });
    
    // Generation phase
    const content = await generateContent(
      keyword,
      format || 'toplist',
      language || 'en'
    );
    
    res.status(200).json({
      success: true,
      data: {
        content,
        sources: research.sources,
        insights: research.insights
      }
    });
    
  } catch (error) {
    console.error('Content generation error:', error);
    res.status(500).json({ 
      error: 'Failed to generate content',
      message: error.message 
    });
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const waitTime = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Waiting ${waitTime}ms...`);
        await new Promise(resolve => setTimeout(resolve, waitTime));
      } else {
        throw error;
      }
    }
  }
}
```

### Remotion Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

### Missing Environment Variables

```typescript
// Validate env vars on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}

validateEnv();
```

## Tips for AI Agents

1. **Choose the right AI provider**: Use Claude for longer, more nuanced content; OpenAI for faster, structured outputs
2. **Batch video rendering**: Render multiple videos in parallel using worker threads to optimize performance
3. **Cache research data**: Store crawled data with timestamps to avoid redundant API calls within 24h
4. **Implement queues**: Use Bull or BullMQ for handling large-scale content generation jobs
5. **Monitor token usage**: Track API costs by logging token consumption per request
