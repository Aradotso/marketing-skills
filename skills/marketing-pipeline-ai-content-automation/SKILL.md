---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up AI content pipeline for marketing
  - generate videos from text using Remotion
  - crawl news sources for content research
  - create multilingual content with Claude API
  - automate social media video generation
  - build content automation system
  - use marketing pipeline for AI content
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates content creation from research and scriptwriting to automatic video generation and social media posting.

## What It Does

The marketing pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter/X, LinkedIn) for trending topics
2. **AI Content Generation**: Uses Claude 3 and OpenAI to generate articles in multiple formats (toplist, POV, case study, how-to)
3. **Multilingual Support**: Automatically creates content in both English and Vietnamese
4. **Video Generation**: Uses Remotion to render infographics and short videos optimized for Reels, TikTok, and YouTube Shorts
5. **Auto-Publishing**: Posts content directly to social media platforms

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm >= 9.0.0
```

### Setup Steps

1. Clone the repository:

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
```

2. Install dependencies:

```bash
npm install
# or
yarn install
# or
pnpm install
```

3. Configure environment variables:

```bash
cp .env.example .env
```

4. Add required API keys to `.env`:

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Remotion (Video Generation)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Social Media APIs
FACEBOOK_PAGE_ACCESS_TOKEN=your_facebook_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

5. Run development server:

```bash
npm run dev
```

The application will be available at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── video/       # Remotion video generation
│   │   └── publisher/   # Social media posting
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Key APIs and Usage

### 1. Content Research Module

```typescript
import { crawlNewsSource } from '@/lib/crawler';

// Crawl TechCrunch for AI-related articles
const articles = await crawlNewsSource({
  source: 'techcrunch',
  keyword: 'artificial intelligence',
  timeframe: '24h',
  limit: 10
});

console.log(articles);
// Returns: Array of { title, url, content, publishedAt, author }
```

### 2. AI Content Generation with Claude

```typescript
import { generateContent } from '@/lib/ai/claude';

const content = await generateContent({
  topic: 'AI in Marketing',
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  language: 'en', // 'en' | 'vi'
  tone: 'professional', // 'professional' | 'friendly' | 'humorous'
  researchData: articles, // From crawler
  apiKey: process.env.ANTHROPIC_API_KEY
});

console.log(content);
// Returns: { title, body, summary, keywords, metadata }
```

### 3. OpenAI Alternative

```typescript
import { generateWithOpenAI } from '@/lib/ai/openai';

const content = await generateWithOpenAI({
  prompt: 'Write a professional article about AI trends',
  model: 'gpt-4-turbo-preview',
  apiKey: process.env.OPENAI_API_KEY,
  maxTokens: 2000
});
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

// Prepare video composition
const bundled = await bundle({
  entryPoint: './remotion/index.ts',
  webpackOverride: (config) => config
});

// Render video from content
const video = await renderVideo({
  compositionId: 'ContentVideo',
  inputProps: {
    title: content.title,
    points: content.body.split('\n').slice(0, 5),
    duration: 30, // seconds
    platform: 'instagram-reels' // 'instagram-reels' | 'tiktok' | 'youtube-shorts'
  },
  outputPath: './output/video.mp4',
  serveUrl: bundled
});

console.log(`Video rendered: ${video.outputPath}`);
```

### 5. Auto-Publishing

```typescript
import { publishToSocial } from '@/lib/publisher';

const result = await publishToSocial({
  platforms: ['facebook', 'linkedin'],
  content: {
    text: content.summary,
    media: {
      type: 'video',
      path: video.outputPath
    },
    hashtags: content.keywords
  },
  tokens: {
    facebook: process.env.FACEBOOK_PAGE_ACCESS_TOKEN,
    linkedin: process.env.LINKEDIN_ACCESS_TOKEN
  }
});

console.log(result);
// Returns: { facebook: { postId, url }, linkedin: { postId, url } }
```

## Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Initialize pipeline
const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY
});

