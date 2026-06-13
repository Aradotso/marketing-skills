---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, social posting and video generation using AI
triggers:
  - automate content creation with AI research
  - generate video from content automatically
  - build AI content pipeline with Claude and OpenAI
  - crawl news and create social media posts
  - create automated marketing content workflow
  - set up content automation with video rendering
  - generate multilingual content with AI
  - build content research and video pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a comprehensive AI-powered content automation system that handles the entire content lifecycle: from researching trending topics across news sources, generating multilingual scripts with Claude/OpenAI, to automatically rendering videos with Remotion and posting to social platforms.

## What It Does

The Ultimate AI Content Pipeline automates:

1. **Research Phase**: Crawls recent articles from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **Content Generation**: Creates posts in multiple formats (toplist, POV, case study, how-to) and languages (EN/VI)
3. **Video Rendering**: Converts written content into infographics and short-form videos using Remotion
4. **Publishing**: Schedules and posts content to various platforms

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

### Required Environment Variables

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Social Media (optional for auto-posting)
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key
```

## Project Structure

```typescript
/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (OpenAI, Claude)
│   │   ├── crawler/     # News crawling logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Video templates
└── config/              # Configuration files
```

## Core APIs and Usage

### 1. Content Research (Crawling)

```typescript
import { researchTopic } from '@/lib/crawler/research';

// Crawl and analyze recent news on a topic
const research = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h',
  maxResults: 20
});

console.log(research.insights);
// Returns: { articles: [...], trends: [...], keyInsights: [...] }
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/generator/content';

// Generate content with Claude/OpenAI
const content = await generateContent({
  topic: 'AI automation trends',
  format: 'toplist', // or 'pov', 'case-study', 'how-to'
  language: 'vi', // or 'en'
  tone: 'expert', // or 'friendly', 'humorous'
  researchData: research.insights,
  provider: 'claude' // or 'openai'
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/render';

// Render video from content
const video = await renderVideo({
  content: content.body,
  template: 'infographic', // or 'reel', 'short'
  aspectRatio: '9:16', // TikTok/Reels/Shorts
  duration: 30,
  outputPath: './output/video.mp4'
});

console.log(video.path);
console.log(video.thumbnail);
```

### 4. Complete Pipeline Execution

```typescript
import { runContentPipeline } from '@/lib/pipeline';

// Run the entire pipeline
const result = await runContentPipeline({
  keyword: 'AI marketing automation',
  formats: ['toplist', 'pov'],
  languages: ['en', 'vi'],
  generateVideo: true,
  autoPost: false // Set to true to auto-publish
});

// Result contains all generated content and videos
result.contents.forEach(item => {
  console.log(`${item.language} ${item.format}: ${item.title}`);
  if (item.video) {
    console.log(`Video: ${item.video.path}`);
  }
});
```

## Configuration

### Content Templates

Create custom content templates in `config/templates.ts`:

```typescript
export const contentTemplates = {
  toplist: {
    structure: [
      'introduction',
      'items_with_numbers',
      'conclusion',
      'cta'
    ],
    minItems: 5,
    maxItems: 10
  },
  pov: {
    structure: [
      'hook',
      'personal_opinion',
      'supporting_evidence',
      'counterargument',
      'conclusion'
    ]
  }
};
```

### AI Provider Configuration

```typescript
// lib/ai/config.ts
export const aiConfig = {
  openai: {
    model: 'gpt-4-turbo-preview',
    temperature: 0.7,
    maxTokens: 2000
  },
  claude: {
    model: 'claude-3-opus-20240229',
    temperature: 0.8,
    maxTokens: 2000
  }
};
```

### Crawler Configuration

```typescript
// lib/crawler/config.ts
export const crawlerConfig = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com/feed/',
      selector: '.post-block',
      enabled: true
    },
    a16z: {
      url: 'https://a16z.com/feed/',
      selector: 'article',
      enabled: true
    }
  },
  rateLimit: {
    requestsPerMinute: 10,
    concurrent: 3
  }
};
```

## Common Patterns

### Pattern 1: Research-First Workflow

```typescript
// 1. Research the topic
const research = await researchTopic({
  keyword: 'AI agents',
  sources: ['techcrunch', 'twitter'],
  timeframe: '48h'
});

// 2. Generate content from research
const content = await generateContent({
  topic: 'AI agents revolution',
  researchData: research.insights,
  format: 'case-study',
  language: 'en'
});

