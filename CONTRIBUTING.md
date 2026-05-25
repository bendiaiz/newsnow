# Contributing to NewsNow

Thank you for considering contributing to NewsNow! This document provides guidelines and instructions for contributing to the project.

## Adding a New Source

NewsNow is built to be easily extensible with new sources. Here's a step-by-step guide on how to add a new source:

### 1. Create a Feature Branch

Always create a feature branch for your changes:

```bash
git checkout -b feature-name
```

For example, to add a Bilibili hot video source:

```bash
git checkout -b bilibili-hot-video
```

### 2. Register the Source in Configuration

Add your new source to the source configuration in `/shared/pre-sources.ts`:

```
  "bilibili": {
  name: "哔哩哔哩",
  color: "blue",
  home: "https://www.bilibili.com",
  sub: {
    "hot-search": {
      title: "热搜",
      column: "china",
      type: "hottest"
    },
    "hot-video": {  // Add your new sub-source here
      title: "热门视频",
      column: "china",
      type: "hottest"
    }
  }
}
```

For a completely new source, add a new top-level entry:

```
"newsource": {
  name: "New Source",
  color: "blue",
  home: "https://www.example.com",
  column: "tech", // Pick an appropriate column
  type: "hottest" // Or "realtime" if it's a news feed
};
```

### 3. Implement the Source Fetcher

Create or modify a file in the `/server/sources/` directory. If your source is related to an existing one (like adding a new Bilibili sub-source), modify the existing file:

```typescript
// In /server/sources/bilibili.ts

// Define interface for API response
interface HotVideoRes {
  code: number
  message: string
  ttl: number
  data: {
    list: {
      aid: number
      // ... other fields
      bvid: string
      title: string
      pubdate: number
      desc: string
      pic: string
      owner: {
        mid: number
        name: string
        face: string
      }
      stat: {
        view: number
        like: number
        reply: number
        // ... other stats
      }
    }[]
  }
}

// Define source getter function
const hotVideo = defineSource(async () => {
  const url = "https://api.bilibili.com/x/web-interface/popular"
  const res: HotVideoRes = await myFetch(url)

  return res.data.list.map(video => ({
    id: video.bvid,
    title: video.title,
    url: `https://www.bilibili.com/video/${video.bvid}`,
    pubDate: video.pubdate * 1000,
    extra: {
      info: `${video.owner.name} · ${formatNumber(video.stat.view)}观看 · ${formatNumber(video.stat.like)}点赞`,
      hover: video.desc,
      icon: video.pic,
    },
  }))
})

// Helper function for formatting numbers
// Note: Bilibili uses 万 (10,000) as the base unit, so we use 'w' as shorthand here.
// If you're adding a non-Chinese source, you may want to use 'k' (1,000) and 'm' (1,000,000) instead.
function formatNumber(num: number): string {
  if (num >= 10000) {
    return `${Math.floor(num / 10000)}w+`
  }
  return num.toString()
}

// Export the source
export default defineSource({
  "bilibili": hotSearch,
  "bilibili-hot-search": hotSearch,
  "bilibili-hot-video": hotVideo, // Add your new source here
})
```

For completely new sources, create a new file in `/server/sources/` named after your source (e.g., `newsour
