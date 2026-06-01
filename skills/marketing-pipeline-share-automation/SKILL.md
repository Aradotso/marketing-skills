---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video using AI (Claude, OpenAI) and Remotion for Vietnamese and English marketing content
triggers:
  - how do I automate content research and video generation
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content from keyword to video
  - generate multilingual blog posts with auto research
  - use Remotion to render videos from AI content
  - build end-to-end content automation system
  - crawl news and generate content automatically
  - create TikTok and Reels videos from text automatically
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

Marketing Pipeline Share is an end-to-end content automation system that transforms a single keyword into complete marketing assets including:

- **Auto-research**: Crawls recent news from TechCrunch, a16z, Twitter, LinkedIn
- **AI content generation**: Creates blog posts in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual support**: Generates both English and Vietnamese content simultaneously
- **Video rendering**: Automatically creates short-form videos, infographics, and Reels using Remotion
- **Multiple content formats**: Adapts tone and style for different audiences

This is built with Next.js (TypeScript), integrates with Anthropic Claude, OpenAI, RapidAPI for data sources, and Remotion for video rendering.

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Providers
ANTHROPIC_API_KEY=your_anthropic_key
OPENAI_API_KEY=your_openai_key

# Data Sources (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key
RAPIDAPI_HOST=your_api_host

# Content Scraping
TECHCRUNCH_RSS_URL=https://techcrunch.com/feed/
A16Z_RSS_URL=https://a16z.com/feed/

# Remotion Video Settings
REMOTION_OUTPUT_DIR=./public/videos
VIDEO_WIDTH=1080
VIDEO_HEIGHT=1920

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Visit `http://localhost:3000` to access the application.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── scrapers/    # Content crawlers
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core API and Usage Patterns

### 1. Content Research Module

The research module automatically crawls and analyzes recent news:

```typescript
import { researchContent } from '@/lib/scrapers/research';

// Automatic content research
async function performResearch(keyword: string) {
  const researchData = await researchContent({
    keyword: keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });

  return {
    insights: researchData.insights,
    trends: researchData.trends,
    dataPoints: researchData.statistics,
    sources: researchData.sourceUrls
  };
}

// Usage
const data = await performResearch('AI automation');
console.log(data.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  researchData: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const systemPrompt = `You are an expert content creator. Generate ${format} content in ${language} based on the research data provided.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Research data: ${JSON.stringify(researchData)}\n\nCreate engaging ${format} content that incorporates these insights.`
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Generate bilingual content
const englishContent = await generateContentWithClaude(
  researchData, 
  'toplist', 
  'en'
);
const vietnameseContent = await generateContentWithClaude(
  researchData, 
  'toplist', 
  'vi'
);
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(
  researchData: any,
  tone: 'professional' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer. Create engaging marketing content based on research data.`
      },
      {
        role: 'user',
        content: `Research: ${JSON.stringify(researchData)}\n\nWrite a compelling article.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

Create a Remotion composition for short-form videos:

```typescript
// remotion/compositions/ShortVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ShortVideoProps {
  title: string;
  points: string[];
  bgColor: string;
}

export const ShortVideo: React.FC<ShortVideoProps> = ({ 
  title, 
  points, 
  bgColor 
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        opacity, 
        padding: 60,
        color: 'white',
        fontSize: 48,
        fontWeight: 'bold'
      }}>
        <h1>{title}</h1>
        <ul>
          {points.map((point, idx) => (
            <li key={idx} style={{ marginTop: 20 }}>{point}</li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

Render the video programmatically:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  content: { title: string; points: string[] }
) {
  const compositionId = 'ShortVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      points: content.points,
      bgColor: '#1a1a2e'
    },
  });

  const outputLocation = path.join(
    process.env.REMOTION_OUTPUT_DIR || './public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps,
  });

  return outputLocation;
}

// Usage
const videoPath = await renderContentVideo({
  title: 'Top 5 AI Trends 2024',
  points: [
    'AI Automation',
    'LLM Integration',
    'Video Generation',
    'Content Personalization',
    'Multi-language Support'
  ]
});
```

### 5. Complete Pipeline Integration

End-to-end content creation workflow:

```typescript
import { researchContent } from '@/lib/scrapers/research';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/remotion';

interface ContentPipelineOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
}

