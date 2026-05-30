---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion rendering
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline with research and video generation
  - create automated marketing content with Claude and OpenAI
  - generate videos from text content automatically
  - build a content automation system with AI
  - auto-research and create content with AI agents
  - use remotion to render marketing videos
  - scrape news and generate content automatically
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete AI-powered content automation pipeline that handles research (crawling news sources), content generation (using Claude/OpenAI), and video rendering (via Remotion). It transforms keywords into multi-format content with automatic translation and video generation.

## What It Does

The Ultimate AI Content Pipeline automates:

1. **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for recent data (last 24h)
2. **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates English and Vietnamese versions simultaneously
4. **Video Rendering**: Auto-generates infographics and short videos using Remotion
5. **Platform Optimization**: Outputs video in formats optimized for Reels, TikTok, Shorts

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
# AI Provider APIs
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research/Crawling API
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database and Storage
DATABASE_URL=your_database_url_here
STORAGE_URL=your_storage_url_here

# Remotion Configuration
REMOTION_RENDER_CONCURRENCY=4
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping and research
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Helper functions
│   └── remotion/        # Remotion video compositions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Research & Data Collection

```typescript
import { researchTopic } from '@/lib/research/crawler';

async function gatherResearch(keyword: string) {
  const researchData = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });

  return {
    articles: researchData.articles,
    insights: researchData.insights,
    statistics: researchData.statistics
  };
}

// Usage
const data = await gatherResearch('AI marketing automation');
console.log(`Found ${data.articles.length} articles`);
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
  researchData: any
) {
  const prompt = `
Based on this research data: ${JSON.stringify(researchData)}

Create a ${format} article about "${topic}" with:
- Engaging headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure

Format: ${format}
Language: English
Tone: Professional yet approachable
`;

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

// Usage
const content = await generateContent(
  'AI in Marketing 2026',
  'toplist',
  researchData
);
```

### 3. Multi-language Generation

```typescript
async function generateBilingual(
  topic: string,
  format: string,
  researchData: any
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(topic, format, researchData),
    generateContentVietnamese(topic, format, researchData)
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}

async function generateContentVietnamese(
  topic: string,
  format: string,
  researchData: any
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Viết bài ${format} về "${topic}" bằng Tiếng Việt dựa trên dữ liệu: ${JSON.stringify(researchData)}`
      }
    ]
  });

  return message.content[0].text;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './src/remotion/webpack-override';

async function generateVideo(content: {
  title: string;
  points: string[];
  statistics: any[];
}) {
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'MarketingInfographic',
    inputProps: {
      title: content.title,
      points: content.points,
      statistics: content.statistics,
      theme: 'modern'
    },
  });

  // Render video
  const outputLocation = `./public/videos/${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    concurrency: Number(process.env.REMOTION_RENDER_CONCURRENCY) || 4,
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  title: 'Top 5 AI Marketing Trends',
  points: [
    'AI-powered personalization',
    'Automated content creation',
    'Predictive analytics'
  ],
  statistics: [
    { label: 'ROI Increase', value: '90%' },
    { label: 'Time Saved', value: '15 hours/week' }
  ]
});
```

### 5. Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateBilingual } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/renderer';

export async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h'
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateBilingual(
      keyword,
      'toplist',
      research
    );

    // Step 3: Extract key points for video
    const videoData = {
      title: content.en.split('\n')[0],
      points: extractKeyPoints(content.en),
      statistics: research.statistics
    };

    // Step 4: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo(videoData);

    return {
      content,
      videoPath,
      research: {
        sources: research.articles.length,
        insights: research.insights.length
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - customize based on your format
  const lines = content.split('\n');
  return lines
    .filter(line => line.match(/^\d+\./))
    .slice(0, 5)
    .map(line => line.replace(/^\d+\.\s*/, ''));
}

// Run the pipeline
const result = await runContentPipeline('AI Marketing Automation 2026');
console.log('✅ Complete!', result);
```

## Remotion Video Composition Example

Create a composition at `src/remotion/compositions/MarketingInfographic.tsx`:

```typescript
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';

export const MarketingInfographic: React.FC<{
  title: string;
  points: string[];
  statistics: Array<{ label: string; value: string }>;
}> = ({ title, points, statistics }) => {
  const frame = useCurrentFrame();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity,
          }}
        >
          <h1 style={{ color: 'white', fontSize: 60, textAlign: 'center' }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Points Sequence */}
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={90 + index * 60}
          durationInFrames={60}
        >
          <PointSlide point={point} index={index + 1} />
        </Sequence>
      ))}

      {/* Statistics Sequence */}
      <Sequence from={90 + points.length * 60} durationInFrames={90}>
        <StatisticsSlide statistics={statistics} />
      </Sequence>
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ point: string; index: number }> = ({
  point,
  index,
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15], [0, 1]);

  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
      }}
    >
      <div style={{ color: 'white', fontSize: 40, maxWidth: '80%' }}>
        <span style={{ color: '#00d4ff', fontWeight: 'bold' }}>
          {index}.
        </span>{' '}
        {point}
      </div>
    </AbsoluteFill>
  );
};

