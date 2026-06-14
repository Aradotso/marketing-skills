---
name: marketing-pipeline-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for multi-platform marketing content
triggers:
  - how do I automate content creation with AI
  - generate marketing content from research to video
  - set up AI content pipeline with Claude and OpenAI
  - create automated video content from articles
  - build content automation with Remotion
  - configure multi-language content generation
  - automate social media content research and creation
  - set up AI-powered marketing automation pipeline
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - a complete automation system that handles content research, scriptwriting, multi-language generation, and video rendering for marketing at scale.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Uses Claude 3 and OpenAI to create content in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically converts written content into videos and infographics using Remotion
5. **Multi-platform Export**: Optimized for Reels, TikTok, Shorts, and other social platforms

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment variables
cp .env.example .env
```

## Configuration

Set up your environment variables in `.env`:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for research/crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Content settings
DEFAULT_LANGUAGE=en
SECONDARY_LANGUAGE=vi
CONTENT_TONE=professional # or friendly, humorous, expert

# Remotion video settings
REMOTION_OUTPUT_DIR=./output/videos
VIDEO_FORMAT=mp4
VIDEO_RESOLUTION=1080x1920 # for vertical social media
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── research/         # Auto-crawling and research modules
│   ├── content/          # AI content generation
│   ├── video/            # Remotion video rendering
│   ├── api/              # API integrations
│   └── utils/            # Helper functions
├── remotion/             # Remotion video templates
├── public/               # Static assets
└── pages/                # Next.js pages
```

## Core API Usage

### Research Module

```typescript
import { autoResearch } from '@/src/research/crawler';

// Automatically research a topic
const researchData = await autoResearch({
  keyword: 'AI marketing automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h', // Last 24 hours
  maxResults: 50
});

console.log(researchData);
// {
//   insights: [...],
//   articles: [...],
//   trends: [...],
//   dataPoints: [...]
// }
```

### Content Generation

```typescript
import { generateContent } from '@/src/content/generator';

// Generate multi-format content
const content = await generateContent({
  topic: 'AI in Marketing 2026',
  format: 'toplist', // or 'pov', 'case-study', 'how-to'
  tone: 'professional',
  languages: ['en', 'vi'],
  aiProvider: 'claude', // or 'openai'
  researchData: researchData, // From previous step
  targetPlatform: 'linkedin'
});

console.log(content);
// {
//   en: { title: '...', body: '...', metadata: {...} },
//   vi: { title: '...', body: '...', metadata: {...} }
// }
```

### Video Rendering

```typescript
import { renderVideo } from '@/src/video/renderer';
import { bundle } from '@remotion/bundler';

// Render video from content
const videoPath = await renderVideo({
  content: content.en,
  template: 'infographic', // or 'talking-head', 'slideshow'
  aspectRatio: '9:16', // Vertical for social
  duration: 60, // seconds
  outputPath: './output/videos'
});

console.log(`Video rendered: ${videoPath}`);
```

## Complete Workflow Example

```typescript
import { autoResearch } from '@/src/research/crawler';
import { generateContent } from '@/src/content/generator';
import { renderVideo } from '@/src/video/renderer';
import { publishToSocial } from '@/src/api/social';

async function createContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('Starting research...');
    const research = await autoResearch({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
      maxResults: 30
    });

    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContent({
      topic: keyword,
      format: 'toplist',
      tone: 'expert',
      languages: ['en', 'vi'],
      aiProvider: 'claude',
      researchData: research,
      targetPlatform: 'linkedin'
    });

    // Step 3: Render Video
    console.log('Rendering video...');
    const video = await renderVideo({
      content: content.en,
      template: 'infographic',
      aspectRatio: '9:16',
      duration: 60
    });

    // Step 4: Optional - Auto publish
    console.log('Publishing...');
    const published = await publishToSocial({
      content: content.en,
      video: video,
      platforms: ['linkedin', 'twitter']
    });

    return {
      research,
      content,
      video,
      published
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
createContentPipeline('AI marketing trends 2026')
  .then(result => console.log('Success:', result))
  .catch(err => console.error('Failed:', err));
```

## CLI Commands