async function runContentPipeline(options: ContentPipelineOptions) {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const researchData = await researchContent({
    keyword: options.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  });

  // Step 2: Generate content for each language
  console.log('✍️ Generating content...');
  const contentResults = await Promise.all(
    options.languages.map(async (lang) => {
      const content = await generateContentWithClaude(
        researchData,
        options.format,
        lang
      );
      return { language: lang, content };
    })
  );

  // Step 3: Extract key points for video
  const keyPoints = researchData.insights.slice(0, 5);

  // Step 4: Generate video if requested
  let videoPath = null;
  if (options.generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo({
      title: options.keyword,
      points: keyPoints
    });
  }

  return {
    research: researchData,
    content: contentResults,
    video: videoPath
  };
}

// Example usage
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true
});

console.log('✅ Pipeline complete!');
console.log('Articles:', result.content.length);
console.log('Video:', result.video);
```

## Configuration Patterns

### Custom Content Format Templates

```typescript
// src/lib/ai/templates.ts
export const contentTemplates = {
  toplist: {
    systemPrompt: 'Create a numbered list article with clear structure',
    structure: ['intro', 'points', 'conclusion']
  },
  pov: {
    systemPrompt: 'Write from a strong personal perspective with opinion',
    structure: ['hook', 'opinion', 'arguments', 'conclusion']
  },
  'case-study': {
    systemPrompt: 'Analyze a real-world example with data',
    structure: ['context', 'challenge', 'solution', 'results']
  },
  'how-to': {
    systemPrompt: 'Create step-by-step instructional content',
    structure: ['overview', 'steps', 'tips', 'conclusion']
  }
};

export function getTemplate(format: keyof typeof contentTemplates) {
  return contentTemplates[format];
}
```

### Video Style Presets

```typescript
// src/lib/video/presets.ts
export const videoPresets = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 450, // 15 seconds
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900, // 30 seconds
  },
  youtube_shorts: {
    width: 1080,
    height: 1920,
    fps: 60,
    durationInFrames: 1800, // 30 seconds
  }
};
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// Add retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
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
  generateContentWithClaude(data, 'toplist', 'en')
);
```

### Video Rendering Memory Issues

For large video renders:

```typescript
// Use chunked rendering for long videos
import { renderFrames } from '@remotion/renderer';

async function renderInChunks(composition: any, outputPath: string) {
  const chunkSize = 300; // frames per chunk
  const totalFrames = composition.durationInFrames;
  
  for (let i = 0; i < totalFrames; i += chunkSize) {
    await renderFrames({
      composition,
      serveUrl: composition.serveUrl,
      onFrameUpdate: (frame) => {
        console.log(`Rendering frame ${frame}/${totalFrames}`);
      },
      frameRange: [i, Math.min(i + chunkSize, totalFrames)],
      outputDir: outputPath,
    });
  }
}
```

### Content Quality Issues

Improve AI output with better prompts:

```typescript
function buildDetailedPrompt(
  researchData: any,
  format: string,
  language: string
) {
  return `
You are an expert marketing content writer.

CONTEXT:
- Format: ${format}
- Language: ${language}
- Target audience: Marketing professionals and business owners
- Tone: Professional yet engaging

RESEARCH DATA:
${JSON.stringify(researchData, null, 2)}

REQUIREMENTS:
1. Use specific data points and statistics from the research
2. Include actionable insights
3. Structure with clear headings and bullet points
4. Add relevant examples
5. Keep paragraphs short (2-3 sentences max)
6. End with a strong call-to-action

Create the content now:
  `.trim();
}
```

### Debugging Scraper Issues

```typescript
// Add verbose logging for scraper debugging
async function debugResearch(keyword: string) {
  console.log('Starting research for:', keyword);
  
  try {
    const results = await researchContent({
      keyword,
      sources: ['techcrunch'],
      timeRange: '24h',
      debug: true // Enable debug mode
    });
    
    console.log('Sources found:', results.sources.length);
    console.log('Insights extracted:', results.insights.length);
    
    return results;
  } catch (error) {
    console.error('Research failed:', error);
    throw error;
  }
}
```

## Common Use Cases

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run content pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI automation', 'Marketing trends', 'Content strategy'];
  
  for (const keyword of keywords) {
    await runContentPipeline({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true
    });
  }
});
```

### Batch Processing Multiple Keywords

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['en'],
        generateVideo: false
      })
    )
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  console.log(`✅ Success: ${successful.length}`);
  console.log(`❌ Failed: ${failed.length}`);

  return { successful, failed };
}
```