// 3. Create video
const video = await renderVideo({
  content: content.body,
  template: 'reel',
  aspectRatio: '9:16'
});
```

### Pattern 2: Batch Content Generation

```typescript
const topics = ['AI automation', 'Marketing trends', 'Content strategy'];
const formats = ['toplist', 'pov', 'how-to'];

const batchResults = await Promise.all(
  topics.flatMap(topic =>
    formats.map(format =>
      generateContent({
        topic,
        format,
        language: 'en',
        provider: 'claude'
      })
    )
  )
);

console.log(`Generated ${batchResults.length} pieces of content`);
```

### Pattern 3: Multilingual Content Pipeline

```typescript
async function createMultilingualContent(keyword: string) {
  const research = await researchTopic({ keyword });
  
  const languages = ['en', 'vi'];
  const contents = await Promise.all(
    languages.map(lang =>
      generateContent({
        topic: keyword,
        language: lang,
        format: 'toplist',
        researchData: research.insights
      })
    )
  );
  
  return {
    en: contents[0],
    vi: contents[1]
  };
}
```

### Pattern 4: Video Template Customization

```typescript
// remotion/templates/custom-reel.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const CustomReel: React.FC<{ content: string }> = ({ content }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ 
        opacity: Math.min(1, frame / 30),
        color: 'white',
        fontSize: 48
      }}>
        {content}
      </div>
    </AbsoluteFill>
  );
};

// Use in rendering
import { CustomReel } from '@/remotion/templates/custom-reel';

const video = await renderVideo({
  content: 'Your content here',
  template: CustomReel,
  aspectRatio: '9:16'
});
```

## CLI Commands

If the project includes CLI tools:

```bash
# Run research on a topic
npm run research -- --keyword "AI trends" --sources techcrunch,a16z

# Generate content
npm run generate -- --topic "Marketing automation" --format toplist --lang vi

# Render video from content file
npm run render -- --input ./content.json --template reel --output ./video.mp4

# Run full pipeline
npm run pipeline -- --keyword "AI agents" --formats toplist,pov --languages en,vi --video
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run production server
npm start

# Run tests
npm test

# Lint code
npm run lint
```

## Troubleshooting

### Issue: API Rate Limiting

```typescript
// Implement retry logic with exponential backoff
import { retry } from '@/lib/utils/retry';

const content = await retry(
  () => generateContent({ topic: 'AI trends', format: 'toplist' }),
  {
    maxAttempts: 3,
    delay: 1000,
    backoff: 2
  }
);
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for complex videos
const video = await renderVideo({
  content: longContent,
  template: 'infographic',
  timeout: 120000, // 2 minutes
  concurrency: 1 // Reduce concurrent renders
});
```

### Issue: Crawler Blocked

```typescript
// Use proxy or add delays
import { crawlerConfig } from '@/lib/crawler/config';

crawlerConfig.proxy = process.env.PROXY_URL;
crawlerConfig.rateLimit.delay = 2000; // 2s between requests
```

### Issue: Out of Memory During Batch Processing

```typescript
// Process in smaller chunks
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent operations

const results = await Promise.all(
  topics.map(topic =>
    limit(() => generateContent({ topic, format: 'toplist' }))
  )
);
```

## Best Practices

1. **Always provide research data** to AI generators for better, data-backed content
2. **Cache research results** to avoid redundant crawling
3. **Use appropriate AI models**: Claude for creative content, GPT-4 for structured data
4. **Monitor API costs**: Track token usage across OpenAI/Claude calls
5. **Test video templates** before batch rendering
6. **Implement error handling** for all external API calls
7. **Use environment-specific configs** for development vs production

## Integration Examples

### Scheduling with Cron

```typescript
// pages/api/cron/daily-content.ts
import { runContentPipeline } from '@/lib/pipeline';

export default async function handler(req, res) {
  if (req.headers.authorization !== `Bearer ${process.env.CRON_SECRET}`) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  const result = await runContentPipeline({
    keyword: 'daily tech news',
    formats: ['toplist'],
    languages: ['en', 'vi'],
    generateVideo: true,
    autoPost: true
  });

  res.status(200).json({ success: true, generated: result.contents.length });
}
```

### Webhook Integration

```typescript
// pages/api/webhook/generate.ts
export default async function handler(req, res) {
  const { topic, format, language } = req.body;

  const content = await generateContent({
    topic,
    format,
    language,
    provider: 'claude'
  });

  // Send to your CMS or storage
  await saveToDatabase(content);

  res.status(200).json({ content });
}
```

This skill enables AI coding agents to help developers implement comprehensive content automation workflows using the Ultimate AI Content Pipeline project.