// Run complete automation
async function runContentAutomation() {
  try {
    // Step 1: Research
    const research = await pipeline.research({
      keyword: 'AI marketing automation',
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h'
    });

    console.log(`Found ${research.length} articles`);

    // Step 2: Generate content in multiple languages
    const contentEN = await pipeline.generateContent({
      research,
      format: 'toplist',
      language: 'en',
      tone: 'professional'
    });

    const contentVI = await pipeline.generateContent({
      research,
      format: 'toplist',
      language: 'vi',
      tone: 'professional'
    });

    // Step 3: Generate videos
    const videoReels = await pipeline.generateVideo({
      content: contentEN,
      platform: 'instagram-reels',
      duration: 30
    });

    const videoTikTok = await pipeline.generateVideo({
      content: contentEN,
      platform: 'tiktok',
      duration: 60
    });

    // Step 4: Publish
    const published = await pipeline.publish({
      content: contentEN,
      video: videoReels,
      platforms: ['facebook', 'instagram', 'linkedin'],
      schedule: new Date(Date.now() + 3600000) // 1 hour later
    });

    return {
      research: research.length,
      content: { en: contentEN, vi: contentVI },
      videos: { reels: videoReels, tiktok: videoTikTok },
      published
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute
runContentAutomation()
  .then(result => console.log('Pipeline completed:', result))
  .catch(error => console.error('Pipeline failed:', error));
```

## Configuration

### Content Formats

```typescript
type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';

const formatConfigs = {
  toplist: {
    structure: 'numbered list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    structure: 'opinion piece',
    perspective: 'first-person',
    argumentStyle: 'persuasive'
  },
  'case-study': {
    structure: 'problem-solution-results',
    includeMetrics: true,
    includeQuotes: true
  },
  'how-to': {
    structure: 'step-by-step',
    includeVisuals: true,
    difficultyLevel: 'beginner' | 'intermediate' | 'advanced'
  }
};
```

### Video Templates

```typescript
// remotion/compositions.ts
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';
import { InfographicVideo } from './InfographicVideo';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920} // 9:16 for Reels/TikTok
      />
      <Composition
        id="InfographicVideo"
        component={InfographicVideo}
        durationInFrames={1800} // 60 seconds
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
};
```

### Crawler Configuration

```typescript
// Configure news sources
const crawlerConfig = {
  sources: {
    techcrunch: {
      baseUrl: 'https://techcrunch.com',
      apiEndpoint: '/wp-json/wp/v2/posts',
      rateLimit: 100, // requests per hour
      enabled: true
    },
    twitter: {
      apiVersion: '2',
      searchEndpoint: 'https://api.twitter.com/2/tweets/search/recent',
      maxResults: 100,
      enabled: true
    },
    linkedin: {
      apiVersion: 'v2',
      enabled: true
    }
  },
  retryAttempts: 3,
  timeout: 30000 // ms
};
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Starting scheduled content generation...');
  
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY
  });

  const topics = ['AI', 'Marketing Automation', 'Social Media Trends'];
  
  for (const topic of topics) {
    await pipeline.run({
      keyword: topic,
      autoPublish: true,
      platforms: ['facebook', 'linkedin']
    });
  }
});
```

### Pattern 2: Content Variation Testing

```typescript
// Generate multiple versions for A/B testing
async function generateContentVariations(topic: string) {
  const variations = await Promise.all([
    generateContent({ topic, tone: 'professional' }),
    generateContent({ topic, tone: 'friendly' }),
    generateContent({ topic, tone: 'humorous' })
  ]);

  return variations.map((content, index) => ({
    id: `var-${index}`,
    content,
    tone: ['professional', 'friendly', 'humorous'][index]
  }));
}
```

### Pattern 3: Bulk Video Generation

```typescript
async function generateBulkVideos(contents: Content[]) {
  const platforms = ['instagram-reels', 'tiktok', 'youtube-shorts'] as const;
  
  const videos = [];
  
  for (const content of contents) {
    for (const platform of platforms) {
      const video = await renderVideo({
        compositionId: 'ContentVideo',
        inputProps: {
          ...content,
          platform
        },
        outputPath: `./output/${content.id}-${platform}.mp4`
      });
      
      videos.push({ content: content.id, platform, video });
    }
  }
  
  return videos;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

const results = await Promise.all(
  urls.map(url => limit(() => crawlUrl(url)))
);
```

### Claude API Errors

```typescript
try {
  const content = await generateContent({...});
} catch (error) {
  if (error.status === 429) {
    console.error('Rate limit exceeded, waiting...');
    await new Promise(resolve => setTimeout(resolve, 60000));
    // Retry logic
  } else if (error.status === 500) {
    console.error('Claude API error, falling back to OpenAI');
    // Fallback to OpenAI
  }
}
```

### Remotion Rendering Issues

```typescript
// Debug video rendering
import { renderMedia, getCompositions } from '@remotion/renderer';

const compositions = await getCompositions(bundleLocation);
console.log('Available compositions:', compositions);

// Check for missing props
const composition = compositions.find(c => c.id === 'ContentVideo');
if (!composition) {
  throw new Error('Composition not found');
}

console.log('Required props:', composition.defaultProps);
```

### Memory Issues with Large Videos

```typescript
// Configure Remotion for better memory management
const video = await renderMedia({
  composition: compositionId,
  serveUrl: bundled,
  codec: 'h264',
  imageFormat: 'jpeg',
  jpegQuality: 80, // Reduce quality to save memory
  scale: 0.8, // Reduce resolution slightly
  concurrency: 2, // Limit concurrent frames
  outputLocation: outputPath
});
```

### Database Connection Issues

```typescript
// Implement connection pooling
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});

// Use connection
const client = await pool.connect();
try {
  await client.query('INSERT INTO content ...');
} finally {
  client.release();
}
```

## Advanced Usage

### Custom AI Prompts

```typescript
// Customize content generation prompts
const customPrompt = `
You are an expert content writer specializing in ${topic}.
Create a ${format} article that:
- Uses data from: ${research.map(r => r.title).join(', ')}
- Targets audience: ${targetAudience}
- Includes actionable insights
- Maintains ${tone} tone
- Length: ${wordCount} words
`;

const content = await generateWithOpenAI({
  prompt: customPrompt,
  temperature: 0.7,
  apiKey: process.env.OPENAI_API_KEY
});
```

### Webhook Integration

```typescript
// Send notifications on completion
import axios from 'axios';

async function notifyCompletion(result: PipelineResult) {
  await axios.post(process.env.WEBHOOK_URL!, {
    event: 'content_generated',
    data: {
      contentId: result.content.id,
      platforms: result.published.platforms,
      timestamp: new Date().toISOString()
    }
  });
}
```

This skill provides comprehensive coverage of the marketing pipeline automation system, enabling AI agents to effectively assist developers in implementing automated content creation workflows.
