---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline with research scraping, multi-format writing, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create marketing content using Claude and OpenAI
  - automate content creation from research to video
  - build an AI content automation system
  - use the marketing pipeline to generate posts
  - scrape news and generate content automatically
  - create videos from written content with Remotion
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a comprehensive AI-powered content automation system that handles the entire content creation workflow: from researching trending topics across platforms like TechCrunch, Twitter, and LinkedIn, to generating multi-format articles in multiple languages, to rendering videos and graphics using Remotion. Built with Next.js and TypeScript, it integrates Claude 3, OpenAI, and RapidAPI for a complete content production pipeline.

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
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion Config (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Content scraping utilities
│   │   ├── renderer/    # Remotion video rendering
│   │   └── utils/       # Helpers
│   └── remotion/        # Remotion compositions
├── public/              # Static assets
└── package.json
```

## Core Features & Usage

### 1. Auto-Research & Content Scraping

The pipeline automatically scrapes trending content from multiple sources:

```typescript
// lib/scraper/research.ts
import axios from 'axios';

interface ResearchSource {
  platform: 'techcrunch' | 'twitter' | 'linkedin' | 'a16z';
  timeframe: '24h' | '7d' | '30d';
}

export async function scrapeResearch(
  keyword: string,
  sources: ResearchSource[]
): Promise<ResearchData[]> {
  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await axios.get(
        `https://api.rapidapi.com/${source.platform}/search`,
        {
          params: { q: keyword, timeframe: source.timeframe },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
            'X-RapidAPI-Host': `${source.platform}.api.rapidapi.com`
          }
        }
      );
      
      return parseSourceData(response.data, source.platform);
    })
  );
  
  return results.flat();
}

function parseSourceData(data: any, platform: string): ResearchData[] {
  // Extract relevant insights, trends, and data points
  return data.articles.map((article: any) => ({
    title: article.title,
    summary: article.summary,
    url: article.url,
    publishedAt: article.publishedAt,
    platform,
    metrics: {
      engagement: article.engagement,
      shares: article.shares
    }
  }));
}
```

### 2. AI-Powered Content Generation

Generate multi-format content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchData[];
}

export async function generateContent(
  topic: string,
  config: ContentConfig
): Promise<GeneratedContent> {
  const researchContext = config.research
    .map(r => `- ${r.title}: ${r.summary} (${r.url})`)
    .join('\n');

  const prompt = buildPrompt(topic, config, researchContext);

  // Use Claude for long-form content
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';

  return {
    content,
    metadata: {
      wordCount: content.split(' ').length,
      format: config.format,
      language: config.language
    }
  };
}

function buildPrompt(
  topic: string,
  config: ContentConfig,
  research: string
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with detailed explanations',
    'pov': 'Write a thought-leadership piece with a unique perspective',
    'case-study': 'Analyze a specific example with data and outcomes',
    'how-to': 'Provide step-by-step instructions with practical examples'
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terms',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and engaging storytelling'
  };

  return `Write a ${config.format} article about "${topic}" in ${config.language}.

Tone: ${toneInstructions[config.tone]}
Format: ${formatInstructions[config.format]}

Use this recent research as context:
${research}

Requirements:
- Include data-backed insights
- Reference recent trends (within 24-48h)
- ${config.language === 'vi' ? 'Write in Vietnamese' : 'Write in English'}
- Make it engaging and actionable
- Include relevant statistics and examples`;
}
```

### 3. Bilingual Content Generation

Generate content simultaneously in English and Vietnamese:

```typescript
// lib/ai/bilingual-generator.ts
export async function generateBilingualContent(
  topic: string,
  format: ContentConfig['format'],
  research: ResearchData[]
): Promise<{ en: GeneratedContent; vi: GeneratedContent }> {
  const [enContent, viContent] = await Promise.all([
    generateContent(topic, {
      format,
      language: 'en',
      tone: 'expert',
      research
    }),
    generateContent(topic, {
      format,
      language: 'vi',
      tone: 'friendly',
      research
    })
  ]);

  return { en: enContent, vi: viContent };
}
```

### 4. Video Generation with Remotion