```bash
# Run development server
npm run dev

# Generate content from keyword
npm run generate -- --keyword "your topic" --format toplist

# Render video from existing content
npm run render -- --content ./content/article.json --template infographic

# Run full pipeline
npm run pipeline -- --keyword "AI trends" --publish

# Build for production
npm run build

# Start production server
npm start
```

## API Integrations

### Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [{ role: 'user', content: prompt }],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

## Common Patterns

### Custom Content Template

```typescript
import { ContentTemplate } from '@/src/types';

const customTemplate: ContentTemplate = {
  name: 'product-launch',
  structure: {
    hook: 'attention-grabbing-opening',
    problem: 'pain-point-description',
    solution: 'product-introduction',
    benefits: 'key-features-list',
    cta: 'call-to-action'
  },
  tone: 'excited',
  length: 'medium' // short, medium, long
};

const content = await generateContent({
  topic: 'New AI Tool Launch',
  customTemplate,
  languages: ['en'],
  aiProvider: 'openai'
});
```

### Batch Processing

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      createContentPipeline(keyword)
        .catch(err => ({ error: err.message, keyword }))
    )
  );

  const successful = results.filter(r => !r.error);
  const failed = results.filter(r => r.error);

  console.log(`Success: ${successful.length}, Failed: ${failed.length}`);
  return { successful, failed };
}

// Generate content for multiple topics
batchGenerate([
  'AI in healthcare',
  'Marketing automation 2026',
  'Social media trends'
]);
```

### Custom Video Composition (Remotion)

```typescript
import { Composition } from 'remotion';
import { InfographicTemplate } from '@/remotion/templates/Infographic';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="Infographic"
        component={InfographicTemplate}
        durationInFrames={1800} // 60 seconds at 30fps
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Title',
          data: [],
          theme: 'modern'
        }}
      />
    </>
  );
};
```

## Troubleshooting

### Rate Limiting Issues

```typescript
import pRetry from 'p-retry';

async function generateWithRetry(prompt: string) {
  return pRetry(
    async () => {
      return await generateWithClaude(prompt);
    },
    {
      retries: 3,
      minTimeout: 1000,
      maxTimeout: 5000,
      onFailedAttempt: error => {
        console.log(
          `Attempt ${error.attemptNumber} failed. Retries left: ${error.retriesLeft}`
        );
      }
    }
  );
}
```

### Memory Issues with Large Research

```typescript
import { chunk } from 'lodash';

async function processLargeResearch(keywords: string[]) {
  // Process in chunks to avoid memory overflow
  const batches = chunk(keywords, 10);
  const results = [];

  for (const batch of batches) {
    const batchResults = await Promise.all(
      batch.map(k => autoResearch({ keyword: k, maxResults: 20 }))
    );
    results.push(...batchResults);
    
    // Clear memory between batches
    if (global.gc) global.gc();
  }

  return results;
}
```

### Video Rendering Timeout

```typescript
import { renderMedia } from '@remotion/renderer';

async function renderWithTimeout(composition: string, timeout = 300000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    const output = await renderMedia({
      composition,
      serveUrl: 'http://localhost:3000',
      codec: 'h264',
      outputLocation: './output/video.mp4',
      signal: controller.signal
    });
    clearTimeout(timeoutId);
    return output;
  } catch (error) {
    if (error.name === 'AbortError') {
      throw new Error('Video rendering timeout exceeded');
    }
    throw error;
  }
}
```

### Debug Mode

```typescript
// Enable detailed logging
process.env.DEBUG = 'marketing-pipeline:*';

import debug from 'debug';
const log = debug('marketing-pipeline:main');

log('Starting content generation...');
log('Research data:', researchData);
log('Generated content length:', content.length);
```

## Best Practices

1. **API Key Management**: Always use environment variables, never hardcode
2. **Rate Limiting**: Implement retry logic for API calls
3. **Error Handling**: Wrap pipeline steps in try-catch blocks
4. **Caching**: Cache research results to avoid redundant API calls
5. **Batch Processing**: Process multiple items in controlled batches
6. **Memory Management**: Clear large objects after processing
7. **Logging**: Use structured logging for debugging and monitoring
