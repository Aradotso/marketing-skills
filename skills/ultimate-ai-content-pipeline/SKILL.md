---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline that researches, generates scripts, creates videos, and auto-posts using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate video content from text automatically
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content with video rendering
  - build content automation workflow with Remotion
  - research and generate social media content automatically
  - automate content from research to video generation
  - create multi-format AI content pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from researching trending news across platforms like TechCrunch, Twitter/X, and LinkedIn, to generating multi-format content (articles, scripts) using Claude/OpenAI, and finally rendering videos automatically with Remotion. It's designed for content creators, marketers, and businesses to reduce manual work by up to 90%.

## What This Project Does

- **Auto-Research**: Crawls and analyzes real-time data from major news sources (TechCrunch, a16z, X, LinkedIn) within the last 24 hours
- **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
- **Multi-Language Support**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically converts text content into infographics and short-form videos using Remotion
- **Platform Optimization**: Exports videos in formats optimized for Reels, TikTok, and YouTube Shorts

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Clone and Install

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research modules
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key API Usage

### 1. Content Research Module

```typescript
import { researchContent } from '@/lib/research/auto-scan';

// Research trending topics
async function getTrendingContent(keyword: string) {
  const research = await researchContent({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });
  
  return research;
}

// Example response structure
interface ResearchResult {
  articles: Array<{
    title: string;
    url: string;
    summary: string;
    publishedAt: string;
    source: string;
  }>;
  insights: string[];
  trends: string[];
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/generator';

// Generate content with Claude or OpenAI
async function createArticle(topic: string, format: string) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    prompt: {
      topic,
      format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
      language: 'vi', // or 'en'
      tone: 'professional' // 'friendly' | 'humorous'
    },
    researchData: await researchContent({ keyword: topic })
  });
  
  return content;
}

// Example with OpenAI
async function generateWithOpenAI(topic: string) {
  const content = await generateContent({
    provider: 'openai',
    model: 'gpt-4-turbo-preview',
    prompt: {
      topic,
      format: 'how-to',
      language: 'en',
      tone: 'friendly'
    }
  });
  
  return content;
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-render';
import { VideoComposition } from '@/remotion/compositions';

// Render video from content
async function createVideo(content: GeneratedContent) {
  const video = await renderVideo({
    composition: VideoComposition,
    props: {
      title: content.title,
      sections: content.sections,
      style: 'modern',
      platform: 'tiktok' // 'reels' | 'shorts' | 'tiktok'
    },
    outputFormat: 'mp4',
    dimensions: {
      width: 1080,
      height: 1920 // 9:16 for vertical video
    }
  });
  
  return video.url;
}

// Custom Remotion composition example
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  sections: string[];
}> = ({ title, sections }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  return (
    <div className="video-container">
      <h1 style={{ opacity: Math.min(1, frame / 30) }}>
        {title}
      </h1>
      {sections.map((section, i) => (
        <div key={i} className="section">
          {section}
        </div>
      ))}
    </div>
  );
};
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Full automation from research to video
async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoRenderer: 'remotion'
  });
  
  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h'
  });
  
  // Step 2: Generate content
  const content = await pipeline.generate({
    researchData: research,
    format: 'toplist',
    languages: ['en', 'vi']
  });
  
  // Step 3: Render video
  const video = await pipeline.renderVideo({
    content: content.en,
    platform: 'tiktok'
  });
  
  // Step 4: Schedule post (optional)
  await pipeline.schedulePost({
    content,
    video,
    platforms: ['facebook', 'tiktok'],
    scheduledAt: new Date('2024-06-03T10:00:00Z')
  });
  
  return {
    research,
    content,
    video
  };
}
```

## Configuration Patterns

### Custom Research Sources

```typescript
// lib/research/config.ts
export const researchConfig = {
  sources: {
    techcrunch: {
      enabled: true,
      apiEndpoint: process.env.TECHCRUNCH_API,
      categories: ['ai', 'startup', 'fintech']
    },
    twitter: {
      enabled: true,
      apiKey: process.env.TWITTER_API_KEY,
      searchHashtags: ['#AI', '#Tech', '#Marketing']
    },
    linkedin: {
      enabled: true,
      apiKey: process.env.LINKEDIN_API_KEY,
      targetInfluencers: ['influencer1', 'influencer2']
    }
  },
  caching: {
    enabled: true,
    ttl: 3600 // 1 hour
  }
};
```

### AI Model Configuration