Transform written content into videos:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  duration
}) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity
          }}
        >
          <h1 style={{ color: 'white', fontSize: 60 }}>{title}</h1>
        </AbsoluteFill>
      </Sequence>

      {points.map((point, index) => (
        <Sequence
          key={index}
          from={90 + index * 120}
          durationInFrames={120}
        >
          <PointSlide point={point} index={index} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ point: string; index: number }> = ({
  point,
  index
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15], [0, 1]);
  
  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        opacity
      }}
    >
      <div style={{ color: 'white', fontSize: 40, maxWidth: '80%' }}>
        <span style={{ color: '#00ff00' }}>{index + 1}. </span>
        {point}
      </div>
    </AbsoluteFill>
  );
};
```

Render the video:

```typescript
// lib/renderer/video-render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: GeneratedContent,
  outputPath: string
): Promise<string> {
  const bundled = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      duration: 30 // seconds
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps
  });

  return outputPath;
}
```

### 5. Complete Pipeline Execution

Orchestrate the entire workflow:

```typescript
// lib/pipeline/executor.ts
export async function executeContentPipeline(
  keyword: string,
  options: PipelineOptions
): Promise<PipelineResult> {
  // Step 1: Research
  console.log('🔍 Researching trending content...');
  const research = await scrapeResearch(keyword, [
    { platform: 'techcrunch', timeframe: '24h' },
    { platform: 'twitter', timeframe: '24h' },
    { platform: 'linkedin', timeframe: '7d' }
  ]);

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await generateBilingualContent(
    keyword,
    options.format || 'toplist',
    research
  );

  // Step 3: Render video (optional)
  let videoPath: string | null = null;
  if (options.generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo(
      content.en,
      path.join(process.cwd(), 'output', `${keyword}-video.mp4`)
    );
  }

  // Step 4: Save results
  const result = {
    keyword,
    research,
    content,
    videoPath,
    createdAt: new Date()
  };

  await saveToDatabase(result);

  return result;
}
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { executeContentPipeline } from '@/lib/pipeline/executor';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, generateVideo } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await executeContentPipeline(keyword, {
      format: format || 'toplist',
      generateVideo: generateVideo || false
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Pattern 1: Custom Content Format

```typescript
// Add a new format to the system
const customFormats = {
  'interview': {
    prompt: 'Create an interview-style Q&A format',
    sections: ['intro', 'questions', 'conclusion']
  },
  'comparison': {
    prompt: 'Compare and contrast different approaches',
    sections: ['overview', 'comparison-table', 'recommendation']
  }
};

function buildCustomPrompt(format: string, topic: string): string {
  const formatConfig = customFormats[format];
  return `${formatConfig.prompt} about ${topic}. 
  Include these sections: ${formatConfig.sections.join(', ')}`;
}
```

### Pattern 2: Content Scheduling

```typescript
// Schedule content generation
import { CronJob } from 'cron';

const dailyContentJob = new CronJob('0 9 * * *', async () => {
  const trendingTopics = await getTrendingTopics();
  
  for (const topic of trendingTopics) {
    await executeContentPipeline(topic, {
      format: 'pov',
      generateVideo: true
    });
  }
});

dailyContentJob.start();
```

### Pattern 3: Multi-Platform Export

```typescript
// Export content for different platforms
export async function exportToPlatforms(
  content: GeneratedContent,
  platforms: Platform[]
): Promise<void> {
  const exports = platforms.map(async (platform) => {
    switch (platform) {
      case 'facebook':
        return formatForFacebook(content);
      case 'linkedin':
        return formatForLinkedIn(content);
      case 'twitter':
        return formatForTwitterThread(content);
      default:
        return content.content;
    }
  });

  await Promise.all(exports);
}
```

## CLI Usage

If the project includes CLI commands:

```bash
# Generate content from command line
npm run generate -- --keyword "AI trends 2024" --format toplist --video

# Research only
npm run research -- --keyword "machine learning" --sources techcrunch,twitter

# Render video from existing content
npm run render -- --input content.json --output video.mp4
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting and retry logic
import pRetry from 'p-retry';

async function callAPIWithRetry<T>(
  apiCall: () => Promise<T>,
  retries = 3
): Promise<T> {
  return pRetry(apiCall, {
    retries,
    onFailedAttempt: (error) => {
      console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
    },
    minTimeout: 1000,
    maxTimeout: 5000
  });
}
```

### Issue: Video Rendering Memory Issues

```typescript
// Configure Remotion for lower memory usage
export const videoConfig = {
  codec: 'h264',
  concurrency: 1, // Reduce concurrent rendering
  imageFormat: 'jpeg',
  scale: 0.5, // Lower resolution for testing
  quality: 80
};
```

### Issue: Research Data Quality

```typescript
// Filter and validate research data
function validateResearch(data: ResearchData[]): ResearchData[] {
  return data.filter(item => {
    return (
      item.title &&
      item.summary &&
      item.summary.length > 50 && // Minimum content length
      new Date(item.publishedAt) > new Date(Date.now() - 48 * 60 * 60 * 1000) // Max 48h old
    );
  });
}
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use streaming responses** for real-time content generation feedback
3. **Implement content versioning** to track iterations
4. **Add quality checks** before publishing content
5. **Monitor API costs** with usage tracking middleware
6. **Test video renders locally** before production deployment