const StatisticsSlide: React.FC<{
  statistics: Array<{ label: string; value: string }>;
}> = ({ statistics }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        flexDirection: 'column',
        gap: 40,
      }}
    >
      {statistics.map((stat, index) => {
        const opacity = interpolate(
          frame,
          [index * 10, index * 10 + 15],
          [0, 1]
        );

        return (
          <div
            key={index}
            style={{
              opacity,
              color: 'white',
              textAlign: 'center',
            }}
          >
            <div style={{ fontSize: 60, color: '#00d4ff' }}>
              {stat.value}
            </div>
            <div style={{ fontSize: 30 }}>{stat.label}</div>
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Preview Remotion compositions
npm run remotion

# Render a specific video
npm run render -- --composition=MarketingInfographic --props='{"title":"Test"}'
```

## Common Patterns

### Scheduled Content Generation

```typescript
import cron from 'node-cron';
import { runContentPipeline } from '@/lib/pipeline';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI Marketing', 'Social Media Trends', 'SEO Updates'];
  
  for (const topic of topics) {
    try {
      await runContentPipeline(topic);
      console.log(`✅ Generated content for: ${topic}`);
    } catch (error) {
      console.error(`❌ Failed for ${topic}:`, error);
    }
  }
});
```

### Custom Research Sources

```typescript
import axios from 'axios';

async function crawlCustomSource(url: string) {
  const response = await axios.get(url, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'your-api-host.rapidapi.com'
    }
  });

  return response.data.articles.map((article: any) => ({
    title: article.title,
    content: article.content,
    url: article.url,
    publishedAt: article.publishedAt
  }));
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }]
      });
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Remotion Rendering Memory Issues

```bash
# Increase Node memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run render

# Or reduce concurrency in .env.local
REMOTION_RENDER_CONCURRENCY=2
```

### Missing Research Data

```typescript
// Add fallback data sources
async function researchWithFallback(keyword: string) {
  try {
    return await researchTopic({ keyword, sources: ['techcrunch', 'a16z'] });
  } catch (error) {
    console.warn('Primary sources failed, using fallback...');
    return {
      articles: [],
      insights: [`General insight about ${keyword}`],
      statistics: []
    };
  }
}
```

### Video Rendering Failures

```typescript
// Add validation before rendering
function validateVideoData(data: any) {
  if (!data.title || data.points.length === 0) {
    throw new Error('Invalid video data: missing title or points');
  }
  
  if (data.title.length > 100) {
    data.title = data.title.substring(0, 97) + '...';
  }
  
  return data;
}

const videoData = validateVideoData({
  title: content.en.split('\n')[0],
  points: extractKeyPoints(content.en),
  statistics: research.statistics
});
```

## Best Practices

1. **Cache Research Data**: Store research results to avoid redundant API calls
2. **Queue Video Renders**: Use a job queue (Bull, BullMQ) for video rendering to avoid memory issues
3. **Monitor API Usage**: Track API calls to stay within rate limits
4. **Version Control Prompts**: Store AI prompts in separate files for easy iteration
5. **Test Compositions**: Preview Remotion videos before full render

This skill enables AI agents to build and customize complete content automation pipelines with research, AI generation, and video rendering capabilities.