```typescript
// lib/ai/config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7
  },
  fallback: 'openai' // Fallback if primary fails
};
```

### Remotion Video Templates

```typescript
// remotion/config.ts
export const videoConfig = {
  templates: {
    tiktok: {
      width: 1080,
      height: 1920,
      fps: 30,
      durationInFrames: 900 // 30 seconds
    },
    reels: {
      width: 1080,
      height: 1920,
      fps: 30,
      durationInFrames: 1800 // 60 seconds
    },
    youtube: {
      width: 1920,
      height: 1080,
      fps: 30,
      durationInFrames: 3600 // 120 seconds
    }
  }
};
```

## Common Usage Patterns

### Pattern 1: Quick Content Generation

```typescript
import { quickGenerate } from '@/lib/shortcuts';

// Generate content with minimal configuration
const result = await quickGenerate({
  keyword: 'AI Marketing Trends 2024',
  outputFormats: ['article', 'video'],
  autoPost: false
});
```

### Pattern 2: Batch Content Creation

```typescript
async function batchCreateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await researchContent({ keyword });
      const content = await generateContent({
        provider: 'claude',
        prompt: { topic: keyword, format: 'toplist' },
        researchData: research
      });
      const video = await renderVideo({ content });
      
      return { keyword, content, video };
    })
  );
  
  return results
    .filter((r) => r.status === 'fulfilled')
    .map((r) => r.value);
}
```

### Pattern 3: Multi-Language Content

```typescript
async function createMultiLanguageContent(topic: string) {
  const [enContent, viContent] = await Promise.all([
    generateContent({
      provider: 'claude',
      prompt: { topic, language: 'en', format: 'how-to' }
    }),
    generateContent({
      provider: 'claude',
      prompt: { topic, language: 'vi', format: 'how-to' }
    })
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### Pattern 4: Custom Video Style

```typescript
import { Composition } from 'remotion';

// Define custom video composition
export const CustomVideoStyle: React.FC<VideoProps> = ({
  title,
  content
}) => {
  return (
    <div className="custom-video">
      <div className="header">{title}</div>
      <div className="content">{content}</div>
      <div className="footer">@YourBrand</div>
    </div>
  );
};

// Register in remotion/index.ts
<Composition
  id="CustomStyle"
  component={CustomVideoStyle}
  durationInFrames={900}
  fps={30}
  width={1080}
  height={1920}
/>
```

## CLI Commands

### Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

### Remotion Video Commands

```bash
# Preview video compositions
npm run remotion:preview

# Render specific composition
npm run remotion:render -- --composition=VideoName --output=output.mp4

# Render all compositions
npm run remotion:render:all
```

### Testing

```bash
# Run tests
npm test

# Run with coverage
npm run test:coverage
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting
import { rateLimit } from '@/lib/utils/rate-limit';

const limiter = rateLimit({
  interval: 60 * 1000, // 1 minute
  uniqueTokenPerInterval: 500
});

async function apiCall() {
  await limiter.check(10); // 10 requests per minute
  // Make API call
}
```

### Issue: Video Rendering Fails

```typescript
// Add error handling and retries
async function renderWithRetry(config: VideoConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await renderVideo(config);
    } catch (error) {
      console.error(`Render attempt ${i + 1} failed:`, error);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

### Issue: AI Generation Timeout

```typescript
// Implement timeout wrapper
async function generateWithTimeout(
  generateFn: () => Promise<any>,
  timeoutMs = 30000
) {
  return Promise.race([
    generateFn(),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), timeoutMs)
    )
  ]);
}
```

### Issue: Memory Issues with Large Videos

```typescript
// Configure Remotion for better memory management
export const remotionConfig = {
  chromiumArgs: [
    '--max-old-space-size=4096',
    '--disable-dev-shm-usage'
  ],
  concurrency: 2 // Limit concurrent renders
};
```

## Best Practices

1. **Cache Research Results**: Store research data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue system for video rendering to prevent server overload
3. **Validate Content**: Always validate AI-generated content before auto-posting
4. **Monitor API Usage**: Track API usage to stay within rate limits and budgets
5. **Version Control Templates**: Keep video templates in version control for consistency

## Additional Resources

- Check `HUONG_DAN_CAI_DAT.md` for detailed Vietnamese installation guide
- Remotion documentation: https://www.remotion.dev/docs/
- Claude API: https://docs.anthropic.com/
- OpenAI API: https://platform.openai.com/docs/
