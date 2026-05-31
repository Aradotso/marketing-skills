```markdown
---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scripting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate content with automated research
  - create video content from articles automatically
  - configure Claude and OpenAI for content generation
  - use Remotion to render marketing videos
  - automate content workflow from research to video
  - troubleshoot AI content pipeline errors
  - integrate content auto-posting features
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end automated content creation system that:
- **Auto-researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 or OpenAI
- **Renders videos** automatically using Remotion for social media platforms
- **Supports bilingual** content generation (English/Vietnamese)
- **Automates posting** to social media pages

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_api_key

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Social Media Auto-Post
FACEBOOK_PAGE_TOKEN=your_fb_page_token
FACEBOOK_PAGE_ID=your_page_id
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── research/          # Auto-crawling & data extraction
│   ├── content/           # AI content generation
│   ├── video/             # Remotion video rendering
│   ├── scheduling/        # Auto-post scheduler
│   └── api/               # API routes
├── components/            # React/Next.js UI components
├── remotion/              # Remotion video templates
└── public/                # Static assets
```

## Core Features & Usage

### 1. Automated Research Module

Crawl and analyze trending content from multiple sources:

```typescript
import { researchTopic } from './src/research/crawler';

async function runResearch() {
  const topic = "AI marketing automation";
  
  const insights = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });
  
  console.log('Research insights:', insights);
  // Returns: { articles: [], trends: [], dataPoints: [] }
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { generateContent } from './src/content/generator';

async function createArticle() {
  const content = await generateContent({
    topic: "Top 10 AI Tools for Marketing",
    format: 'toplist', // 'toplist' | 'pov' | 'casestudy' | 'howto'
    tone: 'expert', // 'expert' | 'friendly' | 'humorous'
    language: 'both', // 'en' | 'vi' | 'both'
    provider: 'claude', // 'claude' | 'openai'
    researchData: insights, // Optional: inject research data
    includeImages: true,
    includeVideo: true
  });
  
  return content;
  // Returns: { title, body, images, videoScript, metadata }
}
```

### 3. Video Generation with Remotion

Render videos from content automatically:

```typescript
import { renderVideo } from './src/video/remotion-renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

async function generateVideoContent(article) {
  const videoConfig = {
    composition: 'MarketingReel', // Template name
    inputProps: {
      title: article.title,
      highlights: article.keyPoints,
      backgroundColor: '#1a1a2e',
      accentColor: '#00ff87'
    },
    fps: 30,
    durationInFrames: 900, // 30 seconds at 30fps
    width: 1080,
    height: 1920, // Vertical for Reels/TikTok
  };
  
  const video = await renderVideo(videoConfig);
  return video.outputPath;
}
```

### 4. Complete Content Pipeline

End-to-end workflow combining all modules:

```typescript
import { ContentPipeline } from './src/pipeline';

async function runFullPipeline() {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    autoPost: true,
    platforms: ['facebook', 'linkedin']
  });
  
  const result = await pipeline.execute({
    keyword: "AI content marketing trends 2026",
    contentFormats: ['article', 'video'],
    schedulePost: {
      enabled: true,
      datetime: new Date('2026-06-01T09:00:00Z')
    }
  });
  
  console.log('Pipeline complete:', result);
  // Returns: { articleUrl, videoUrl, postIds, analytics }
}
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { researchTopic } from '@/src/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, sources } = req.body;
  
  try {
    const data = await researchTopic({ keyword, sources });
    res.status(200).json({ success: true, data });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/src/content/generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { topic, format, tone, language, provider } = req.body;
  
  try {
    const content = await generateContent({
      topic,
      format,
      tone,
      language,
      provider
    });
    
    res.status(200).json({ success: true, content });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Common Patterns

### Pattern 1: Daily Automated Content

```typescript
import cron from 'node-cron';
import { ContentPipeline } from './src/pipeline';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const pipeline = new ContentPipeline();
  
  const topics = [
    "AI marketing tools",
    "Content automation trends",
    "Video marketing strategies"
  ];
  
  for (const topic of topics) {
    await pipeline.execute({
      keyword: topic,
      contentFormats: ['article', 'video'],
      autoPost: true
    });
  }
});
```

### Pattern 2: Custom Video Template

```typescript
// remotion/MarketingReel.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const MarketingReel: React.FC<{
  title: string;
  highlights: string[];
  backgroundColor: string;
}> = ({ title, highlights, backgroundColor }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <div style={{ 
      backgroundColor, 
      width: '100%', 
      height: '100%',
      display: 'flex',
      flexDirection: 'column',
      justifyContent: 'center',
      alignItems: 'center',
      opacity
    }}>
      <h1 style={{ fontSize: 64, color: 'white' }}>{title}</h1>
      <ul style={{ fontSize: 32, color: 'white', marginTop: 40 }}>
        {highlights.map((h, i) => (
          <li key={i} style={{ marginBottom: 20 }}>{h}</li>
        ))}
      </ul>
    </div>
  );
};
```

### Pattern 3: Multi-Language Content Strategy

```typescript
async function generateBilingualContent(topic: string) {
  const [enContent, viContent] = await Promise.all([
    generateContent({
      topic,
      language: 'en',
      tone: 'expert',
      format: 'howto'
    }),
    generateContent({
      topic,
      language: 'vi',
      tone: 'friendly',
      format: 'howto'
    })
  ]);
  
  return {
    english: enContent,
    vietnamese: viContent
  };
}
```

## Configuration Options

### AI Provider Selection

```typescript
// config/ai-providers.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7
  }
};
```

### Content Formats

Available formats:
- `toplist` - Top 10/5 style articles
- `pov` - Point of view opinion pieces
- `casestudy` - In-depth case studies
- `howto` - Step-by-step tutorials

### Video Output Formats

```typescript
export const VIDEO_PRESETS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  youtube: { width: 1920, height: 1080 },
  square: { width: 1080, height: 1080 }
};
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting
import { RateLimiter } from 'limiter';

const limiter = new RateLimiter({
  tokensPerInterval: 10,
  interval: 'minute'
});

async function rateLimitedRequest(fn: () => Promise<any>) {
  await limiter.removeTokens(1);
  return fn();
}
```

### Issue: Remotion Rendering Fails

Check memory allocation and video complexity:

```typescript
const video = await renderVideo({
  ...config,
  chromiumOptions: {
    args: ['--disable-web-security', '--no-sandbox']
  },
  timeoutInMilliseconds: 120000 // 2 minutes
});
```

### Issue: Content Quality Issues

Improve prompts with more context:

```typescript
const content = await generateContent({
  topic,
  format: 'toplist',
  additionalContext: `
    Target audience: Marketing professionals
    Tone: Professional but accessible
    Include: Real statistics and examples
    Avoid: Generic advice
  `,
  researchData: insights // Always include research
});
```

### Issue: Failed Social Media Posts

Implement retry logic:

```typescript
async function postWithRetry(content, platform, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await postToSocial(content, platform);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
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
npx remotion render src/index.tsx MarketingReel out/video.mp4
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use webhooks** for long-running video renders
3. **Implement queue system** (Bull, BullMQ) for pipeline tasks
4. **Store generated content** in a database for analytics
5. **Monitor API usage** to stay within rate limits
6. **Test video templates** before production rendering
7. **Schedule posts** during peak engagement hours

## Advanced Usage: Custom Research Sources

```typescript
// src/research/custom-source.ts
import axios from 'axios';

export async function crawlCustomSource(url: string) {
  const response = await axios.get(url, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
    }
  });
  
  return {
    articles: response.data.articles.map(a => ({
      title: a.title,
      url: a.url,
      publishedAt: a.publishedAt,
      summary: a.description
    }))
  };
}
```

This skill covers the complete workflow for using the Marketing Pipeline Share project to automate content creation from research through video generation and social posting.
```
